# Shim

The shim is responsible for starting and coordinating the interaction between the `Chaincode` implementation and the peer the process is connected to. The coordination and setup logic is implemented in the [shim.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/shim.go) and the main method of the package is `shim.Start(Chaincode)`.

The setup procedure is as follows:

- retrieval of the chaincode name in the format `name:version` from the environment;
- retrieval of the configuration (i.e. peer address and port, GRPC settings);
- initialisation of the bidirectional communication stream with the peer; and
- setup of the "chat" with the peer.

The chat with the peer is implemented in the `chatWithPeer(chaincodename string, stream PeerChaincodeStream, cc Chaincode)` that is responsible for:

- initialising an instance of the chaincode handler with the bidirectional stream and the chaincode implementation
- sending the `REGISTER` message to register the chaincode process with the peer
- initialising the message and error channels
- initialising the message receving loop

The shim package also provide another main entry point for the execution of chaincode. This is the the `StartInProc(chaincodename string, stream ClientStream, cc Chaincode)`. This method is used for the execution of system chaincodes within the peer. The logic implemented here, eventually leads to calling `chatWithPeer(...)`. The only difference is the provision of the stream that connects the chaincode to the peer and the resolution of the chaincode name, which in this case is passed as an argument.
