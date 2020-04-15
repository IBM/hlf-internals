# Protocol Messages Overview

The table below shows a mapping to the of the various message types to the corresponding type of payloads that the protocol use to exchange information. Intuitively, the name of the constant of the message type provides insights on what type of information the payload conveys and what type of operation it is associated to.

| Message Type             | Byte Value | Meaning                                                                      | Sent By | Payload Type                              |
:--------------------------|:----------:|:-----------------------------------------------------------------------------|:-------:|:------------------------------------------|
| UNDEFINED                | 0          | Identifies an unrecognised message (default value for type).                 | N/A     | NIL _(not relevant)_                      |
| REGISTER                 | 1          | Registers a chaincode process with the peer.                                 | shim    | ChaincodeID                               |
| REGISTERED               | 2          | Acknowledges the registration of the chaincode process.                      | peer    | NIL                                       |
| INIT                     | 3          | Initialises the chaincode in the specified channel.                          | peer    | ChaincodeInput                            |
| READY                    | 4          | Communicates to the chaincode to enter the ready state.                      | peer    | NIL                                       |
| TRANSACTION              | 5          | Invokes a transaction (smart contract method).                               | peer    | ChaincodeInput                            |
| COMPLETED                | 6          | Successful response to an INIT or TRANSACTION message.                       | shim    | Response                                  |
| ERROR                    | 7          | Communicates an error, as a result of the processing of a previous message.  | both    | _Varies (e.g. Error, string)_             |
| GET_STATE                | 8          | Retrieves the value for the specified state (i.e. key).                      | shim    | GetState                                  |
| PUT_STATE                | 9          | Sets the value for the  specified state (i.e. key).                          | shim    | PutState                                  |
| DEL_STATE                | 10         | Removes the value for the specified key from the current view of the ledger. | shim    | DelState                                  |
| INVOKE_CHAINCODE         | 11         | Invokes a chaincode installed on the same peer and deployed on a channel.    | peer    | ChaincodeSpec                             |
| RESPONSE                 | 13         | Responds to a chaincode invocation or a ledger request.                      | peer    | _Varies based on type of message replied_ |
| GET_STATE_BY_RANGE       | 14         | Retrieves the values for a subset of the keys.                               | shim    | GetStateByRange                           |
| GET_QUERY_RESULT         | 15         | Performs a rich query against the ledger database.                           | shim    | GetQueryResult                            |
| QUERY_STATE_NEXT         | 16         | Retrieves the next page of keys from the resultset.                          | shim    | QueryResponse                             |
| QUERY_STATE_CLOSE        | 17         | Closes the iteration over the states of the rich query.                      | shim    | QueryResponse                             |
| KEEPALIVE                | 18         | Keep alive message sent by the peer and replied by the shim.                 | both    | NIL                                       |
| GET_HISTORY_FOR_KEY      | 19         | Retrieves the historical values of a given key.                              | shim    | GetHistoryForKey                          |
| GET_STATE_METADATA       | 20         | Retrieves the state metadata (used for state-based endorsements).            | shim    | GetStateMetadata                          |
| PUT_STATE_METADATA       | 21         | Sets the state metadata (used for state-based endorsements).                 | shim    | PutStateMetadata                          |
| GET_PRIVATE_DATA_HASH    | 22         | Retrieves the hash of the value associated to the key in a private collection. | shim  | GetState                                  |
