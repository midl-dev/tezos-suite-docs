# Active-standby baking nodes

For release `v2.0` of tezos-on-gke, we are introducing a new, experimental mode of operation: the active-standby baker.

This feature can be activated by passing the following terraform variable:

```
experimental_active_standby_mode=true
```

## Architecture

Tezos-on-gke runs in a Kubernetes cluster with two availability zones. It uses a regional persistent volume for the private node storage, so in case one kubernetes node goes down, the private node can restart in another zone.

Reasons for kubernetes node to go down include:

* catastrophic failure of one cloud availability zone (unlikely)
* Kubernetes cluster auto-upgrades (more common)

When this happens, the private node migrates to the other node. This has been working fine and providing good uptime.

There is a problem, though. During a migration, the node does a cold restart, which means it needs to initialize itself, establish connections with the sentry nodes, and import the blocks it missed during the time off. This can easily take minutes, and during this time, blocks and endorsements may be missed.

We introduce an active-standby model where:

* we start two private baking nodes, one in each availability zone
* we no longer use regional volumes: storage is local to the AZ
* only one node is baking at a time; kubernetes native consensus mechanism is used to elect the master

### Kubernetes master elector

When two baking nodes are live, we need to select one to be the active baker. This problem can be solved with a distributed consensus protocol such as Raft. The good news is, Kubernetes already uses Raft and etcd natively, so we do not have to deploy it inside our cluster. We can leverage the native mechanism to elect our baker. The master election pattern is a [Kubernetes classic](https://kubernetes.io/blog/2016/01/simple-leader-election-with-kubernetes/).

Kubernetes Stateful Sets are a good fit for active-standby baker nodes, since they each have their own persistent storage. The Kubernetes scheduler will do its best to ensure at least one member of the set is ready at any point in time.

In this context, an individual pod teardown is a normal event. It could be due to an auto-upgrade, or a cluster scale-down. These events driven by Kubernetes control plane affect the master election.

We are making the pods aware of these changes of master by running a `master-elector` container and wrapping the baker and endorser processes into a supervisord script that will only start the daemons on the current master node.

### Tezos sidecar

We can further improve this setup by informing Kubernetes of the baking node `liveness` and `readiness`.

A Tezos node is `alive` if and only if it has at least one connection to a peer. A Tezos node is `ready` if the timestamp of its head block is at most 4 minutes old.

The Tezos sidecar running inside the baking pod will inform Kubernetes of the status of the baker. If for some reason, one node did not get the most recent blocks and is stuck in the past, the node will be marked as "not ready" and the other node will start baking. This further improves the usefulness of the active-standby setup.

### Nonce exchange

A baker selected to bake a block whose height is a multiple of 32 must produce a nonce and store it for revelation at the next cycle. These nonces are stored locally in the `client` storage dir.

When a switchover happens, the new master does not have the nonce, so it must retrieve it from the standby node. If the nonce is not revealed, the baking reward is lost.

We built `nonce-importer` and `nonce-exposer` containers that ensure that any new nonce built on one baker is written to the other baker's storage.

## Benefits

For a very busy baker, this setup can eliminate small amounts of downtime caused by a unique baker pod when it is restarting.

It makes operations easier: if some work has to be made on a baking node for any reason (software upgrade for example), the standby can take over.

This also protects against software failures: if the node is rendered unrecoverable, it is possible to rebuild it from scratch while the other node takes over the baking tasks.

## Risks

Split-brain is a classic scenario where two nodes are active because they lost communication with each other. In this case, the second level of defense is the signer.

The signer should refuse to sign two blocks at the same height. Using a remote signer device connected to a Ledger will achieve that.

When using Tezos-on-GKE with a cloud hosted key, do not enable this active-standby mode.
