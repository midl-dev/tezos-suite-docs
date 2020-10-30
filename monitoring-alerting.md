# Monitoring and alerting

## Monitoring stack: Prometheus, AlertManager

The standard way of doing monitoring and alerting within a Kubernetes cluster is using Prometheus and AlertManager.

When installing a Tezos Suite cluster, the [terraform-gke-blockchain module](https://github.com/midl-dev/terraform-gke-blockchain) will automatically install the [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator).

This sets up a default instance of Prometheus and Alertmanager to alert you of cluster-wide conditions.

The `monitoring_slack_url` parameter should be set to a Slack webhook (see [how to configure incoming webhooks](https://api.slack.com/messaging/webhooks)). You will receive alerts when your persistent volumes are filling up.

Additionally, Tezos-on-GKE configures a different instance of Prometheus and Alertmanager for Tezos-specific alerts, such as:

* Private node disconnected from public nodes
* public nodes do not have enough connections

These alerts will be sent to the same webhook configured above.

### Remote signer alerts

When used in combination with one or several remote signers, this monitoring framework will alert you of additional failure conditions, such as:

* remote signer unreachable
* remote signer has lost power (runs on battery)
* remote signer has lost wired Internet connection (runs on LTE dongle)

These alerts can be configured to be sent to a separate Slack channel. This is useful for separation of concerns, when different people or organizations are responsible for keeping the signers up vs. the cloud platform itself.

The webhook must be configured within the `baking_nodes` Terraform map parameter. Every baker map should contain the `monitoring_slack_url` and `monitorin_slack_channel` keys.

For example:

```
baking_nodes = {
  mynode = {
    mybaker = {
      public_baking_key="tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU"
      ledger_authorized_path="ledger://my-four-key-words/ed25519/0h/1h",
      monitoring_slack_url="https://hooks.slack.com/services/<snip>",
      monitoring_slack_channel="#remote-signer-monitor",
      authorized_signers : [
                { "ssh_pubkey" : "ssh-rsa AAAAB<snip>==",
                  "signer_port" : 8444 } ]
    }
  }
}
```

## External monitoring

In addition to the above, it is recommended to set up external monitoring from a completely different configuration, to monitor the baker from another node in the chain, and alert about missed bakes and endorsements. [Tezos-auxiliary-cluster](https://github.com/midl-dev/tezos-auxiliary-cluster) does that.
