---
title: The Forking Way
feature_text: |
  # The Forking Way
  ### When TEE meets Consensus
excerpt: "Overview of forking attacks against TEE-based blockchains"
aside: true
paperlink: true
---

### What are TEE-based Blokchains?

TEE-based blockchains are blockchain networks that incorporate **Trusted Execution Environments (TEEs)** to enhance data privacy, reduce communication complexity, and boost scalability. In traditional blockchains, decentralized systems achieve consistency among nodes through a Layer One (L1) consensus protocol. This protocol ensures that the majority of (honest) nodes agree on a shared state by requiring all state data and updates to be publicly accessible so that nodes can validate the current state.

**The Role of TEEs:** 
Trusted Execution Environments (TEEs) provide a secure hardware-based layer that protects sensitive computations. By creating an isolated sandbox, or *enclave*, TEEs prevent unauthorized access to runtime memory and ensure that only verified code is executed. This secure environment allows TEE-based blockchains to offer state confidentiality while minimizing the data exchange requirements and enhancing scalability.

TEE-based blockchains use TEEs in four key ways:

- **TEE-Based Smart Contracts** – TEEs execute smart contracts securely, ensuring that sensitive data remains confidential.
- **TEE-Based Consensus Protocols** – TEEs streamline and scale consensus processes, sometimes by providing a secure leader election mechanism.
- **TEE-Based Layer 2 (L2) Solutions** – TEEs implement private L2 solutions, enabling performance and functionality beyond the capabilities of L1.
- **TEE-Based Applications** – TEEs secure interactions with blockchain applications, including key management and secure data retrieval.

Explore more about these approaches [here](/tee_blockchains/).





### Forking Attacks against TEEs

While TEEs add a layer of security, they are vulnerable to forking attacks, which exploit limitations in TEE functionality. The two types of forking attack are:

- **Rollback Attacks** – Attackers exploit the lack of "freshness" in TEE-sealed data, restarting the enclave with an outdated sealed state to reverse previous updates.
- **Cloning Attacks** – Attackers start multiple instances of the same enclave application with different input sets, causing inconsistencies in state.

These forking attacks can compromise the consistency guarantees of TEEs, requiring additional mechanisms to prevent such attacks.





### How TEE-based Blockchains Prevent Forking Attacks

Consensus protocols, especially in blockchains, are designed to ensure a total ordering of events, meaning every transaction and state update is recorded in a consistent sequence across all nodes. Due to this characteristic, TEE-based applications can naturally rely on blockchains to counter forking attacks.

But let's see how TEE-based blockchains acutally handle this "mariage". Here is an overview of strategies in SGX-based blockchains that (can) prevent forking attacks against the enclave and their pitfalls.

#### Stateless Enclaves

Stateless enclaves, as the name implies, doperate without storing previous computation states. Each time a stateless enclave is restarted, it generates output based solely on the current input without accessing any previously sealed state. This design naturally prevents rollback attacks, as there is no historical state that can be reverted to. However, it introduces two primary limitations: (1) stateless enclaves clearly restricts the type of applications that can be deployed since they do not support persistent state storage., and (2) they remain vulnerable to cloning attacks. For instance, if a randomized computation is performed, an adversary can launch multiple clones and selectively forward the most favorable result.

<div class="boxed">
<b>Takeaway 1 - Stateless Enclaves.</b> 
Using enclaves that do not keep a persistent state protects against rollback attacks by design. However, stateless enclaves limit the expressiveness of the TEE application and do not deter cloning attacks when the TEE application is non-deterministic.
</div>


<p></p>

#### Ephemeral Identities

In TEE-based blockchains, enclaves typically have unique identities that help maintain security and trustworthiness in the network. Most TEE-based blockchains rely on long-lasting identities for enclaves. In this model, each enclave is identified by a unique public key. To ensure persistence, the key pair is securely sealed to disk, allowing the enclave to retain its identity even after a restart. Other TEE-based blockchains use ephemeral identities, where the enclave generates a fresh  key pair upon restart and uses the public key as an identifier. This approach helps to deter cloning attacks by enabling a clear distinction between multiple enclave instances. If the application logic cannot tolerate multiple enclave instances, effective key management is essential to track legitimate enclave instances. While ephemeral identities are effective in distinguishing enclave instances and deterring cloning, they do not prevent rollback attacks.

<div class="boxed">
<b>Takeaway 2 - Ephemeral Identities.</b> 
Identifying each enclave by means of an ephemeral ID (i.e., renewed at restart) can prevent cloning attacks. In settings where the state needs to persist, one should additionally rely on antirollback mechanisms.
</div>


<p></p>

#### A Fixed Set of Clients

Some TEE-based blockchains enforce restricted access to smart contracts by allowing only a fixed set of trusted clients to interact with the enclave. In this model, each client maintains a local representation of the system state. The clients exchange and compare their view with the enclave to detect inconsistencies. This shared state verification helps detect rollback attacks, but the system remains vulnerable to cloning attacks. An adversary could launch multiple clones of the enclave, each running the same randomized computations independently, and select the most favorable result. This method has limitations related to fault tolerance and reconfiguration. Managing offline or malicious clients and allowing for new or departing clients can be challenging and resource-intensive.

<div class="boxed">
<b>Takeaway 3 - Fixed Set of Clients.</b> 
Relying on a fixed and mutually trusted set of smart contract clients can prevent rollback attacks; however, it cannot prevent cloning attacks if the enclave is non-deterministic.
</div>


<p></p>

#### Serializing State

State serialization methods rely on the blockchain’s ordering layer to secure and organize enclave states. This approach has three primary techniques, which can also be combined:

- **Transaction replay from the ledger:** Instead of using local sealing, the enclave rebuilds state by replaying all past transactions from the blockchain. This can prevent rollback attacks if the entire transaction history is retrieved, and protect against cloning if blockchain forking is prevented.
- **Timestamping:** Here, the enclave relies on sealed states but includes metadata like block height and hash as an indicator which blocks are included in the state. By incorporating this metadata as a timestamp, the requesting client can validate the state’s freshness.
- **Storing state in the blockchain:** In this variant, the enclave periodically seals its state (or a state representation) on the blockchain by writing hashes of input and output states. Here, the consistent layer can check that the new state naturally evolves from the latest stored state. This can deter rollback and cloning attacks by anchoring state changes to the blockchain’s consistent layer.


While state serialization offers strong protection, it is limited by the underlying Layer 1 (L1) blockchain’s capabilities. Throughput cannot exceed that of the L1, which may limit update frequency. Additionally, permissionless blockchains like Ethereum and Bitcoin provide only eventual consistency, which can weaken these protections in the event of blockchain forks. For full functionality, the enclave must receive complete blockchain history by either directly participating in consensus or connecting to an honest blockchain node which are not trivial to solve. Furthermore, state serialization requires measures to handle randomized computations, as adversaries can still launch clones to achieve favorable outcomes.

<div class="boxed">
<b>Takeaway 4 - Serializing State.</b> 
Serializing the enclave output using a consistent layer (e.g., the consensus layer of blockchains) can prevent rollback and cloning attacks. However, it needs to be combined with ephemeral IDs to prevent cloning attacks when the TEE computations are
non-deterministic.
</div>





<p></p>

### Forking TEE-based Blockchains in the Wild

We demonstrate the impact of forking attacks against the enclave in TEE-based blockchains with three case studies. For our case studies, we chose three (almost) production-ready systems: [Phala](https://phala.network/), [Ten](https://ten.xyz/), and [Secret Network](https://scrt.network/).





### Authors

[Annika Wilde](https://informatik.rub.de/infsec/people/wilde/)\
Tim Niklas Gruel\
Claudio Soriente\
[Ghassan Karame](https://ghassankarame.com/?i=1)





### Responsible Disclosure
We responsibly disclosed our findings on July 10, 2024 to Phala, Ten and the Secret Network, and suggested countermeasures to the developers of these production-ready TEE-based blockchains, respectively.
