# Deploy a baker with remote signer

The [quickstart guide](quickstart) indicates how to set up a cloud baker with the private keys hosted in [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/). In this section, we configure a remote signer on-premises. This configuration is more secure and every mainnet deployment should use it.

## Prerequisites

You will need your baking public key and Ledger authorized path. [Follow these instructions](setup_baker) to get them.

## Instructions

First, deploy the Tezos-on-GKE baking setup in the cluster.

You may deploy it from the tezos-on-gke repo itself using the [quickstart](https://github.com/midl-dev/tezos-on-gke) instructions, however it is better to follow [production best practices](production-readiness).

In these instructions, we will be declaring a `tezos-baker` Terraform module, keeping all sensitive data in a `terraform.tfvars` file.

First, on an empty project dir, create a new `main.tf` file:

```
module "tezos-baker" {
  source = "github.com/midl-dev/tezos-on-gke?ref=v2.0//terraform-no-cluster-create"
  baking_nodes = {
    mynode = {
      mybaker = {
        public_baking_key="edpk..."
        public_baking_key_hash="tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU"
        ledger_authorized_path="ledger://my-four-key-words/ed25519/0h/1h",
        authorized_signers : []
      }
    }
  }
  signer_target_host_key=var.signer_target_host_key
}
```

Note that `authorized_signers` is empty for now.

### ssh endpoint host key

A ssh server normally generates its host keys during installation. Here, we are generating them externally and injecting them into the setup as a terraform and kubernetes secrets.

This way, in case of complete destruction of the cluster, the operator is able to restore the ssh endpoint with the same host key.

Combined with a static hostname (see explanation below), the remote signer will connect to a newly recovered cluster without noticing it has changed. If the host key changes, there will be a ssh warning preventing the remote signer from connecting.

If you cannot directly ssh to the remote signer, setting the ssh host key means that it will automatically reconnect to the cluster, even if you destroy and reprovision it completely.

To generate a RSA host key, issue the following command on any computer:

```
ssh-keygen -q -N "" -t rsa -b 4096 -f tezos_tunnel_endpoint_host_rsa_key
```

This value is sensitive, so we will configure it as a Terraform variable. In the terraform module, we are passing it as a variable:

```
  signer_target_host_key=var.signer_target_host_key
```

In terraform.tfvars, use a heredoc to specify the key:

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

It is recommended to map this IP address to a fixed, randomly generated hostname. This way, the remote signer will always manage to connect to the ssh endpoint, even if the cluster gets completely regenerated and gets assigned a new IP. Otherwise, ssh will give a warning that the host key changed, or that the IP is unknown, and it will not automatically reconnect. This is a problem if you do not have another way of logging in to the remote signer.

### Configure the remote signer

The remote signer installation is fully automated using Ansible. The [README of the tezos-remote-signer-os-repo](https://github.com/midl-dev/tezos-remote-signer-os) has more detail on how to set it up.

The remote signer needs two inputs to configure itself:

* the hostname for the ssh endpoint, to be configured in the [tezos-remote-signer](https://github.com/midl-dev/tezos-remote-signer-os/blob/master/tezos-remote-signer.yaml) manifest
* the port where the signer endpoint will be forwarded on the cloud. You can pick any port, but it must be globally unique across your deployment if you have several remote signers.

Once complete, you must create a RSA key pair in the signer. ssh to it as `tezos` user, then issue the following command:

```
ssh-keygen
```

Then, take note of the RSA public key:

```
cat /home/tezos/.ssh/id_rsa.pub
```

### Configure the endpoint

You may now declare this remote signer in the terraform parameter `baking_nodes`.

The `authorized_signers` list takes signer maps consisting of the following key/value pairs:

* `ssh_pubkey`: the public key of the signer, generated at the previous step
* `signer_port`: the port on the remote signer that is being forwarded to the cluster. This can be identical across signers, we recommend to leave it at the default value of 8443.
* `tunnel_endpoint_port`: the port on the load balancer IP address that the ssh forwarder on the signer connects to. This must be unique to the signer.

```
module "tezos-baker" {
  source = "github.com/midl-dev/tezos-on-gke?ref=v2.0//terraform-no-cluster-create"
  baking_nodes = {
    mynode = {
      mybaker = {
        public_baking_key="edpk..."
        public_baking_key_hash="tz1YmsrYxQFJo5nGj4MEaXMPdLrcRf2a5mAU"
        ledger_authorized_path="ledger://my-four-key-words/ed25519/0h/1h",
        authorized_signers : [
                { "ssh_pubkey" : "ssh-rsa AAAAB<snip>==",
                  "signer_port" : 8443,
                  "tunnel_endpoint_port" : 51756 } ]
      }
    }
  }
  signer_target_host_key=var.signer_target_host_key
}
```

Then `taint` and `apply` the terraform module.

### Test the endpoint

Once Terraform has deployed, you should be able to ssh from the signer to the endpoint.

Test it by sshing to the signer as `tezos` user, then:

```
ssh -p 51756 signer@<tunneling endpoint hostname>
```

The first time, you need to type `yes` to add this host to the known hosts.

If you see the message `This account is not available`, it means that the connection has been succesfully established.

If you are using alerting, and if the Ledger is connected and the baking app is on, the `NoRemoteSigner` alert should clear at this point.

### Remote signer monitoring

See [monitoring](monitoring-alerting) section.
