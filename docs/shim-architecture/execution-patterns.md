# Chaincode Execution Patterns

The fabric shim supports two execution modalities, which control the connection behaviour of the chaincode with the peer:

- __Chaincode as Client__: this is the default modality and the only one used in Hyperledger Fabric v1.4. This is the execution mode that has been discussed in this documentation and features the chaincode process being a client that initiates the connection to the peer.
- __Chaincode as Server__: this modality, not in use yet, is a preview of a future capability and features the chaincode running as a standalone server accepting connection from the peers.

Even though the __Chaincode as Server__ is not currently used it is already available in the implementation of the [fabric-chaincode-go](https://github.com/hyperledger/fabric-chaincode-go) shim. Besides the differences in the initial setup of the process, this modality will eventually invoke the `chatWithPeer(string, ClientStream, Chaincode)` function as happens for the __Chaincode as Client__ execution modality, that is driven by the shim.

In this section we will only focus in the initial setup differences. More details about this execution modality can be found in the [Hyperledger Fabric Documentation](https://hyperledger-fabric.readthedocs.io/en/latest/cc_service.html).

## Chaincode as Server Setup

Starting from v2.0 Hyperledger Fabric supports chaincode deployment outside of Fabric, thus enabling users to manage a chaincode runtime independently from the peer.

This capability becomes particularly useful in cloud deployments such as Kubernetes, where the chaicode can be managed as a service with an independent life-cycle  from the fabric network. In this scenario it is not necessary to build and launch the chaincode for every peer.

To support the chaincode as an external service, it is necessary:

- configure the peer with an external builder and launcher; and
- initialise and run the chaincode shim as a server process.

The implementation of this modality for the chaincode shim, can be found in the following files:

- [shim/chaincodeserver.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/chaincodeserver.go): implementation of the protobuf service that exposes  the Chaincode interface as GRPC service; and
- [shim/internal/server.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/internal/server.go): implementation of a simple GRPC server that is used to register the `ChaincodeServer` service, accept connections from the peers, and relay the control to the configured `Chaincode`.

To initialise the chaincode process in this modality, we wont be using the `shim.Start(Chaincode)` method as a driver for the chaincode process but we will be calling the `ChaincodeServer.Start(Chaincode)` method on a newly created `ChaincodeServer` struct.

```Protobuf

// repository: https://github.com/hyperledger/fabric-protos
// file: peer/chaincode_shim.proto

service Chaincode {
  
  rpc Connect(stream ChincodeMessage) returns (stream ChaincodeMessage)
}
```

The listing above shows the definition of the interface that exposes the Chaincode as a GRPC service. The behaviour is very similar to the __Chaincode as Client__ execution pattern but simply reversed. The interface only declares one method that opens a bidirectional stream with the peer. The implementation of the service bindings for Go can be found in the [fabric-protos-go](https://github.com/hyperledger/fabric-protos-go) repository.

The listing below provides a summarised view of the `ChaincodeServer` implementation with a focus on the key method that setup the chaincode process as a server.

```go
type ChaincodeServer struct {
   CCID string
   Address string
   CC Chaincode
   TLSPRops TLSProperties
   KaOpts *keepalive.ServerParameters
}

func (cs *ChaincodeServer) Connect(stream pb.Chaincode_ConnectServer) error {
   return chatWithPeer(cs.CCID, stream, cs.CC)
}
func (cs *ChaincodeServer) Start() error {
  
  ...
  server, err := internal.newServer(cs.Address, tlsCfg, cs.KaOpts)
  ...
  pb.RegisterChaincodeServer(server.Server, cs)

  return server.Start()
}

```

As shown in the listing the method that is bound to the `Connect` function exposed by the GRPC service simply calls `chatWithPeer(string, ClientStream, Chaincode)` by passing the bidirectional stream just established for the incoming connection and the `Chaincode` implementation that is configured with the server. Once the method is invoked, the control for that connection is passed to message receiving loop that has been previously discussed.

Below is a simple example of how to initialise the process as a server and configure it with a `Chaincode` implementation.

```go
type SimpleChaincode struct {}

func (ssc *SimpleChaincode) Init(stub shim.ChaincodeInterface) *pb.Response { .... }
func (ssc *SimpleChaincode) Invoke(stub shim.ChaincodeInterface) *pb.Response { .... }


func main() {

   ccid := "simplechaincode:1.0"
   server := &shim.ChaincodeServer{ CCID: ccid,
                                    Address: "localhost:9999",
                                    CC: new(SimpleChaincode),
                                    TLSProp: shim.TLSProperties{ Disabled: true }, }

   err := server.Start()
   if err != nil {
       fmt.Printf("Error starting the simple chaincode server: %s", err)
   }
}
```

For the configuration of the connection properties and the security see the section [Securing the Chaincode Process](security.md).
