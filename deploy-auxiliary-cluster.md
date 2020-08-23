# Deploy auxiliary cluster

The Tezos-on-GKE project employs internal monitoring with Prometheus and Alertmanager.

A good complement to internal monitoring is external monitoring: a fully separated setup whose task is to monitor the baking operation as a standalone blockchain node.

This can be a separate Kubernetes cluster, or a separate workspace on the same kubernetes cluster.

This auxiliary cluster also runs payout operations. Since baking and endorsing is more important than payouts, it makes sense to have the main cluster solely focused on this task.

## Codebase

https://github.com/midl-dev/tezos-auxiliary-cluster
