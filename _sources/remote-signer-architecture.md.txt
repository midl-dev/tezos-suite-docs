# Remote signer architecture

The remote signer consists of a computer and a Ledger hardware wallet. The Ledger is always online, and running the Tezos baking app.

The computer can be a laptop, or a Raspberry Pi with battery backup. It supports a redundant 4g connection in case the power or main internet connection goes down.

## Connectivity

The signing device is running the [native remote signer daemon](https://tezos.gitlab.io/user/key-management.html#signer) from the Tezos project. A [wrapper script](https://github.com/midl-dev/tezos-remote-signer-os/blob/master/tezos-remote-signer/templates/usr/lib/python3/tezos-signer-wrapper/signerWrapper.py) adds monitoring endpoints.

The remote signer connects to a cloud endpoint with ssh. When the ssh connection goes down for any reason, it always tries to reestablish it.

Then ssh tunneling is established. The signer endpoint is exposed to the cloud baker and endorser.

When a baking or endorsing signature request comes in, it is tunneled to the remote signer which interacts with the Ledger to sign the bytes. The private keys never leave the Ledger device, so even in case of compromise of the cloud environment or the remote signer itself, the keys are unreachable from the attacker.

Several remote signers are supported within a single cloud environment. Each signer is identified by its ssh public key and its forwarded port number. This data is passed as parameter when spinning up the cloud environment.

The host key of the ssh server that serves as tunneling endpoint must also be passed as parameter. This way, in case of disaster, it is possible to respin the cloud environment from scratch, keeping the same host key, so the remote signer can automatically connect to it once the setup is available again.

Note that **at the outer level**, the signer connects to the cloud environment, not the other way around. This is helpful when the signer is located behind a NAT, it can always connect to the public ssh tunneling endpoint.

## Sshing to the remote signer

In case where the remote signer is in a location where sshing locally is not possible, ssh port 22 is also tunneled. The remote signer also exposes its own ssh port (22) to the tunneling endpoint. By convention, the port for the ssh endpoint is (signer port + 1000).

Kubernetes allows you to forward ports to your workstation. After forwarding the relevant port on the remote signer forwarder, you can ssh to the remote signer locally.
