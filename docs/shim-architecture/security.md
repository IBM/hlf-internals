# Securing the Chaincode Process

The security of the communication with the peer is managed through a set of environment variables passed to the chaincode process at startup. These are:

- `CODE_CHAINCODE_ID_NAME`: it provides the chaincode process with its identification details that are used to when it register itself with the peer. This parameter is essential for the protocol to work and not only a security switch.
- `CORE_PEER_TLS_ENABLED`: this is the main switch to enable TLS communication with the peer. It can be set to true or false. If false all the other TLS parameter are ignored.
- `CORE_PEER_TLS_ROOT_CERT_FILE`: this is file containing the CA root that signed the TLS certificate that will be presented by the peer.
- `CORE_TLS_CLIENT_KEY_FILE` |  `CORE_TLS_CLIENT_KEY_PATH`: location of the of the private key of the client certificate used by the chaincode process to connect. First file is checked, if not found path is looked up.
- `CORE_TLS_CLIENT_CERT_FILE` | `CORE_TLS_CLIENT_CERT_PATH`: location of the of the client certificate used by the chaincode process to connect. First file is checked, if not found path is looked up.

In addition to these environment variables the `--peer.address` command line flag is used to pass the address (host and port) that exposes in the peer the GRPC service for chaincode support. This is the endpoint that the chaincode process will connect to.

These configuration settings are processed and managed by the `Config` component, whose implementation is located in the [config.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/internal/config.go) file. The listing below shows a summarised version of the file and highlights its key functions.

```go
import (
   "crypto/tls"
   ...h
   "google.golang.org/grpc/keepalive"
)

type Config struct {
   ChaincodeName   string,
   TLS             *tls.Config,
   KaOpts          keepalive.ClientParameters
}

func LoadConfig() (*Config, error) { .... }

func LoadTLSConfig(isserver bool, key, cert, root []byte) (*tls.Config, error) { .... }
```

The function `LoadConfig()` parses the environment variables and generates the overall configuration for the shim. The function `LoadTLSConfig(...)` is called by the `LoadConfig()` function if TLS is enabled and configures the TLS section of the shim configuration.

## TLS Setup

The configuration of the TLS (v1.2) capabilities is sensitive to the chaincode executino patterns, since different certificates need to be provided if the chaincode runs as a client to the peer or as a standalone server.

### Chaincode as Client

This is currently the default and it is enforced by hard-coding the value of the `isserver` flag to `false`.

In this modality, the parameters passed to the `LoadTLSConfig(...)` are interpreted as follows:

- the _root_ certificate passed to the `LoadTLSConfig(...)` function is used to construct the pool of root certificates (i.e. `tlsConfig.RoorCAs`) that the GRPC client should trust to validate the server (i.e. the peer); and
- the _key_ and _cert_ pair represent the client certificate that the chaincode process will present to the peer to establish the connection. 

The resulting `tls.Config` instance is passed to the connection setup function which establishes the GRPC connection with the per and creates a client for the chaincode support service:

```go
func NewClientConn(address string, tlsConf *tls.Config, kaOpts keepalive.ClientParameters) (*grpc.ClientConn, error) ( ... }
```

### Chaincode as Server

This mode is not currently used in version 1.4, but it is a preview of future version of the implementation.

In this modality the configuration parameters for the TLS setup are pulled from the `ChaincodeServer.TLSProps` struct that defines the configuration of the server:

- the _root_ certificate cannot be null and contributes to the creation of the pool of certificates that are checked by the server to validate the client certificates provided by the connecting peers (i.e. `tlsConfig.ClientCAs`). These are required and must be valid (i.e. `tlsConfig.ClientAut=tls.RequireAndVerifyClient`); and
- the _key_ and _cert_ pair provides access to server certificate that will be presented to the incoming connections to identify the chaincode process.

The resulting `tls.Config` instance is passed to the server setup function that starts the chaincode server:

```go
func NewServer(address string, tlsConf *tls.Config, srvKaOpts keepalive.ServerParameters) (*Server, error) { ... }
```

This function is located in the file [server.go](https://github.com/hyperledger/fabric-chaincode-go/blob/master/shim/internal/server.go).
