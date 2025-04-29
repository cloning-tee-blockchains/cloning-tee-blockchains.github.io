---
title: Case Studies
feature_text: |
  # Case Studies
  ### Forking TEE-based Blockchains in the Wild
excerpt: "Case studies demonstrating the impact of forking attacks against TEE-based blockchains"
aside: true
sidebar: case_study
---


We demonstrate the impact of forking attacks against the enclave in TEE-based blockchains with three case studies. For our case studies, we selected three (almost) production-ready systems: [Phala](#phala), [Ten](#ten), and [Secret Network](#secret-network). 

### Phala

[Phala](https://phala.network/) is a Layer 1 blockchain built on the Substrate framework and integrated with the Polkadot network. It uses TEEs for off-chain confidential smart contract execution. Phala features two types of nodes: Gatekeepers and Workers. Gatekeepers participate in the consensus process and manage contract keys, while Workers execute the smart contracts.

Phala utilizes the Authority-round (Aura) consensus algorithm, where leaders are selected to create new blocks, though currently, developers control the leaders. The Worker nodes process smart contract transactions, securely sealing contract states to ensure fault tolerance. Here, each smart contract is assigned to a single worker executing it. Additionally, Phala incentivizes contract queries with no gas fees, resulting in fast responses.

As each smart contract is only executed by one enclave, Phala’s worker nodes use a heartbeat mechanism to maintain regular updates. As part of this process, worker enclave's compute a function of blockchain metadata and their public key to determine whether or not to issue heartbeat. This function is designed such that each block contains approximately 20 heartbeats, enabling efficient verification of the heartbeats through Gatekeepers. 

<a id="cloning-attack-on-contract-queries-phala"></a>

#### Cloning Attack on Contract Queries

While heartbeats contain the block height (and, as such, could be used for timestamping), enclaves do not check whether they receive regular heartbeats (or acknowledgments) from others. This check is only done by the Gatekeeper, which allows enclaves to be cloned and even isolated from the rest of the network.

Phala contract queries are encrypted with a temporary symmetric key, which includes the contract’s address and the raw query. Imagine a simple scenario: a smart contract with address *a* has a boolean variable initially set to *False*. After processing a transaction *tx*, the variable changes to *True*, so that any query to address *a* should now return *True*. 

However, an adversary could clone the enclave and disable the *pherry relayer* connecting it to the blockchain, isolating this clone. Without transaction *tx*, the isolated clone retains *False* as its state. If a client sends a query to the smart contract at address *a* (step 1 in the figure below), the adversary could forward the request to the isolated clone (step 2), which decrypts the query and responds with *False* (step 3), thus delivering outdated information.

<img src="/assets/figures/attack_phala.svg" alt="Overview of the cloning attack on on Phala. A malicious worker clones the enclave running the smart contract. It then prevents the clone from receiving state updates and answers contract queries with an outdated state." style="display: block; margin-left: auto; margin-right: auto; width: 75%;"/>

<!--object data="/assets/figures/attack_phala.pdf" type="application/pdf" width="80%">
    <embed src="/assets/figures/attack_phala.pdf">
        <p>This browser does not support PDFs. Please download the PDF to view it: <a href="/assets/figures/attack_phala.pdf">Download PDF</a>.</p>
    </embed>
</object-->


<a id="suggested-countermeasure-phala"></a>

#### Suggested Countermeasure

Phala’s heartbeat mechanism presents a valuable opportunity to add a [timestamping](/#serializing-state) feature that could help enclaves to self-detect forking attacks. By leveraging heartbeats, all enclaves could stay synchronized with the current block height, reflecting the most recent state. Our suggestion is to have enclaves exchange these heartbeats over a dedicated peer-to-peer (P2P) network, ensuring they regularly receive heartbeat messages from others and can detect if they become isolated.

Additionally, we recommend that enclaves include the latest block height as a timestamp in their responses to all contract queries. This approach shifts some responsibility to the requesting client, who would need to verify that the output matches the latest state and is, therefore, valid.

<a id="responsible-disclosure-phala"></a>

#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to Phala and suggested countermeasures to its developers. Phala has acknowledged the reported vulnerability and published a [blog post](https://phala.network/posts/worker-node-cloning-attacks) detailing their planned mitigation steps based on our recommendations in the paper. They are currently working on implementing the recommended fixes. Once the update is released, we will provide an update that includes the fixed version.

---

### Secret Network

The [Secret Network](https://scrt.network/) (SN) is a Layer 1 blockchain built on the Cosmos SDK. In SN, each node leverages a TEE to securely execute smart contracts. When a transaction is sent to invoke a smart contract, it’s encrypted using a public key shared across the network, allowing any SN enclave to decrypt and process it. Once selected for inclusion in the next block, the transaction is decrypted, executed within the enclave, and its output is re-encrypted with the sender's public key before being added to the blockchain. To facilitate quick and gas-free access to contract state, SN enclaves offer an HTTP endpoint that clients can use to directly query smart contract data.


<a id="cloning-attack-on-contract-queries-secret"></a>

#### Cloning Attack on Contract Queries

Similar to Phala, SN contract queries are encrypted with a temprory symmetric key. However, not all parts of the query are encrypted. For instance, the code hash and raw contract query are encrypted, the contract address remains in plaintext. This design allows an adversary to alter the contract address in a query, redirecting it to another instance of the same contract (as verified by the code hash) but with a different state.

Consider a smart contract with address *a* stores a counter variable initially set to *x*. After processing a transaction *tx*, the counter is updated to *x+1*, meaning that queries to *a* should now return *x+1*. An adversary, however, could clone this contract to a new address *a'*, which did not process *tx* and therefore still holds the counter value *x*.

In this scenario, if a client issues a query to the smart contract at address *a'* (step 1 in the figure below), the adversary could intercept the HTTP request (step 2) and change the address to *a'* (step 3). Consequently, the query would be processed by the cloned contract at *a'*, which decrypts the query and responds with *x* (step 4), resulting in incorrect data being returned to the client.

<img src="/assets/figures/attack_secret.svg" alt="Overview of the cloning attack on the Secret Network. A malicious Proxy PM in the network changes the contract address in the client’s query to return the state of a different instance with the same code." style="display: block; margin-left: auto; margin-right: auto; width: 75%;"/>



<a id="suggested-countermeasure-secret"></a>

#### Suggested Countermeasure

In the Secret Network (SN), each smart contract is assigned an instance-specific ID (contract address). In other words, two contract clones (same binary, same machine) will get different IDs, similar to [ephemeral IDs](/#ephemeral-identities). However, this ID is not bound to the messages exchanged with clients. By cryptographically binding the contract ID to the query, the enclave could verify it is the intended receiver.

However, this solution alone does not mitigate rollback attacks. [Jean-Louis et al.](https://eprint.iacr.org/2023/378.pdf) suggest implementing a proof-of-publication to ensure that transactions are securely committed and ordered on-chain before execution, which effectively [serializes transactions](/#serializing-state). Another potential solution would be to use TEEs to maintain a secure record of the active TEEs and their ephemeral IDs within the network.

<a id="responsible-disclosure-secret"></a>

#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to the Secret Network and suggested countermeasures to its developers. The Secret Network has acknowledged the reported vulnerability. They are currently working on implementing the mitigations based on our recommendations. Once the update is released, we will provide an update that includes the fixed version.

---

### Ten

[Ten](https://ten.xyz/) (formerly known as *Obscuro*) is an L2 solution built on Ethereum. In Ten, each node uses a TEE to execute transactions confidentially. The TEE seals both cryptographic keys and smart contract states, allowing nodes to restart without needing re-enrollment. Transactions are encrypted with a network-wide public key and can be decrypted by any Ten enclave.

To maintain order, Ten employs a custom *Proof-of-Block-Inclusion (POBI)* consensus protocol. The enclave references the latest L1 block to extract the previous rollup and generates the next one, which includes a random nonce. Among the submitted rollups, the one with the lowest nonce is committed to the L1 layer through a dedicated Ethereum smart contract. This design protects Ten enclaves from rollback attacks, as any rollup created from an outdated L1 block is automatically discarded by the L1 contract.

#### Cloning Attack on Block Generation

While Ten’s architecture is designed to resist rollback attacks, its POBI consensus protocol remains vulnerable to cloning attacks. In the POBI protocol, Ten enclaves each generate a random nonce, and the enclave with the lowest nonce is given the authority to propose the next rollup. By creating multiple clones of the same enclave, an adversary can increase the likelihood of producing the lowest nonce and thus gain control over the rollup proposal process.

The attack unfolds as follows: the adversary starts a Ten enclave that completes enrollmen and seals its cryptographic keys to disk. Then, the adversary clones enclave on the same machine, allowing both enclave instances to access to the sealed state. When a new block is received from an L1 node (step 1 in the figure below), the adversary feeds it to both clones (step 2). Each enclave generates a random nonce, *N* and *N'*, respectively, and includes it in the rollup proposal (step 3). The adversary then selects the rollup with the lowest nonce and submits it to the L1 layer (step 4). This cloning attack allows the adversary to artificially increase their chances of achieving the lowest nonce.

<img src="/assets/figures/attack_ten.svg" alt="Overview of the cloning attack on on Ten. An adversary increases the chances of proposing the next block by running two enclave clones and choosing the output with the lowest nonce." style="display: block; margin-left: auto; margin-right: auto; width: 75%;"/>


<a id="suggested-countermeasure-ten"></a>

#### Suggested Countermeasure

Ten implements stateful enclaves, incorporating a rollback detection mechanism by serializing state by means of the block hash. If a stale block hash is used, the L1 layer will reject the enclave's commit request. However, preventing cloning attacks in such stateful solutions requires the incorporation of additional mechanisms. An effective anti-cloning mechanism in this particular case would be to use enclave-specific [ephemeral identities](/#ephemeral-identities). In particular, the L1 contract handling rollups can be easily modified to keep track of the (ephemeral) identities of the TEE enclaves.

<a id="responsible-disclosure-ten"></a>

#### Responsible Disclosure

On July 10, 2024, we responsibly disclosed our findings to Ten and suggested countermeasures to the developers. Ten has acknowledged the vulnerability as a potential attack vector and indicated that they will take steps to address the issue based on our recommendations.
