# Production hardening

While the tezos-on-gke project lets you quickly deploy a cluster with a running baker setup, running a production baker requires careful planning and execution.

## Terraform service account

Usage of [Google Default Application Credentials](https://cloud.google.com/docs/authentication/production) is not recommended in a production environment.

Instead:

* ensure that you have set up an Organization - that can be done by registering a domain name and adding it to Google Cloud;
* create a Terraform Admin Project, Terraform Service Account and Service Account Credentials following [this Google guide](https://cloud.google.com/community/tutorials/managing-gcp-projects-with-terraform);
* do not pass `project` as a variable when deploying the resources. Instead, pass `organization_id` and `billing_account` as variables;
* pass the service account credentials JSON file `serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com` as `terraform_service_account_credentials` Terraform variable.

That will create the cluster in a new project, created by the Terraform service account.

You may then grant people in your organization access to the project. It is recommended to write more Terraform manifests to do so.

## Separate cluster definition from baker definition

While you can create a baker in one-shot, it is best suited for demos and testnets. A production baker is best advised to be defined declaratively.

You would normally write all the parameters defining your baker in a `terraform.tfvars` file on your machine.

Instead, it is recommended to create and maintain a private Terraform manifest, declaring a cluster, and every deployment that lives within. This way, the paramters defining your cluster can also be committed to git (except secrets which should be handled separately, more on this below).

This keeps the setup maintainable by letting you define several deployments within the same cluster. For example, you may to deploy the Tezos baker setup, and the Tezos monitoring setup, within the same cluster.

### Define the cluster

The [terraform-gke-blockchain](https://github.com/midl-dev/terraform-gke-blockchain) repository contains boilerplate Terraform code to deploy a Kubernetes cluster.

Start by declaring one empty cluster and one Terraform provider:

```
module "terraform-gke-blockchain" {
  source                                = "github.com/midl-dev/terraform-gke-blockchain?ref=v2.0"
  org_id                                = "<my org id, defined above>"
  billing_account                       = "<my billing account, defined above>"
  project_prefix                        = "mybakingop"
  monitoring_slack_url                  = var.monitoring_slack_url
  terraform_service_account_credentials = "~/.config/gcloud/terraform-service-account-credentials.json"
  node_pools = { "baking_pool" : { "node_count": 1, "instance_type": "e2-standard-2" },
    "monitoring_pool" : { "node_count": 1, "instance_type": "e2-standard-1" } }
}

# This file contains all the interactions with Kubernetes
provider "kubernetes" {
  host             = module.terraform-gke-blockchain.kubernetes_endpoint
  cluster_ca_certificate = module.terraform-gke-blockchain.cluster_ca_certificate
  token = data.google_client_config.current.access_token
}
```

Notice that we created two node pools. These are distinct virtual machines that run your Kubernetes cluster. You can map your pods to either. We will be using these to separate the baker setup from the payout/monitoring setup.

### Define the Tezos baker

Within the `tezos-on-gke` repository, the `terraform-no-cluster-create` folder will deploy the baker on a pre-existing cluster.

The output parameters of the `terraform-gke-blockchain` module become the input parameters of the Tezos baker module.

All variables will appear in the Terraform manifest itself, except secrets. Secrets should be kept as variables, and handled appropriately.

It looks like:

```
module "tezos-baker" {
  source                          = "github.com/midl-dev/tezos-on-gke?ref=v3.0//terraform-no-cluster-create"
  region                          = module.terraform-gke-blockchain.location
  node_locations                  = module.terraform-gke-blockchain.node_locations
  kubernetes_endpoint             = module.terraform-gke-blockchain.kubernetes_endpoint
  cluster_ca_certificate          = module.terraform-gke-blockchain.cluster_ca_certificate
  cluster_name                    = module.terraform-gke-blockchain.name
  kubernetes_access_token         = data.google_client_config.current.access_token
  kubernetes_pool_name            = "baking_pool"
  project                         = module.terraform-gke-blockchain.project
  full_snapshot_url               = "https://mainnet.xtz-shots.io/full"
  rolling_snapshot_url            = "https://mainnet.xtz-shots.io/rolling"
  kubernetes_namespace            = "tezos"
  kubernetes_name_prefix          = "xtz"
  tezos_version                   = "v9.2"
  tezos_network                   = "mainnet"
  baking_nodes = {
    "mybaker" : {
      "mynode" : {
        "public_baking_key_hash": "tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU",
        "public_baking_key": "edpk...",
        "insecure_private_baking_key": "edsk3cftTNcJnxb7ehCxYeCaKPT7mjycdMxgFisLixrQ9bZuTG2yZK"
      }
    }
  }
}
```

### With remote signer

It is recommended to use a remote signer for secure operations.

Below is an example of a baker with remote signer configured:

```
module "tezos-baker" {
  source                          = "github.com/midl-dev/tezos-on-gke?ref=v3.0//terraform-no-cluster-create"
  region                          = module.terraform-gke-blockchain.location
  node_locations                  = module.terraform-gke-blockchain.node_locations
  kubernetes_endpoint             = module.terraform-gke-blockchain.kubernetes_endpoint
  cluster_ca_certificate          = module.terraform-gke-blockchain.cluster_ca_certificate
  cluster_name                    = module.terraform-gke-blockchain.name
  kubernetes_access_token         = data.google_client_config.current.access_token
  kubernetes_pool_name            = "baking_pool"
  project                         = module.terraform-gke-blockchain.project
  full_snapshot_url               = "https://mainnet.xtz-shots.io/full"
  rolling_snapshot_url            = "https://mainnet.xtz-shots.io/rolling"
  kubernetes_namespace            = "tezos"
  kubernetes_name_prefix          = "xtz"
  tezos_version                   = "v9.2"
  tezos_network                   = "mainnet"
  signer_target_host_key=var.signer_target_host_key
  baking_nodes = {
    "mybaker" : {
      "mynode" : {
        "public_baking_key_hash": "tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU",
        "public_baking_key": "edpk...",
        "ledger_authorized_path": "ledger://my-four-key-words/ed25519/0h/1h",
        authorized_signers : [
                { "ssh_pubkey" : "ssh-rsa AAAAB<snip>==",
                  "signer_port" : 8443,
                  "tunnel_endpoint_port" : 51756 }
        ]
      }
    }
  }
}
```

### Payout configuration

The `baking_nodes` section also accepts a config for TRD payouts (see the [TRD payouts](trd-payouts) section for details).

## Terraform remote state

Terraform normally maintains state locally. Accidental loss of this file will cause your setup to be unmaintainable. Therefore, it is good practice to store the state in a remote Storage Bucket.

The best location for this storage bucket is the Terraform Admin project created above.

In the private terraform file, add the following:

```
terraform {
  backend "gcs" {
    bucket  = "terraform-state-midl-prod"
    prefix  = "terraform/state"
  }
}
```

The state will now be stored remotely.

More info in the [Terraform documentation](https://www.terraform.io/docs/backends/types/gcs.html).

## Putting it all together

This terraform manifest deploys a full Tezos baker.

It creates a cluster with two node pools: one for the baker and one for the remaining containers.

It deploys a baker and an auxiliary cluster handling the payouts, external monitoring and website.

```
terraform {
  backend "gcs" {
    bucket  = "terraform-state-midl-prod"
    prefix  = "terraform/state"
  }
}

module "terraform-gke-blockchain" {
  source                                = "github.com/midl-dev/terraform-gke-blockchain?ref=v2.0"
  org_id                                = "<my org id, defined above>"
  billing_account                       = "<my billing account, defined above>"
  project_prefix                        = "mybakingop"
  monitoring_slack_url                  = var.monitoring_slack_url
  terraform_service_account_credentials = "~/.config/gcloud/terraform-service-account-credentials.json"
  node_pools = { "baking_pool" : { "node_count": 1, "instance_type": "e2-standard-2" },
    "monitoring_pool" : { "node_count": 1, "instance_type": "e2-standard-1" } }
}

module "tezos-baker" {
  source                          = "github.com/midl-dev/tezos-on-gke?ref=v3.0//terraform-no-cluster-create"
  region                          = module.terraform-gke-blockchain.location
  node_locations                  = module.terraform-gke-blockchain.node_locations
  kubernetes_endpoint             = module.terraform-gke-blockchain.kubernetes_endpoint
  cluster_ca_certificate          = module.terraform-gke-blockchain.cluster_ca_certificate
  cluster_name                    = module.terraform-gke-blockchain.name
  kubernetes_access_token         = data.google_client_config.current.access_token
  project                         = module.terraform-gke-blockchain.project
  kubernetes_pool_name            = "baking_pool"
  kubernetes_namespace            = "tezos"
  kubernetes_name_prefix          = "xtz"
  full_snapshot_url               = "https://mainnet.xtz-shots.io/full"
  rolling_snapshot_url            = "https://mainnet.xtz-shots.io/rolling"
  tezos_version                   = "v9.2"
  tezos_network                   = "mainnet"
  signer_target_host_key=var.signer_target_host_key
  baking_nodes = {
    "mynode" : {
      "mybaker" : {
        "public_baking_key_hash": "tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU",
        "public_baking_key": "edpk...",
        "ledger_authorized_path": "ledger://my-four-key-words/ed25519/0h/1h",
        authorized_signers : [
                { "ssh_pubkey" : "ssh-rsa AAAAB<snip>==",
                  "signer_port" : 8443,
                  "tunnel_endpoint_port" : 51756 }
        ]
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

module "tezos-mainnet-monitoring" {
  source                          = "github.com/midl-dev/tezos-auxiliary-cluster?ref=v2.0//terraform-no-cluster-create"
  region                          = module.terraform-gke-blockchain.location
  node_locations                  = module.terraform-gke-blockchain.node_locations
  kubernetes_endpoint             = module.terraform-gke-blockchain.kubernetes_endpoint
  cluster_ca_certificate          = module.terraform-gke-blockchain.cluster_ca_certificate
  cluster_name                    = module.terraform-gke-blockchain.name
  kubernetes_access_token         = data.google_client_config.current.access_token
  project                         = module.terraform-gke-blockchain.project
  kubernetes_pool_name            = "monitoring_pool"
  kubernetes_namespace            = "tezos"
  kubernetes_name_prefix          = "xtz"
  tezos_private_version           = "v9.2"
  website                         = "my_baking_website"
  website_bucket_url              = "<my bucket url>"
  website_archive                 = "<my_website_archive_url>"
  public_baking_key               = "<public baking key goes here>"
}
```

Note that all values above are not secrets, it is fine to commit them in a private repository. The secrets are passed with variables and must be handled separately.

## Going further

A production validator should be operated with an on-call rotation, meaning several operators have access to the setup.

Specifically:

* secrets should be moved from a file in the operator workspace to a production secret store such as [Hashicorp Vault](vaultproject.io);
* Terraform deploys should be done by a CI system;
* any manual change in the Kubernetes environment should be recorded in an audit log and committed in the code:
  * the Terraform private file above can be applied with continuous integration;
  * the intermediate Kubernetes code generated with kustomize could be stored in a CI pipeline and deployed in an auditable way as well (see [Gitops](https://www.weave.works/technologies/gitops/)).
