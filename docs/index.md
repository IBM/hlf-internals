# Welcome

This documentation covers the internals of Hyperledger Fabric with a specific focus on _chaincode management and support for new Smart Contract languages_. While developing the support for writing chaincode in Haskell we had to dive into the implementation details of Fabric and understand not only the peer-chaincode interaction protocol but also the dependencies of this functionality from other subsystems. The exploration and analysis of the code-base and its subsequent sense-making into a more cohesive and structured topic eventually led to this project, which now has a life of its own.

Differently from the existing Hyperledger Fabric documentation, which caters more for the end-user and the developer of Smart Contract this documentation covers the internal implementation, its breakdown into subsystems and components, and a detail account of the interactions among them. Therefore, it is quite technical and low-level and primarily addresses the Fabric developer or anyone curious to know how Hyperledger Fabric works, or is tasked to debug it for troubleshooting purposes.

!!! Note
    This documentation would not exist if we didn't venture ourselves into providing support for Haskell as a Smart Contract language. So please check out the [fabric-chaincode-haskell](https://github.com/nwaywood/fabric-chaincode-haskell) project in Github, try it and star it.

## Where To From Here

This documentation is organised into four main sections:

- [Overview](overview/chaincode-execution-model.md): this section provides an overview of the architecture and the key design principles adopted by Hyperledger Fabric to implement the execution and management of smart contracts. It discusses the rationale, benefits, and advantage of the approach and it introduces the the key concepts of shim and chaincode stub.
- [Interaction Protocol](protocol/architecture-components.md): this section dives into the details of the chaincode-peer interaction protocol and discusses the various phases of the protocol, from  the connection establishment to the execution of transactions. It also provides an  overview of the messages exchanged and their meaning.
- [Shim Architecture](shim-architecture/index.md): this section dives into the implementation of the fabric chaincode shim, which is the process that hosts the smart contract and interfaces it with the Fabric peer. The section discusses the architecture, design, its key components and the protocol used to interact with the peer. If you want to know how `GetState`, `PutState`, `GetStateByRange` and other methods work look in here.
- [Peer Architecture](peer-architecture/index.md): this section discusses the internal architecture of the peer, its breakdown into subsystems and components. It details how these are involved and participate in the manageemnt of the life-cycle of the chaincode and the execution of transactions. If you want to know what happens when a client submits a transaction proposal, look here.
- [Support for New Languages](adding-new-languages/index.md): this section provides an overview of the key components that need to be modified and implemented to enable Hyperledger Fabric with the ability of managing chaincode in another language than Java, Node, or Go. If you want to get to the action and see what takes, jump here.

## What You Need To Know

This documentation assumes that the you have a general understanding of Hyperledger Fabric, its key components, and the transaction execution flow, as a user of this platform. If you need a refresh on these concepts, have a look at the [Key Concepts](https://hyperledger-fabric.readthedocs.io/en/release-1.4/key_concepts.html) section on the Hyperledger Fabric documentation.
