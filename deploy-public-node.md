# Deploy a public node

## Instructions

If you do not pass any value to the `baking_nodes` variable, the setup will come up as a set of public nodes, optionally accessible with RPC.

Here is an example Terraform configuration to deploy a public node:

```
tezos_sentry_version="v7.4"
monitoring_slack_url = var.monitoring_slack_url
tezos_network="mainnet"
node_storage_size=40
history_mode="full"
snapshot_url = "https://mainnet.xtz-shots.io/full"
rpc_public_hostname="my-public-rpc.example.com"
# optional, the list of ips authorized to access the public rpc
rpc_subnet_whitelist=["0.0.0.0/0"]
```

The number of replicas for the Tezos node will be equal to the number of Kubernetes nodes (so by default, 2 nodes). Ingress RPC traffic will be load balanced between these nodes.
