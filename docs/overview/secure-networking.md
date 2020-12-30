# Securing Peer Chaincode Interactions

The Chaincode process and the Peer interact over a GRPC connection, this communication can be secured with TLS since it may occur over an insecure network.

There are a variety of attacks that may occur if the network is unsecure:

- __Eavesdropping__: made possible by someone spoofing the GRPC connection and observing the invocation of transactions as well as the proposal responses.
- __Peer Impersonation Attacks__: this can be achieved by altering the configuration of the chaincode process and luring it to connect to a malicious process pretending to be the peer.
- __Chaincode Impersonation Attacks__: this can be achieved by luring the peer into believing that the connected process is a legitimate chaincode, but instead it is malicious process.

All these attacks have different effects and either provide visibility to information that may be confidential and cause damage to the network by introducing a malicious entity in the network. In order to prevent these attacks as of Hyperledger Fabric release 1.4 there are a number of measures that have been introduced:

- Distinct Chaincode Support Service for the Peer (host:port).
- TLS Support for the Interaction Chaincode-Peer.
- Dynamically Configurable Client Certificates for the Chaincode Process.

The first two measures work in concert with the network architecture to protect and address mostly the first two attacks. The third approach, combined with the previous two, is used to prevent chaincode impersonation attacks.

## Distinct Chaincode Support Service for the Peer (host:port)

This measure is effective if combined with the propert network architecture in production environments. It separates the communication between the chaincode process and the peer on different address (host:port) of the peer. This separation allows for setting up a private and secure network where the chaincode process is deployed, thus making inaccessible to entities other than the peer the chaincode process.

This measure is as effective as the security of the private network it relies upon. If the network is secure and not accessible to others an attacker needs to first gain access to the secure network to perform any of the attacks.

There are two configuration parameters that control this setting:

- `chaincodeListenAddress`: this is the property in the YAML configuration file of the peer that determines where the peer expects to receive connection from chaincode processes.
- `chaincodeAddress`: this is the property in the YAML configuration file used by the chaincode process spawned by the peer to connect to the peer.

The second parameter is optional and defaults to  `chaincodeListenAddress` if not specified. If this is not specified it will be set to the listen address of the peer.

!!! Note
    This setup does not apply to the development mode, which is not considered a production deployment and it is designed to provide access to external agents (e.g. the developer) to the network used to connect the chaincode process and the peer.

## TLS Support for the Interaction Chaincode-Peer

This measure ensures that the chaincode process knows the identity of the Peer it is going to connect to and can veify it during the setup of the initial connection.

Again, this measure is as good as the security of the network architecture of the peer deployment. In order for it to work it requires that the TLS certificate of the Peer provided to the chaincode process cannot be accessed, manipulated, or substituted with a malicious one. In a production operation mode the likelyhood of manipulating the chaincode process configuration is significantly reduced: the chaincode proces life-cycle is managed by the peer and therefore compromising the chaincode configuraiton would require compromising the peer itself.

## Dynamically Configurable Client Certificates for the Chaincode Process

This measure ensures that while operating in production mode, the peer does not accept connections that do not provide a client certificate associated to a chaincode container previously launched. The client certificates are created when the runtime environment for the chaincode is built and injected into it. Moreover, each of these certificates has a time window associated to it, to minimise the exposure of peer to malicious connections that have managed to access the client certificate.

This measure works in conjunction with the previous two as it requires a TLS setup and it is further secured by having a separate address to accept connections from the chaincode processes. With these measures in place it would become very hard for an attacker to impersonate a chaincode process: it would require to compromise the peer and access the dynamically created certificate or eventually access the chaincode container after its launch to steal the certificate.
