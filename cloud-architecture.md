# Cloud Architecture

The best known baking tool, Kiln, is a complete solution for home baking, to be installed on physical hardware. MIDL.dev, in contrast, outsources most of the computing to the cloud, increasing reliability and decreasing the operating burden of the baking operations.

## Managed Kubernetes

Tezos Suite deploys the baking operations in a cloud-native way. It eliminates the maintenance burden on the base operating system and avoids configuration drift.

It uses custom containers built on top of official Docker images from the Tezos project.

Kubernetes lets you seamlessly scale your infrastructure. Tezos validation may become more computationally expensive over time, and the size of the blockchain may grow. It is simple to migrate to larger compute instances or bigger disk space without any downtime.

Since the signing operations are of paramount importance, a separate setup takes care of the auxiliary operations such as payout and monitoring.

Tezos mainnet is fully supported as well as Carthagenet and future test networks.

## Secure cloud setup

We use sentry nodes for protection against DDoS attacks on the validator. Security policies are applied in every pod to ensure all traffic is legitimate.

## Fast bringup time

The sentry nodes and private node are brought up from a snapshot for faster turnaround time and disaster recovery.

## Ledgers for key storage

Signing operations on a hardware wallet is best kept in hardware under the baker’s control. We provide a hardened remote signer OS that allows you to securely connect to the cloud setup and sign baking and endorsing operations.

The remote signer is battery backed and has a redundant 4G internet connection to ensure uninterrupted validation operations.

## Monitoring and alerting

The baking setup comes preconfigured with a complete monitoring solution based on Prometheus and Alertmanager.

It will send you a Slack alert based on a variety of scenarios such as:

* remote signer unreachable
* remote signer has lost power
* remote signer has lost primary Internet connection
* private node lost connectivity to the public nodes
* the persistent volume storing the blockchain is getting full

In addition, an external monitoring setup watches the backing operations from the blockchain, and alerts in case of missed blocks and endorsements.

The monitoring setup is fully extensible and configurable. For example, it is possible to add a PagerDuty account for paging.
