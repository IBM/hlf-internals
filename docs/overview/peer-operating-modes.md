# Peer Operating Modes

Hyperledger Fabric provides two different modalities in which the Fabric Peer can be initialised: _development_ and _production_ mode. These two modalities also affects how the interaction between the peer and the chaincode processes is initialised and managed.

Regardless of the operating modality the chaincode will still be subject to three the phases:

- installation onto the peer
- instantiation in a channel
- invocation of transactions

What changes between the two modalities is how the chaincode process is brought to life and who is responsible for its life-cycle.

## Production Mode

The production mode is the standard modality in which the peer is executed. This modality is designed for production deployments and operates under the assumption that the peer will control the life-cycle of the chaincode process. This operating modality works under the assumption that the environment where the peer is deployed is secured and not easily accessible. The chaincode deployement procedure is more strictly controlled and optimised for ease of operations.

In this modality the peer is responsible to initialise and configure the chaincode process as the chaincode is deployed. Only chaincode processes that are spawned by the peer are allowed to interact with and the peer keeps an active registry of all managed processes.

During production mode the life-cycle of the chaincode develops as follows:

- __INSTALL__: this command upload the chaincode package to the peer, the peer stores a local copy of the package and updates the registry of installed chaincodes.
- __INSTANTIATE__: this command triggers the build of the runtime environment for the specified chaincode according to the details defined in the chaincode specification. Once the environment is built the chaincode is launched as docker container and the peer will wait for its connection. Upon connection the peer will invoke the smart contract initialisation function (i.e. `Init(ChaincodeStub)`) to setup the chaincode in the specific channel. This operation is performed only once the chaincode is first instantiated in one channel, subsequent instantiation will reuse the runtime environemnt built and already launched.
- __INVOKE__: this command triggers the simulation of a transaction proposal, which eventually leads to the invocation of a smart contract function in the remote process running the chaincode, via the dispatch method `Invoke(ChaincodeStub)`.

## Development Mode

The development mode is designed to improve the development and debugging experience of smart contract developers. In this modalities the peer does not own the chaincode process and it is configure to accept connections from remote processes that identify themselves as installed chaincodes. This allows developers to improve the development iterations as they can control the chaincode process and therefore make updates to its code without installing a new version in the peer or tearing down the entire network. Moreover, they can run the chaincode as a native OS process from their development environment without the need of packaging the smart contract.

Because the peer is no longer responsible for the life-cycle of the chaincode process, it is responsibility of the developer to initialise and configure the process to connect to a fabric peer correctly.  In this operating modality the phases of the chaincode life-cycle are mapped as follows:

- __INSTALL__: this command uploads to the peer the chaincode package, peer stores a local copy of package and updates the registry of the installed chaincodes.
- __INSTANTIATE__: this command does not trigger the creation of the runtime environment to run the chaincode but simply deploys the chaincode in a channel, which is done simply by invoking the `Init(ChaincodeStub)` method on the smart contract implementation running in the chaincode process.
- __INVOKE__: this command triggers the simulation of a transaction proposal in the same manner as discussed in the standard operating modality.

Because the  peer no longer build and launch the chaincode runtime environment, it is necessary for the developer to start the chaincode process and connect it to the peer once the chaincode package has been installed. This is done by starting the chaincode process with the information about the chaincode name and version and the address of the peer.

```bash
CORE_ID_CHAINCODE_NAME=<name>:<version> CORE_PEER_TLS_ENABLED=false <chaincode-executable-path> -peer.address <host>:<port>
```

The snippet of code above specifies the environment variables that contain the information about the chaincode metadata, the connection mode to the peer, and the hostname and port pair where the peer is  accepting connection from chaincode processes. Note, this port is usually __different__ from the port used by application clients to connect to the peer, and by default set to 7052.

!!! Note
    The information specified in the `CORE_ID_CHAINCODE_NAME` environment variable must match the name and version used in the command `peer chaincode install ...` to install the chaincode in the peer, otherwise the peer will not be able to identify the chaincode and accept its connection request.

More details about how to run the chaincode in development mode can be found in the [fabric samples](https://github.com/hyperledger/fabric-samples/tree/master/chaincode-docker-devmode).
