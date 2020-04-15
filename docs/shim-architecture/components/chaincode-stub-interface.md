# ChaincodeStubInterface

The `ChaincodeStubInterface` represents the interface to the transaction execution context for the smart contract. Implementations of this interface are responsible for providing the chaincode implementation with details about the execution context in which a transaction is invoked (i.e. channel id and other metadata) and for interacting with the peer for any talk back (i.e. ledger queries and cross-chaincode invocations).

## Characterisation

The `ChaincodeStubInteface` is defined in the file [interfaces.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/interfaces.go). The capabilities exposes are a combination of:

- methods that resolve locally; and
- methods that involve the interaction with the peer.

The inteface exposes a synchronous behaviour. This means that for all those methods that involve an interaction with the peer the control will be held back until the peer has responded to the request and the corresponding `ChaincodeMessage` has been processed by the handler to unpack the result and create any supporting component needed.

## Capabilities and Message Mapping

The table below provides an overview of all the methods exposed by the interface with a mapping to the corresponding `ChaincodeMessage` type sent, if any. It is respomsibility of the implementation of this interface to honor this contract and send the specified message for a correct execution of the interaction protocol.

| Method                                       | Is Local | Comments                                                                                          | Chaincode Message Type |
|:---------------------------------------------|:--------:|:-------------------------------------------------------------------------------------------------:|:----------------------:|
| GetArgs                                      |   Yes    | Returns the arguments of the transaction invocation (function and args) as an array of bytes.     |           N/A          |
| GetStringArgs                                |   Yes    | Returns the arguments of the transaction invocation (function and args) as an array of strings.   |           N/A          |
| GetFunctionAndParameters                     |   Yes    | Returns the arguments of the transaction invocation (function and args).                          |           N/A          |
| GetArgsSlice                                 |   Yes    | Returns the arguments of the transaction invocation (function and args) as a single byte array.   |           N/A          |
| GetTxID                                      |   Yes    | Returns the unique identifier of the transaction as a string.                                     |           N/A          |
| GetChannelID                                 |   Yes    | Returns the unique identifier of the channel as a string.                                         |           N/A          |
| InvokeChaincode                              |    No    | Invokes another chaincode installed on the same peer.                                             |    INVOKE_CHAINCODE    |
| GetState                                     |    No    | Retrieves the value of a key.                                                                     |        GET_STATE       |
| PutState                                     |    No    | Sets the value of a key.                                                                          |        PUT_STATE       |
| DelState                                     |    No    | Removes the specified key from the world state.                                                   |        DEL_STATE       |
| SetStateValidationParameter                  |    No    | Set the endorsement policy for the specified key (feature: state-based endorsements).             |   PUT_STATE_METADATA   |
| GetStateValidationParameter                  |    No    | Retrieves the endorsement policy for the specified key (feature: state-based endorsements).       |   GET_STATE_METADATA   |
| GetStateByRange                              |    No    | Returns a range iterator over the keys within the identifier range.                               |   GET_STATE_BY_RANGE   |
| GetStateByRangeWithPagination                |    No    | Returns a range iterator over the keys within the identifier range (paginated version).           |   GET_STATE_BY_RANGE   |
| GetStateByPartialCompositeKey                |    No    | Returns a range iterator over the set of keys that match the specified partial key.               |   GET_STATE_BY_RANGE   |
| GetStateByPartialCompositeKeyWithPagination  |    No    | Returns a range iterator over the keys matching the specified partial key (paginated version).    |   GET_STATE_BY_RANGE   |
| CreateCompositeKey                           |   Yes    | Creates a composite key with the supplied arguments.                                              |           N/A          |
| SplitCompositeKey                            |   Yes    | Splits the specified composite key into its components (objectType and attributes).               |           N/A          |
| GetQueryResult                               |    No    | Performs a rich query against the state database and returns a range iterator over the result.    |    GET_QUERY_RESULT    |
| GetQueryResultWithPagination                 |    No    | As above but with pagination.                                                                     |    GET_QUERY_RESULT    |
| GetHistoryForKey                             |    No    | Returns a range iterator over all the historical changes associated to a specified key.           |  GET_HISTORY_FOR_KEY   |
| GetPrivateData                               |    No    | Returns the value of the specified key in the specified collection.                               |        GET_STATE       |
| GetPrivateDataHash                           |    No    | Returns the hash of the value of the specified key in the specified collection.                                | GET_PRIVATE_DATA_HASH  |
| PutPrivateData                               |    No    | Sets the value of the specified key in the specified collection (when the transaction will be committed).      |        PUT_STATE       |
| DelPrivateData                               |    No    | Removes the value of a specified key in the specified collection (when the transaction will be committed).     |        DEL_STATE       |
| SetPrivateDataStateValidationParameter       |    No    | Set the endorsement policy for the specified key in the specified collection (feature: state-based endorsements). |   PUT_STATE_METADATA   |
| GetPrivateDataStateValidationParameter       |    No    | Retrieves the endorsement policy for the specified key in the specified collection (feature: state-based endorsements). |   GET_STATE_METADATA   |
| GetPrivateDataByRange                        |    No    | Returns a range iterator over all the keys in the specified collection within the specified start and end key. |   GET_STATE_BY_RANGE   |
| GetPrivateDataByPartialCompositeKey          |    No    | Returns a range iterator over all the keys in the specified collection matching the specified partial key.     |   GET_STATE_BY_RANGE   |
| GetPrivateDataQueryResult                    |    No    | Performs a rich query against the specified collection and returns a range iterator over the keys matching the given query. |    GET_QUERY_RESULT    |
| GetCreator                                   |   Yes    | Returns the array of bytes representing the X.509 identity of the  agent/user submitting the transaction.      |           N/A          |
| GetBinding                                   |   Yes    | Returns the transaction binding.                                                                               |           N/A          |
| GetTransient                                 |   Yes    | Returns metadata associated to the transaction, which can be used for instance to implement application confidentiality (e.g. crypto material).
|           N/A          |
| GetDecorations                               |   Yes    | Returns metadata added to the transaction by decorators of the peer. These may mutate the chaincode input  as a result of their function.
|           N/A          |
| GetSignedProposal                            |   Yes    | Returns the signed proposal, which contains all the data elements of the transactio proposal.                  |           N/A          |
| GetTxTimestamp                               |   Yes    | Returns the transaction timestamp of what the transaction was created.                                         |           N/A          |
| SetEvent                                     |   Yes    | Sets an event associated to the transaction.                                                                   |           N/A          |
