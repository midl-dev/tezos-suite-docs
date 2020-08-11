## Initial baker setup

Prerequisites for secure baking are two Ledger Nano S:

* one that is active all the time, signing baking and endorsing operations
* one used for occasionally transfering funds and voting for protocol amendments, otherwise remaining offline

These Ledgers should be used exclusively for Tezos baking. Do not install other apps, and transfer any other cryptocurrency onto them.

### Setting up Ledgers

Upgrade both Ledgers with the most recent firmware available.

Initialize the first Ledger with a new key. Write down the secret seed as prescribed by the documentation. Do not take a picture of it, keep it offline.

On the first Ledger, install the tezos baker app. Do not install any other app.

Initialize the second Ledger with the secret seed that you wrote down from the first Ledger.

The second Ledger is now a clone of the first. It contains the same keys.

On the second Ledger, install the tezos app. Do not install any other app.

Label Ledgers, ideally with printed labels ("My Tezos Baker Signer" / "My Tezos Baker Offline")

NOTE: you may set up the second Ledger later, as it is not necessary to kick off the baking operations

### Get the Ledger mnemonic

To follow the instructions below, you will need to install the Tezos Command Line Interface. [Follow instructions from Tezos documentation](https://tezos.gitlab.io/introduction/howtoget.html).

The simplest way is to install the Docker containers then run the client in [privileged mode with USB volume mounted](https://tezos.stackexchange.com/questions/395/has-anyone-got-ledger-working-within-docker).

Then issue the command `tezos-client list connected ledgers`.

The Ledger URL including the four-animal-word mnemonic will be displayed, together with commands inviting you to import the key.

There are several cryptographic schemes to choose from. We will be using `ed25519`.

### Choose a derivation path

Ledger has the concept of derivation path - instead of just one address, the device stores multiple addresses. They are related to them by a derivation path, but they are undistinguishable by looking at the public key.

For example, the root path is `/0'/0'`, and the first child path is `/0'/1'`.

It is considered good practice to always use a derived path instead of the root path.

Import the key into your client:

```
tezos-client import secret key ledger_tezos "ledger://<mnemonic>/ed25519/0'/1'"
```

Then read the public key hash associated with this Leger key and path:

```
tezos-client list known addresses
```

Write down the Ledger URL and the public key hash as they are necessary to spin up the cloud baker.

### Register as delegate

To start baking, you must sign an on-chain operation indicating the network that you register to get baking and endorsing slots.

When you register, there is a grace period of two full cycles. Then, you get assigned baking and endorsing slots for five cycles in the future. It means that there is approximately a 21 day gap between the moment you register as delegate and your first on-chain validation operation. Hence it is required to perform this operation well in advance of the planned launch date of the baker.

You need either a fully synced local Tezos node, or to connect to a public node. We recommend [Tezos Giga Node](https://tezos.giganode.io/).

Issue the following operation:

```
tezos-client register key legder_tezos as delegate
```

[See Tezos official documentation for more details](https://tezos.gitlab.io/introduction/howtorun.html#register-and-check-your-rights)
