---
title: Case Studies
feature_text: |
  # Case Studies
  ### Forking TEE-based Blockchains in the Wild
excerpt: "Case studies demonstrating the impact of forking attacks against TEE-based blockchains"
aside: true
---


We demonstrate the impact of forking attacks against the enclave in TEE-based blockchains with three case studies. For our case studies, we selected three (almost) production-ready systems: [Phala](#phala), [Ten](#ten), and [Secret Network](#secret-network). 

### Phala

[Phala](https://phala.network/) is a Layer 1 blockchain built on the Substrate framework and integrated with the Polkadot network. It uses TEEs for off-chain confidential smart contract execution. Phala features two types of nodes: Gatekeepers and Workers. Gatekeepers participate in the consensus process and manage contract keys, while Workers execute the smart contracts.

Phala utilizes the Authority-round (Aura) consensus algorithm, where leaders are selected to create new blocks, though currently, developers control the leaders. The Worker nodes process smart contract transactions, securely sealing contract states to ensure fault tolerance. Here, each smart contract is assigned to a single worker executing it. Additionally, Phala incentivizes contract queries with no gas fees, resulting in fast responses.

As each smart contract is only executed by one enclave, Phala’s worker nodes use a heartbeat mechanism to maintain regular updates. As part of this process, worker enclave's compute a function of blockchain metadata and their public key to determine whether or not to issue heartbeat. This function is designed such that each block contains approximately 20 heartbeats, enabling efficient verification of the heartbeats through Gatekeepers. 

#### Cloning Attack on Contract Queries

While heartbeats contain the block height (and, as such, could be used for timestamping), enclaves do not check whether they receive regular heartbeats (or acknowledgments) from others. This check is only done by the Gatekeeper, which allows enclaves to be cloned and even isolated from the rest of the network.

Contract queries in Phala are encrypted with an ephemeral symmetric key. Among other information, the encrypted query contains the smart contract address and the raw contract query. An adversary 

<div style="center">
![Overview of the cloning attack against Phala Worker enclaves.](/assets/figures/attack_phala.png "Sketch of the cloning attack on Phala. A malicious worker clones the enclave running the smart contract. It then prevents the clone from receiving state updates and answers contract queries with an outdated state.")
</div>
```
Test
```
Given the above setting, an adversary can operate two
instances of the pruntime and freely choose the instance to
answer a request. The attack is depicted in Figure 10 in
Appendix C. We assume a simple contract with address a
using a single boolean variable as state, initialized to False.
At a certain point, a transaction tx causes the boolean variable
to be set to True. From this moment, clients issuing contract
queries to the contract at address a should receive True as a
response. However, assume that the adversary creates a clone
of the pruntime. The adversary does not start a pherry relayer
for the clone, effectively isolating it from the network. The first
instance is still connected to the network and issues regular
heartbeats. As the cloned pruntime did not receive tx (it is
isolated from the network), its internal state remains False. At
this stage, a client issues a contract query (step 1 ) to the smart
contract at address a. The adversary forwards the request to
the isolated pruntime instance (step 2 ), which decrypts the
query and provides False as a response to the client (step 3 ).


#### Suggested Countermeasure
Heartbeats in Phala offer a
good opportunity to integrate a timestamping mechanism that
allows enclaves to self-detect forking attacks: (1) they are
authenticated by the enclaves, (2) they contain the block
height, and (3) they are sent regularly every 45 seconds. We
suggest leveraging those heartbeat messages to ensure that all
enclaves are aware of the current block height (and hence the
current state). To this end, we suggest that enclaves exchange
heartbeats via a separate P2P network and check that they
regularly receive heartbeat messages from others (i.e., they
are not eclipsed). A major challenge with this approach lies in
Limitation 6 (existential honesty): enclaves need to ensure they
are connected to at least one honest node to get the latest state
from the network (and reflect it in their heartbeat messages).
This, alone, however, is not sufficient to deter forking
attacks. Here, we suggest that the enclaves include the latest
block height as a timestamp in responses to all contract
queries (as suggested in the whitepaper [13]). This burdens the
requesting client to determine whether the output corresponds
to a fresh state and is, therefore, valid. In case Phala opts to
support randomized contract execution, we suggest the reliance
on ephemeral IDs (cf. Limitation 8) as well.


#### Responsible Disclosure

---

### Secret Network

[Secret Network](https://scrt.network/)

#### Responsible Disclosure

---

### Ten

[Ten](https://ten.xyz/)

#### Responsible Disclosure

We responsibly disclosed our findings to Ten on July 10, 2024, and suggested our countermeasures to the developers of Ten.
