---
title: Case Studies
feature_text: |
  # Case Studies
  ### Forking TEE-based Blockchains in the Wild
excerpt: "Case studies demonstrating the impact of forking attacks against TEE-based blockchains"
aside: false
---


We demonstrate the impact of forking attacks against the enclave in TEE-based blockchains with three case studies. For our case studies, we selected three (almost) production-ready systems: [Phala](#phala), [Ten](#ten), and [Secret Network](#secret-network). 

### Phala

[Phala](https://phala.network/) is a Layer 1 blockchain built on the Substrate framework and integrated with the Polkadot network. It uses TEEs for off-chain confidential smart contract execution. Phala features two types of nodes: Gatekeepers and Workers. Gatekeepers participate in the consensus process and manage contract keys, while Workers execute the smart contracts.

Phala utilizes the Authority-round (Aura) consensus algorithm, where leaders are selected to create new blocks, though currently, developers control the leaders. The Worker nodes process smart contract transactions, securely sealing contract states to ensure fault tolerance. Here, each smart contract is assigned to a single worker executing it. Additionally, Phala incentivizes contract queries with no gas fees, resulting in fast responses.

As each smart contract is only executed by one enclave, Phala’s worker nodes use a heartbeat mechanism to maintain regular updates. As part of this process, worker enclave's compute a function of blockchain metadata and their public key to determine whether or not to issue heartbeat. This function is designed such that each block contains approximately 20 heartbeats, enabling efficient verification of the heartbeats through Gatekeepers. 

#### Cloning Attack on Contract Queries

While heartbeats contain the block height (and, as such, could be used for timestamping), enclaves do not check whether they receive regular heartbeats (or acknowledgments) from others. This check is only done by the Gatekeeper, which allows enclaves to be cloned and even isolated from the rest of the network.

Phala contract queries are encrypted with a temporary symmetric key, which includes the contract’s address and the raw query. Imagine a simple scenario: a smart contract with address *a* has a boolean variable initially set to *False*. After processing a transaction *tx*, the variable changes to *True*, so that any query to address a should now return *True*. 

However, an adversary could clone the enclave and disable the *pherry relayer* connecting it to the blockchain, isolating this clone. Without transaction *tx*, the isolated clone retains *False* as its state. If a client sends a query to the smart contract at address *a* (step 1 in the figure below), the adversary could forward the request to the isolated clone (step 2), which decrypts the query and responds with *False* (step 3), thus delivering outdated information.

![Overview of the cloning attack against Phala Worker enclaves.](/assets/figures/attack_phala.png "Sketch of the cloning attack on Phala. A malicious worker clones the enclave running the smart contract. It then prevents the clone from receiving state updates and answers contract queries with an outdated state.")


#### Suggested Countermeasure

Phala’s heartbeat mechanism presents a valuable opportunity to add a timestamping feature that could help enclaves to self-detect forking attacks. By leveraging heartbeats, all enclaves could stay synchronized with the current block height, reflecting the most recent state. Our suggestion is to have enclaves exchange these heartbeats over a dedicated peer-to-peer (P2P) network, ensuring they regularly receive heartbeat messages from others and can detect if they become isolated.

Additionally, we recommend that enclaves include the latest block height as a timestamp in their responses to all contract queries. This approach shifts some responsibility to the requesting client, who would need to verify that the output matches the latest state and is, therefore, valid.


#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to Phala and provided the developers of Ten with suggested countermeasures. As of now, we have not received an acknowledgment of the vulnerability from Phala.

---

### Secret Network

The [Secret Network](https://scrt.network/) (SN) is a Layer 1 blockchain built on the Cosmos SDK. In SN, each node leverages a Trusted Execution Environment (TEE) to securely execute smart contracts. When a transaction is sent to invoke a smart contract, it’s encrypted using a public key shared across the network, allowing any SN enclave to decrypt and process it. Once selected for inclusion in the next block, the transaction is decrypted, executed within the enclave, and its output is re-encrypted with the sender's public key before being added to the blockchain. To facilitate quick and gas-free access to contract state, SN enclaves offer an HTTP endpoint that clients can use to directly query smart contract data.

#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to the Secret Network and provided its developers with suggested countermeasures. As of now, we have not received an acknowledgment of the vulnerability from the Secret Network.

---

### Ten

[Ten](https://ten.xyz/)

#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to Ten and provided the developers of Ten with suggested countermeasures. Ten acknowledged the vulnerability as a potential attack vector.
