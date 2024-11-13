---
title: TEE-based Blockchains
feature_text: |
  # TEE-based Blockchains
excerpt: "Overview of the main approaches for integrating TEE technology into blockchain systems"
aside: false
sidebar: tee_blockchains
---

TEE-based blockchains utilize Trusted Execution Environments (TEEs) to ensure state confidentiality, reduce communication complexity, and improve scalability. Here are four main approaches for integrating TEE technology into blockchain systems:

#### TEE-Based Smart Contracts

This approach uses TEEs to enhance the confidentiality of smart contract executions. The TEE handles encrypted inputs, processes transactions securely, and returns encrypted outputs, ensuring that sensitive information remains private throughout the process. Some blockchains also enable clients to make direct read requests (queries) to smart contracts, bypassing the ledger to access the contract state securely.

#### TEE-Based Consensus Protocols

In this model, TEEs boost consensus speed and scalability. Certain blockchains use TEEs to facilitate secure leader election, as seen in the "Proof of Luck" consensus protocol, where the TEE provides a trusted source of randomness for block proposer selection. In other designs, the consensus mechanism runs directly within the TEE to achieve even greater scalability.

#### TEE-Based Layer 2 Solutions

TEEs are also used to develop confidential Layer 2 (L2) solutions that address Layer 1 (L1) limitations in performance and functionality. TEEs help implement confidential smart contracts for L1 blockchains or support private operations over transactions, including mixers, payment channels, and cross-chain bridges. These confidential L2 solutions provide scalability and flexibility while maintaining privacy.

#### TEE-Based Blockchain Applications

Finally, TEE-based applications enhance blockchain interaction security by securely storing cryptographic keys, validating block data, and retrieving data for blockchain applications.