# Chaincode Events

## Overview

Hyperledger Fabric supports the ability to raise events during the invocation of a chaincode transaction.

Events are __named objects__ that are recorded as part of the transaction execution and eventually bubbled up to the application clients connected to the same channel for their perusal once the transaction is committed.

!!! Note
    Events are ephemeral and are only raised at the transaction commitment time. If the application client is not connected to the channel at that point in time it will loose the opportunity to receive the event. For those applications that rely upon events for the correct operations of the off-chain functions, must implement event persistence within the smart contracts and provide access functions.

Smart contract developers can call the following method to raise events from within the chaincode logic:

```go
func (s *ChaincodeStubInterface) SetEvent(name string, payload []byte) error
```

Only __one event__ is alloiwed per transaction execution. In case of multiple invocations to this method, __only the last one__ will be recorded in the transaction execution record, and therefore raised to the application clients, listening the channel events.

## Implementation

Events are piggy-backed to the peer together wit the `ChaincodeMessage` of type `COMPLETED` (or `ERROR`) at the end of the transaction simulation execution. The listing below represents the structure of the `ChaincodeMessage` and the associated field for the chaincode event.

```go
type ChaincodeMessage struct {

   Type           ChaincodeMessage_Type
   Timestamp      *timestamp.Timestamp
   Payload        []byte
   Txid           string
   Proposal       *SignedProposal

   // Event emitted by the chaincode. Used only with Init or Invoke.
   // This event is then stored with Block.NonHashData.TransactionResult
   ChaincodeEvent *ChaincodeEvent

   ChannelId      string

   // Protobuf marshalling fields
   XXX_NoUnkeyedLiteral struct{}
   XXX_unrecognized     []byte
   XXX_sizecache        int32
}

type ChaincodeEvent struct {
   ChaincodeId     string
   Txid            string
   EventName       string
   Payload         []byte

   // Protobuf marshalling fields
   XXX_NoUnkeyedLiteral struct{}
   XXX_unrecognized     []byte
   XXX_sizecache        int32
}

```

The last event se is locally cached in the stub and if present, then added to the chaincode message by the Handler when composing the response `ChaincodeMessage` at the end of the execution of the `handleInit(...)` and `handleTransaction(...)` methods.

```go

// file: shim/stub.go
//
func (s *ChaincodeStub) SetEvent(name string, payload []byte) error {
   if name == "" {
      return errors.New("event name cannot be empty string")
   }
   s.chaincodeEvent = &pb.ChaincodeEvent{EventName: name, Payload: payload}
}

// file: shim/handler.go
//
func (h *Handler) handleInit(msg *pb.ChaincodeMessage) (*pb.ChaincodeMessage, error) {
    ....
    stub, err := newChaincodeStub(h, msg.ChannelId, msg.Txid, input, msg.Proposal)
    ....
    res := h.cc.Init(stub)
    resBytes, err := proto.Marshal(&res)
    ...
    return &pb.ChaincodeMessage{Type: pb.ChaincodeMessage_COMPLETED, 
                                Payload: resBytes, Txid: msg.Txid, ChaincodeEvent: stub.chaincodeEvent,
                                ChannelId: stub.ChannelID}, nil
}

```

!!! Note
    The listing shows the implementation of the `handleInit(....)` method. The implementation of the of the `handleTransaction(....)` method is identical, except for the invocation of the `h.cc.Invoke(stub)` method.
