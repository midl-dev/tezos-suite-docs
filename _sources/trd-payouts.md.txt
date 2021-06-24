# TRD payouts

We support the [Tezos Reward Distributor](https://github.com/tezos-reward-distributor-organization/tezos-reward-distributor) natively.

The payout address must be on the same remote signer used for baking.

While the baking address can be on a Ledger device, the payout address must be on a hot wallet directly on the signer.

TRD works as a **cronjob**, meaning that it runs on a schedule, attempts to do a payout if there is one outstanding, then exits immediately. In TRD documentation, this is refered to as "run mode 2".

### Configuration

The `baking_nodes` object passed to tezos-on-gke optionally supports passing a `payout_config` object structured as follows:

```
module "tezos-baker" {
  baking_nodes = {
    "mynode" : {
      "mybaker" : {
        "public_baking_key": "tz1xxx",
        # more parameters (signer config, etc...) go here
        "payout_config" = {
          "schedule"="06 */3 * * *",
          "initial_cycle": 370,
          "release_override": -5,
          "network": "MAINNET",
          "reward_data_provider": "tzkt",
          "dry_run": "false",
          "payment_address": "tz1",
          "rewards_type": "actual",
          "service_fee": 5,
          "rules_map": {}
        }
      }
    }
  }
}
```

This is then converted into TRD parameters and passed to the TRD configuration.

Most [TRD configuration parameters](https://tezos-reward-distributor-organization.github.io/tezos-reward-distributor/configuration.html) are supported.

Be sure to run in `dry_mode` `true` at first to make sure that the payout engine works as expected.

The `schedule` parameter indicates how often the cronjob should run. It should be in cron format.


### Isolation

It is recommended to run TRD payouts in a different node pool than the tezos nodes, for isolation.

You can configure the node pool used for payouts with the `kubernetes_payout_pool_name` terraform variable.

### Full example

See the [production readiness](production-readiness) section for a full example of baker with payout configured.
