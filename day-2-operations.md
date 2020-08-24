# Day 2 operations

### Apply an update

If you have pulled the most recent version of `tezos-on-gke` and wish to apply updates, you may do so with a `terraform taint`:

```
terraform taint null_resource.push_containers && terraform taint null_resource.apply && terraform plan -out plan.out
terraform apply plan.out
```

This will rebuild the containers locally, then do a `kubectl apply` to push the most recent changes to your cluster.

The daemons will restart after some time. However, you may kill the pods to restart them immediately.

### Tezos Protocol update

When the Tezos protocol changes, be sure to edit the terraform variables `protocol` and `protocol_short` to match the new version.

Then, apply the changes. Your baker will restart with the right baking and endorsing daemons.

### Remotely ssh into the remote signers

For remote connectivity and debugging purposes, ssh port 22 for the on-prem remote signers is being forwarded on port [signer port + 1000].

To connect to the signers, forward port [signer port + 1000] from the `tezos-remote-signer-forwarder` locally, then ssh to localhost using your private key associated with the public key injected into the baker during initial setup.
