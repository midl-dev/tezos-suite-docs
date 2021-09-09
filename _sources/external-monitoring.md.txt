# External monitoring

The Tezos-on-GKE project employs internal monitoring with Prometheus and Alertmanager.

A good complement to internal monitoring is external monitoring: a fully separated setup whose task is to monitor the baking operation as a standalone blockchain node.

Software such as [Tezos-network-monior] or [Pyrometer](https://gitlab.com/tezos-kiln/pyrometer) can accomplish this task.

## Auxiliary cluster

We maintain a separate kubernetes codebase to deploy external monitoring in a separate cluster:

https://github.com/midl-dev/tezos-auxiliary-cluster
