Welcome to MIDL.dev Tezos Suite's documentation!
================================================

### The project

MIDL.dev Tezos Suite is a complete solution for Tezos baking using industry best practices.

This documentation covers the following open-source projects:

* [Tezos-on-GKE](https://github.com/midl-dev/tezos-on-gke/) : deploys a secure baker setup in Google Kubernetes Engine in just one command
* [Tezos-Auxiliary-Cluster](https://github.com/midl-dev/tezos-auxiliary-cluster) : a toolkit for public bakers: monitors baking operations, issues payouts, and generates a dynamic baking website.
* [Tezos-remote-signer-OS](https://github.com/midl-dev/tezos-remote-signer-os/tree/master/tezos-remote-signer) : operating system for a secure, highly available Tezos remote signer setup on a Raspberry Pi connected to a Ledger device


All code is released under the terms of the [Apache License](https://www.apache.org/licenses/LICENSE-2.0).

**This project is supported by the Tezos Foundation.**

Contents
--------

* [Cloud Architecture](cloud-architecture)
* [Remote Signer Architecture](remote-signer-architecture)
* [Initial baker setup](setup_baker)
* [Configure remote signer](deploy-remote-signer)
* [Remote signer setup](setup_remote_signer)
* [Deploy auxiliary cluster](deploy-auxiliary-cluster)
* [Deploy public node only](deploy-public-node)
* [Production readiness](production-readiness)
* [Monitoring and alerting](monitoring-alerting)
* [Day 2 operations](day-2-operations)

Quick start
-----------

To quickly spin up a baking node on the Tezos testnet, follow instructions from the [Tezos-on-GKE README](https://github.com/midl-dev/tezos-on-gke/)
