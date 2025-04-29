---
title: The Forking Way
feature_text: |
  # The Forking Way: When TEEs meet Consensus
excerpt: "The Forking Way: When TEEs meet Consensus (NDSS'25) systemizes TEE-based blockchains and how they mitigate forking attacks against the TEE, including rollback and cloning attacks."
aside: true
paperlink: true
sidebar: "index"
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

#### Mitigation Distribution

The following table shows an excerpt of our analysis of 29 TEE-based blockchains. We previously described that [TEE-based blockchains use TEEs in four key ways](#what-are-tee-based-blokchains): for TEE-based smart contracts, consensus protocols, L2 solutions, and applications. Let's see how the mitigation strategies are adopted across these system categories:

- **Stateless enclaves** are employed by platforms in all four categories.
- **Ephemeral identities** are primarily used by TEE-based Layer 2 solutions to counter cloning attacks. Specifically, five out of 13 platforms in this category utilize ephemeral IDs. In contrast, none of the TEE-based smart contracts uses ephemeral identities.
- **A fixed set of clients** is a technique exclusively employed by TEE-based L2 solutions.
- **State serialization** techniques are utilized across all four system categories. TEE-based smart contracts mostly rely on transaction replay and timestamping, whereas TEE-based consensus protocols and L2 solutions more commonly store their states on the ledger. Many TEE-based blockchain applications use timestamping. Lastly, no TEE-based consensus protocol replays transactions to recover state information, and no enclave in a blockchain application stores its state on the ledger.

If you are interested in a more thorough evaluation of our results, we encourage you to take a closer look into our [paper]().

<p></p>

<style>
table, th, td {
  border:1px solid black;
  padding-left: 5px;
  padding-right: 5px;
}
th {
  background-color: #F2F2F2;
}
.centercol {
  text-align: center;
}
</style>

<div>
<table style="width:100%">
  <tr>
    <th><strong>Project</strong></th>
    <th class="centercol"><strong>Stateless enclaves</strong></th>
    <th class="centercol"><strong>Ephemeral IDs</strong></th>
    <th class="centercol"><strong>Fixed set</strong></th>
    <th class="centercol"><strong>Transaction replay</strong></th>
    <th class="centercol"><strong>Time- stamp</strong></th>
    <th class="centercol"><strong>State on the ledger</strong></th>
  </tr>
  <tr>
    <td class="centercol" colspan="7" style="background-color: #F2F2F2"><strong>TEE-based Smart Contracts</strong></td>
  </tr>
  <tr>
    <td>Azure CCF</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>CONFIDE</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>CreDB</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Ekiden</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>Phala</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Secret Network</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td class="centercol" colspan="7" style="background-color: #F2F2F2"><strong>TEE-based Consensus Protocols</strong></td>
  </tr>
  <tr>
    <td>Crust sWorker</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>ENGRAFT</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>MobileCoin</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Proof of Luck</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>REM</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td  class="centercol" colspan="7" style="background-color: #F2F2F2"><strong>TEE-based Layer 2 Solutions</strong></td>
  </tr>
  <tr>
    <td>COMMITEE</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>FastKitten</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Hybridchain</td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>IntegriTEE</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>Obscuro Mixer</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>PrivacyGuard</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Private Chaincode</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>Private Data Objects</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>ShadowEth</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>Teechain</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Ten</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
  </tr>
  <tr>
    <td>Tesseract</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Twilight</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td  class="centercol" colspan="7" style="background-color: #F2F2F2"><strong>TEE-based Blockchain Applications</strong></td>
  </tr>
  <tr>
    <td>BITE</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>LSKV</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>sgxwallet</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Ternoa Network</td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
  <tr>
    <td>Town Crier</td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol"></td>
    <td class="centercol">&#10004;</td>
    <td class="centercol"></td>
  </tr>
</table>
</div>





<p></p>

### Forking TEE-based Blockchains in the Wild

We demonstrate the impact of forking attacks against the enclave in TEE-based blockchains with three case studies. For our case studies, we chose three (almost) production-ready systems: Phala, Ten, and the Secret Network.

- **[Phala](/case_studies#phala)**  is an L1 blockchain leveraging TEEs for off-chain confidential smart contract execution. By cloning the enclave, an adversary can isolate the cloned instance from the network and provide rogue responses to contract queries.
- **[Secret Network](/case_studies#secret-network)** is an L1 blockchain leveraging TEEs to securely execute smart contracts. By cloning the smart contract, an adversary can return incorrect data in response to contract queries from clients.
- **[Ten](/case_studies#ten)** is an L2 solution leveraging TEEs to provide transaction confidentiality. By cloning the enclave, an adversary can artificially increase their chances to propose the next rollup, breaking fairness guarantees.


Explore the details of these three attacks [here](/case_studies/).





### Responsible Disclosure
We responsibly disclosed our findings on July 10, 2024 to Phala, Ten and the Secret Network, and suggested countermeasures to the developers of these production-ready TEE-based blockchains, respectively. While Ten acknowledged the issue, we still await responses from Phala and the Secret Network.  





### Authors

You can find full details on our study about the use of TEEs in the context of blockchains in our NDSS 2025 paper which is available [here](https://arxiv.org/pdf/2412.00706).

[Annika Wilde](https://informatik.rub.de/infsec/people/wilde/)\
[Tim Niklas Gruel](https://www.timniklasgruel.com/)\
Claudio Soriente\
[Ghassan Karame](https://ghassankarame.com/?i=1)
