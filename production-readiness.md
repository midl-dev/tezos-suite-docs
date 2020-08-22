# Production hardening

While the tezos-on-gke project lets you quickly deploy a cluster with a running baker setup, running a production baker requires careful planning and execution.

## Terraform service account

Usage of [Google Default Application Credentials](https://cloud.google.com/docs/authentication/production) is not recommended in a production environment.

Instead:

* ensure that you have set up an Organization - that can be done by registering a domain name and adding it to gcloud
* create a Terraform Admin Project, Terraform Service Account and Service Account Credentials following [this Google guide](https://cloud.google.com/community/tutorials/managing-gcp-projects-with-terraform)
* do not pass `project` as a variable when deploying the resources. Instead, pass `organization_id` and `billing_account` as variables
* pass the service account credentials json file `serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com` as `terraform_service_account_credentials` terraform variable

That will create the cluster in a new project, created by the terraform service account.

You may then grant people in your organization access to the project. It is recommended to write more terraform manifests to do so.

## Separate cluster definition from baker definition

For maintainability purposes, do not create a baker straight from tezos-on-gke `terraform` folder.

Instead, it is recommended to create a maintain a private terraform file, declaring a cluster, and every deployment that lives within.

The `tezos-on-gke` project supports this. The `terraform-no-cluster-create` folder will deploy the baker on a pre-existing cluster.

This keeps the setup maintainable by letting you define several deployments within the same cluster. For example, you may deploy the Tezos baker setup, and the Tezos monitoring setup, within the same cluster.

```
module "terraform-gke-blockchain" "my-cluster" {
...
}

module 
```

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

## Full example

A terraform manifest to deploy a Tezos baker.

## Going further

The repository is optimized for quick spinup with one operator: secrets are stored locally in a `terraform.tfvars` file.

A production validator should be operated with an on-call rotation, meaning several operators have access to the setup.

Specifically:

* secrets should be moved from a file in the operator workspace to a production secret store such as [Hashicorp Vault](vaultproject.io)
* terraform state should be stored centrally (in a google storage bucket)
* terraform deploys should be done by a CI system
* any manual change in the kubernetes environment should be recorded in an audit log and committed in the code (see [Gitops](https://www.weave.works/technologies/gitops/)).
