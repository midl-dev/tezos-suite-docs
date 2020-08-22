# Deploy a baker with remote signer

## Prerequisites

You will need your baking address and Ledger authorized path. [Follow these instructions](cul) to get them.

## Instructions

First, deploy the Tezos-on-GKE baking setup in the cluster.

You may deploy it from the tezos-on-gke repo itself using the [quickstart](cul) instructions, however it is better to follow [production best practices](cul).

Create a file `main.tf` where you store your cluster proprietary information.

As parameter to deploy the cluster, provide a `baking_nodes` map as follows:

```
baking_nodes = {
  mynode = {
    mybaker = {
      public_baking_key="tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU"
      ledger_authorized_path="ledger://my-four-key-words/ed25519/0h/1h",
      authorized_signers : []

    }
  }
}
```

Note that `authorized_signers` is empty for now.

### ssh endpoint host key

A ssh server normally generates its host keys during installation. Here, we are generating them externally and injecting them into the setup as a terraform and kubernetes secrets.

This way, in case of complete destruction of the cluster, the operator is able to restore the ssh endpoint with the same host key.

Combined with a static hostname (see explanation below), the remote signer will connect to a newly recovered cluster without noticing it has changed. If the host key changes, there will be a ssh warning preventing the remote signer from connecting.

Since you may not have access to the remote signer, it is essential to configure this host key.

Pass it as a variable to keep it secret:

```
signer_target_host_key=var.signer_target_host_key
```

Then in terraform.tfvars, use a heredoc to specify the key:

```
signer_target_host_key = <<-EOK
-----BEGIN OPENSSH PRIVATE KEY-----
<the key>
-----END OPENSSH PRIVATE KEY-----
EOK
```


### Get the tunneling endpoint IP address

Issue the following command:

```
gcloud compute addresses list --project <project_name>
```

The `tezos-baker-lb` address is the one that you will configure the remote signer to connect to.

It is recommended

Use the [Tezos remote signer OS](
