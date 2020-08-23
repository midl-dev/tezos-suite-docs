# Architecture

The best known baking tool, Kiln, is a complete solution for home baking, to be installed on physical hardware. Tezos-on-GKE, in contrast, outsources most of the computing to the cloud, increasing reliability and decreasing the operating burden of the baking operations.

Signing operations on a hardware wallet is best kept in hardware under the baker’s control. We provide a hardened remote signer OS that allows you to securely connect to the cloud setup and sign baking and endorsing operations.

Since the signing operations are of paramount importance, a separate Kubernetes cluster with separate authentication credentials takes care of the auxiliary operations such as payout and monitoring.

These three projects, taken together, allow you to deploy a secure, best-practices baking operation.

The setup supports Tezos mainnet as well as Carthagenet and future test networks.
