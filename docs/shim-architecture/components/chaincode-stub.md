# ChaincodeStub

The `ChaincodeStub` is the default implementation of the `ChaincodeStubInterface` and it is the component that connects the Chaincode to the hosting environment by providing the services exposed by the interface.

The responsibilities of the `ChaincodeStub` are the following:

- providing access to the transaction context reconstructed from the `INIT` and `TRANSACTION` messages sent by the peer;
- preparing the data for the `Handler` for chaincode requests back to the peer;  and
- managing the iterators that are returned when the smart contract makes complex or range queries.

The chaincode stub structure and methods are defined in the file [stub.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/stub.go).

## Initialisation

A new chaincode stub is created by the handler for each transaction invocation. Therefore, the stub has only memory of the execution context of the current transation. Upon the reception of an `INIT` or `TRANSACTION` message, the handler unpacks the payload and invokes the following function to create the stub:

```go
  func newChaincodeStub(handler *Handler, channelID, txid string, input *pb.ChaincodeInput, signedProposal *pb.SignedProposal) (*ChaincodeStub, error) { ... }
```

The method returns a pointer to the `ChaincodeStub` structure that is shown in the listing below:

```go
type ChaincodeStub struct {
    TxID                       string
    ChannelID                  string
    chaincodeEvent             *pb.ChaincodeEvent
    args                       [][]byte
    handler                    *Handler
    signedProposal             *pb.SignedProposal
    proposal                   *pb.Proposal
    validationParameterMetakey string

    // Additional fields extracted from the signedProposal
    creator   []byte
    transient map[string][]byte
    binding   []byte

    decorations map[string][]byte
}
```

The fields `TxID`, `ChannelID`, `args`, `signedProposal`, `proposal`, `creator`, `transient`, `binding`, `decorations`, and `validationParameterMetaKey` are extracted by the message that triggered the execution of the transaction, while the `chaincodeEvent` is a place-holder to maintain the last event triggered by the smart contract during the transaction invocation.

The initialisation of the stub entails the validation of the signed proposal (if not null) to ensure that the proposal has a correct header and to compute the binding for the chaincode message. Normal chaincode invocation have a non-null signed proposal, while the system chaincodes can be initialised with a null proposal.

## Interfacing

The `ChaincodeStub` maintains a pointer to the `Handler` that has created it and uses such reference to implement all the services that require talking back to the peer. Most of the methods of the stubs are simple pass-through to the handler, which manages the interaction with the peer and packaging of arguments in corresponding chaincode messages. Here is an example of the complexity of the methods implemented to retrieve the both the public and private state for a channel:

```go
func (s *ChaincodeStub)GetState(key string) ([]byte, error) {

   collection := ""
   return s.handler.handleGetState(collection, key, s.ChannelID, s.TxID)
}

func (s *ChaincodeStub)GetPrivateData(collection string, key string) ([]byte, error) {

   if collection == "" {
      return nil, fmt.Errorf("collection must not be an empty string")
   }
   return s.handler.handleGetState(collection, key, s.ChannelID, s.TxID)
}
```

With a few exceptions, the majority of the methods of the stub are this simple. As shown in the listing one of the roles of the stub is rely a set of core methods of the handler to provide a rich interface to the smart contract. In this case, the listing shows how the retrieval of both public aivate data relies on the same methods of the handler, which is invoked with different parameters.

A capability that sees the `ChaincodeStub` playing an essential role is are _range_ and _history queries_. For these functions exposed by the `ChaincodeStubInterface` the stub implements and manage the iterators that are retrieved by the corresponding methods and coordinates their interaction with the handler to fetch new batches of data once the local cache has been consumed. The section [Range Queries and Iterators](../interaction-flow/range-queries.md) will discuss these capabilities in detail.
