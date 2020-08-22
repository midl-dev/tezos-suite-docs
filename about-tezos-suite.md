About Tezos Suite
=================

Tezos Suite was started as a personal baking project. We made some early architecural decisions based on what are considered best practices:

* use cloud computing wherever possible
* use Hardware Security Modules
* follow SRE best practices
* easy disaster recovery

Most of the setup is in the cloud, however the signing device and hardware security model are managed by the user.

### Comparison with other baking software

#### Kiln

The main baking software suite, Kiln, is node-centric. We aim to abstract away the concept of node by leveraging Kubernetes and have everything defined as pods, deployments, services, and policies.

#### DIY

You may download and compile the Tezos software and run the baker in a home server or VPS, but it exposes you to:

* hardware failures
* OS maintenance duties
* configuration drift, accumulation of state
