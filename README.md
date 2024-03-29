## Go Ethereum

Golang execution layer implementation of the Ethereum protocol.

[![API Reference](
https://pkg.go.dev/badge/github.com/ethereum/go-ethereum
)](https://pkg.go.dev/github.com/ethereum/go-ethereum?tab=doc)
[![Go Report Card](https://goreportcard.com/badge/github.com/ethereum/go-ethereum)](https://goreportcard.com/report/github.com/ethereum/go-ethereum)
[![Travis](https://app.travis-ci.com/ethereum/go-ethereum.svg?branch=master)](https://app.travis-ci.com/github/ethereum/go-ethereum)
[![Discord](https://img.shields.io/badge/discord-join%20chat-blue.svg)](https://discord.gg/nthXNEv)

Automated builds are available for stable releases and the unstable master branch. Binary
archives are published at https://geth.ethereum.org/downloads/.

## Building the source

For prerequisites and detailed build instructions please read the [Installation Instructions](https://geth.ethereum.org/docs/getting-started/installing-geth).

Building `geth` requires both a Go (version 1.19 or later) and a C compiler. You can install
them using your favourite package manager. Once the dependencies are installed, run

```shell
make geth
```

or, to build the full suite of utilities:

```shell
make all
```

## Executables

The go-ethereum project comes with several wrappers/executables found in the `cmd`
directory.

|  Command   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :--------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`geth`** | Our main Ethereum CLI client. It is the entry point into the Ethereum network (main-, test- or private net), capable of running as a full node (default), archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Ethereum network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `geth --help` and the [CLI page](https://geth.ethereum.org/docs/fundamentals/command-line-options) for command line options. |
|   `clef`   | Stand-alone signing tool, which can be used as a backend signer for `geth`.                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|  `devp2p`  | Utilities to interact with nodes on the networking layer, without running a full blockchain.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|  `abigen`  | Source code generator to convert Ethereum contract definitions into easy-to-use, compile-time type-safe Go packages. It operates on plain [Ethereum contract ABIs](https://docs.soliditylang.org/en/develop/abi-spec.html) with expanded functionality if the contract bytecode is also available. However, it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://geth.ethereum.org/docs/developers/dapp-developer/native-bindings) page for details.                                  |
| `bootnode` | Stripped down version of our Ethereum client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks.                                                                                                                                                                                                                                               |
|   `evm`    | Developer utility version of the EVM (Ethereum Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of EVM opcodes (e.g. `evm --code 60ff60ff --debug run`).                                                                                                                                                                                                                                               |
| `rlpdump`  | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp)) dumps (data encoding used by the Ethereum protocol both network as well as consensus wise) to user-friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`).                                                                                                                                                                                |

## Running `geth`

Going through all the possible command line flags is out of scope here (please consult our
[CLI Wiki page](https://geth.ethereum.org/docs/fundamentals/command-line-options)),
but we've enumerated a few common parameter combos to get you up to speed quickly
on how you can run your own `geth` instance.

### Hardware Requirements

Minimum:

* CPU with 2+ cores
* 4GB RAM
* 1TB free storage space to sync the Mainnet
* 8 MBit/sec download Internet service

Recommended:

* Fast CPU with 4+ cores
* 16GB+ RAM
* High-performance SSD with at least 1TB of free space
* 25+ MBit/sec download Internet service

### Full node on the main Ethereum network

By far the most common scenario is people wanting to simply interact with the Ethereum
network: create accounts; transfer funds; deploy and interact with contracts. For this
particular use case, the user doesn't care about years-old historical data, so we can
sync quickly to the current state of the network. To do so:

```shell
$ geth console
```

This command will:
 * Start `geth` in snap sync mode (default, can be changed with the `--syncmode` flag),
   causing it to download more data in exchange for avoiding processing the entire history
   of the Ethereum network, which is very CPU intensive.
 * Start the built-in interactive [JavaScript console](https://geth.ethereum.org/docs/interacting-with-geth/javascript-console),
   (via the trailing `console` subcommand) through which you can interact using [`web3` methods](https://github.com/ChainSafe/web3.js/blob/0.20.7/DOCUMENTATION.md) 
   (note: the `web3` version bundled within `geth` is very old, and not up to date with official docs),
   as well as `geth`'s own [management APIs](https://geth.ethereum.org/docs/interacting-with-geth/rpc).
   This tool is optional and if you leave it out you can always attach it to an already running
   `geth` instance with `geth attach`.

### A Full node on the Görli test network

Transitioning towards developers, if you'd like to play around with creating Ethereum
contracts, you almost certainly would like to do that without any real money involved until
you get the hang of the entire system. In other words, instead of attaching to the main
network, you want to join the **test** network with your node, which is fully equivalent to
the main network, but with play-Ether only.

```shell
$ geth --goerli console
```

The `console` subcommand has the same meaning as above and is equally
useful on the testnet too.

Specifying the `--goerli` flag, however, will reconfigure your `geth` instance a bit:

 * Instead of connecting to the main Ethereum network, the client will connect to the Görli
   test network, which uses different P2P bootnodes, different network IDs and genesis
   states.
 * Instead of using the default data directory (`~/.ethereum` on Linux for example), `geth`
   will nest itself one level deeper into a `goerli` subfolder (`~/.ethereum/goerli` on
   Linux). Note, on OSX and Linux this also means that attaching to a running testnet node
   requires the use of a custom endpoint since `geth attach` will try to attach to a
   production node endpoint by default, e.g.,
   `geth attach <datadir>/goerli/geth.ipc`. Windows users are not affected by
   this.

*Note: Although some internal protective measures prevent transactions from
crossing over between the main network and test network, you should always
use separate accounts for play and real money. Unless you manually move
accounts, `geth` will by default correctly separate the two networks and will not make any
accounts available between them.*

### Configuration

As an alternative to passing the numerous flags to the `geth` binary, you can also pass a
configuration file via:

```shell
$ geth --config /path/to/your_config.toml
```

To get an idea of how the file should look like you can use the `dumpconfig` subcommand to
export your existing configuration:

```shell
$ geth --your-favourite-flags dumpconfig
```

*Note: This works only with `geth` v1.6.0 and above.*

#### Docker quick start

One of the quickest ways to get Ethereum up and running on your machine is by using
Docker:

```shell
docker run -d --name ethereum-node -v /Users/alice/ethereum:/root \
           -p 8545:8545 -p 30303:30303 \
           ethereum/client-go
```

This will start `geth` in snap-sync mode with a DB memory allowance of 1GB, as the
above command does.  It will also create a persistent volume in your home directory for
saving your blockchain as well as map the default ports. There is also an `alpine` tag
available for a slim version of the image.

Do not forget `--http.addr 0.0.0.0`, if you want to access RPC from other containers
and/or hosts. By default, `geth` binds to the local interface and RPC endpoints are not
accessible from the outside.

### Programmatically interfacing `geth` nodes

As a developer, sooner rather than later you'll want to start interacting with `geth` and the
Ethereum network via your own programs and not manually through the console. To aid
this, `geth` has built-in support for a JSON-RPC based APIs ([standard APIs](https://ethereum.github.io/execution-apis/api-documentation/)
and [`geth` specific APIs](https://geth.ethereum.org/docs/interacting-with-geth/rpc)).
These can be exposed via HTTP, WebSockets and IPC (UNIX sockets on UNIX based
platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by `geth`,
whereas the HTTP and WS interfaces need to manually be enabled and only expose a
subset of APIs due to security reasons. These can be turned on/off and configured as
you'd expect.

HTTP based JSON-RPC API options:

  * `--http` Enable the HTTP-RPC server
  * `--http.addr` HTTP-RPC server listening interface (default: `localhost`)
  * `--http.port` HTTP-RPC server listening port (default: `8545`)
  * `--http.api` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--http.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--ws.addr` WS-RPC server listening interface (default: `localhost`)
  * `--ws.port` WS-RPC server listening port (default: `8546`)
  * `--ws.api` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--ws.origins` Origins from which to accept WebSocket requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to
connect via HTTP, WS or IPC to a `geth` node configured with the above flags and you'll
need to speak [JSON-RPC](https://www.jsonrpc.org/specification) on all transports. You
can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based
transport before doing so! Hackers on the internet are actively trying to subvert
Ethereum nodes with exposed APIs! Further, all browser tabs can access locally
running web servers, so malicious web pages could try to subvert locally available
APIs!**

### Operating a private network

Maintaining your own private network is more involved as a lot of configurations taken for
granted in the official networks need to be manually set up.

#### Defining the private genesis state

First, you'll need to create the genesis state of your networks, which all nodes need to be
aware of and agree upon. This consists of a small JSON file (e.g. call it `genesis.json`):

```json
{
  "config": {
    "chainId": <arbitrary positive integer>,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "londonBlock": 0
  },
  "alloc": {},
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x20000",
  "extraData": "",
  "gasLimit": "0x2fefd8",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```

The above fields should be fine for most purposes, although we'd recommend changing
the `nonce` to some random value so you prevent unknown remote nodes from being able
to connect to you. If you'd like to pre-fund some accounts for easier testing, create
the accounts and populate the `alloc` field with their addresses.

```json
"alloc": {
  "0x0000000000000000000000000000000000000001": {
    "balance": "111111111"
  },
  "0x0000000000000000000000000000000000000002": {
    "balance": "222222222"
  }
}
```

With the genesis state defined in the above JSON file, you'll need to initialize **every**
`geth` node with it prior to starting it up to ensure all blockchain parameters are correctly
set:

```shell
$ geth init path/to/genesis.json
```

#### Creating the rendezvous point

With all nodes that you want to run initialized to the desired genesis state, you'll need to
start a bootstrap node that others can use to find each other in your network and/or over
the internet. The clean way is to configure and run a dedicated bootnode:

```shell
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key
```

With the bootnode online, it will display an [`enode` URL](https://ethereum.org/en/developers/docs/networking-layer/network-addresses/#enode)
that other nodes can use to connect to it and exchange peer information. Make sure to
replace the displayed IP address information (most probably `[::]`) with your externally
accessible IP to get the actual `enode` URL.

*Note: You could also use a full-fledged `geth` node as a bootnode, but it's the less
recommended way.*

#### Starting up your member nodes

With the bootnode operational and externally reachable (you can try
`telnet <ip> <port>` to ensure it's indeed reachable), start every subsequent `geth`
node pointed to the bootnode for peer discovery via the `--bootnodes` flag. It will
probably also be desirable to keep the data directory of your private network separated, so
do also specify a custom `--datadir` flag.

```shell
$ geth --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*Note: Since your network will be completely cut off from the main and test networks, you'll
also need to configure a miner to process transactions and create new blocks for you.*

#### Running a private miner


In a private network setting a single CPU miner instance is more than enough for
practical purposes as it can produce a stable stream of blocks at the correct intervals
without needing heavy resources (consider running on a single thread, no need for multiple
ones either). To start a `geth` instance for mining, run it with all your usual flags, extended
by:

```shell
$ geth <usual-flags> --mine --miner.threads=1 --miner.etherbase=0x0000000000000000000000000000000000000000
```

Which will start mining blocks and transactions on a single CPU thread, crediting all
proceedings to the account specified by `--miner.etherbase`. You can further tune the mining
by changing the default gas limit blocks converge to (`--miner.targetgaslimit`) and the price
transactions are accepted at (`--miner.gasprice`).

## Contribution

Thank you for considering helping out with the source code! We welcome contributions
from anyone on the internet, and are grateful for even the smallest of fixes!

If you'd like to contribute to go-ethereum, please fork, fix, commit and send a pull request
for the maintainers to review and merge into the main code base. If you wish to submit
more complex changes though, please check up with the core devs first on [our Discord Server](https://discord.gg/invite/nthXNEv)
to ensure those changes are in line with the general philosophy of the project and/or get
some early feedback which can make both your efforts much lighter as well as our review
and merge procedures quick and simple.

Please make sure your contributions adhere to our coding guidelines:

 * Code must adhere to the official Go [formatting](https://golang.org/doc/effective_go.html#formatting)
   guidelines (i.e. uses [gofmt](https://golang.org/cmd/gofmt/)).
 * Code must be documented adhering to the official Go [commentary](https://golang.org/doc/effective_go.html#commentary)
   guidelines.
 * Pull requests need to be based on and opened against the `master` branch.
 * Commit messages should be prefixed with the package(s) they modify.
   * E.g. "eth, rpc: make trace configs optional"

Please see the [Developers' Guide](https://geth.ethereum.org/docs/developers/geth-developer/dev-guide)
for more details on configuring your environment, managing project dependencies, and
testing procedures.

### Contributing to geth.ethereum.org

For contributions to the [go-ethereum website](https://geth.ethereum.org), please checkout and raise pull requests against the `website` branch.
For more detailed instructions please see the `website` branch [README](https://github.com/ethereum/go-ethereum/tree/website#readme) or the 
[contributing](https://geth.ethereum.org/docs/developers/geth-developer/contributing) page of the website.

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html),
also included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) are licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also
included in our repository in the `COPYING` file.
[grpc]
# Enable grpc. The Cosmos gRPC service is used by various daemon processes,
# and must be enabled in order for the protocol to operate:
enable = true
# Non-standard gRPC ports are not supported at this time. Please run on port 9090, which is the default
# port specified in the config file.
# Note: grpc can be also be configured via start flags. Be careful not to change the default settings
# with either of the following flags: `--grpc.enable`, `--grpc.address`.
address = "0.0.0.0:9090""><img src="https://dydx.exchange/icon.svg?" width="256" /></p>

<h1 align="center">dYdX Chain</h1>

<div align="center">
  <a href='https://github.com/dydxprotocol/v4-chain/blob/main/LICENSE'>
    <img src='https://img.shields.io/badge/License-AGPL_v3-blue.svg' alt='License' />
  </a>
</div>

The dYdX v4 software (the â€dYdX Chainâ€) is a sovereign blockchain software built using Cosmos SDK and CometBFT. It powers a robust decentralized perpetual futures exchange with a high-performance orderbook and matching engine for a feature-rich, self-custodial perpetual trading experience on any market.

This repository contains the source code for the Cosmos SDK application responsible for running the chain and the associated indexer services. The indexer services are a read-only layer that indexes on-chain and off-chain data from a node and serves it to users in a more performant and user-friendly way.

# Getting Started

[Architecture Overview](https://dydx.exchange/blog/v4-technical-architecture-overview)

[Indexer Deep Dive](https://dydx.exchange/blog/v4-deep-dive-indexer)

[Front End Deep Dive](https://dydx.exchange/blog/v4-deep-dive-front-end)

# Resources and Documentation

[Official Documentation](https://docs.dydx.exchange/)

[dYdX Academy](https://dydx.exchange/crypto-learning#)

[dYdX Blog](https://dydx.exchange/blog#)

# Clients

[v4-clients](https://github.com/dydxprotocol/v4-clients)

# Third-party Clients

[C++ Client](https://github.com/asnefedovv/dydx-v4-client-cpp)

[Python Client](https://github.com/kaloureyes3/v4-clients/tree/main/v4-client-py)

By clicking the above links to third-party clients, you will leave the dYdX Trading Inc. (â€œdYdXâ€) GitHub repository and join repositories made available by third parties, which are independent from and unaffiliated with dYdX. dYdX is not responsible for any action taken or content on third-party repositories.

# Directory Structure

`audits` â€” Audit reports live here.

`docs` â€” Home for documentation pertaining to the entire repo.

`indexer` â€” Contains source code for indexer services. See its [README](https://github.com/dydxprotocol/v4-chain/blob/main/indexer/README.md) for developer documentation.

`proto` â€” Contains all protocol buffers used by protocol and indexer.

`protocol` â€” Contains source code for the Cosmos SDK app that runs the protocol. See its [README](https://github.com/dydxprotocol/v4-chain/blob/main/protocol/README.md) for developer documentation.

`v4-proto-js` â€” Scripts for publishing proto package to [npm](https://www.npmjs.com/package/@dydxprotocol/v4-proto).

`v4-proto-py` â€” Scripts for publishing proto package to [PyPI](https://pypi.org/project/v4-proto/).

# Contributing

If you encounter a bug, please file an [issue](https://github.com/dydxprotocol/v4-chain/issues) to let us know. Alternatively, please feel free to contribute a bug fix directly. If you are planning to contribute changes that involve significant design choices, please open an issue for discussion instead.

# License and Terms

The dYdX Chain is licensed under AGPLv3 and subject to the [v4 Terms of Use](https://dydx.exchange/v4-terms). The full license can be found [here](https://github.com/dydxprotocol/v4-chain/blob/main/LICENSE). dYdX does not deploy or run v4 software for public use, or operate or control any dYdX Chain infrastructure. dYdX products and services are not available to persons or entities who reside in, are located in, are incorporated in, or have registered offices in the United States or Canada, or Restricted Persons (as defined in the dYdX [Terms of Use](https://dydx.exchange/terms)).
# This is a TOML document.

title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00 # First class dates

[database]
server = "192.168.1.1"
ports = [ 8000, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # Indentation (tabs and/or spaces) is allowed but not required
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]# This is a TOML config file.
# For more information, see https://github.com/toml-lang/toml

###############################################################################
###                           Base Configuration                            ###
###############################################################################

# The minimum gas prices a validator is willing to accept for processing a
# transaction. A transaction's fees must meet the minimum of any denomination
# specified in this config (e.g. 0.25token1,0.0001token2).
minimum-gas-prices = "0stake"

# default: the last 362880 states are kept, pruning at 10 block intervals
# nothing: all historic states will be saved, nothing will be deleted (i.e. archiving node)
# everything: 2 latest states will be kept; pruning at 10 block intervals.
# custom: allow pruning options to be manually specified through 'pruning-keep-recent', and 'pruning-interval'
pruning = "default"

# These are applied if and only if the pruning strategy is custom.
pruning-keep-recent = "0"
pruning-interval = "0"

# HaltHeight contains a non-zero block height at which a node will gracefully
# halt and shutdown that can be used to assist upgrades and testing.
#
# Note: Commitment of state will be attempted on the corresponding block.
halt-height = 0

# HaltTime contains a non-zero minimum block time (in Unix seconds) at which
# a node will gracefully halt and shutdown that can be used to assist upgrades
# and testing.
#
# Note: Commitment of state will be attempted on the corresponding block.
halt-time = 0

# MinRetainBlocks defines the minimum block height offset from the current
# block being committed, such that all blocks past this offset are pruned
# from Tendermint. It is used as part of the process of determining the
# ResponseCommit.RetainHeight value during ABCI Commit. A value of 0 indicates
# that no blocks should be pruned.
#
# This configuration value is only responsible for pruning Tendermint blocks.
# It has no bearing on application state pruning which is determined by the
# "pruning-*" configurations.
#
# Note: Tendermint block pruning is dependent on this parameter in conjunction
# with the unbonding (safety threshold) period, state pruning and state sync
# snapshot parameters to determine the correct minimum value of
# ResponseCommit.RetainHeight.
min-retain-blocks = 0

# InterBlockCache enables inter-block caching.
inter-block-cache = true

# IndexEvents defines the set of events in the form {eventType}.{attributeKey},
# which informs Tendermint what to index. If empty, all events will be indexed.
#
# Example:
# ["message.sender", "message.recipient"]
index-events = []

# IavlCacheSize set the size of the iavl tree cache (in number of nodes).
iavl-cache-size = 781250

# IAVLDisableFastNode enables or disables the fast node feature of IAVL. 
# Default is false.
iavl-disable-fastnode = false

# IAVLLazyLoading enable/disable the lazy loading of iavl store.
# Default is false.
iavl-lazy-loading = false

# AppDBBackend defines the database backend type to use for the application and snapshots DBs.
# An empty string indicates that a fallback will be used.
# The fallback is the db_backend value set in Tendermint's config.toml.
app-db-backend = ""

###############################################################################
###                         Telemetry Configuration                         ###
###############################################################################

[telemetry]

# Prefixed with keys to separate services.
service-name = ""

# Enabled enables the application telemetry functionality. When enabled,
# an in-memory sink is also enabled by default. Operators may also enabled
# other sinks such as Prometheus.
enabled = false

# Enable prefixing gauge values with hostname.
enable-hostname = false

# Enable adding hostname to labels.
enable-hostname-label = false

# Enable adding service to labels.
enable-service-label = false

# PrometheusRetentionTime, when positive, enables a Prometheus metrics sink.
prometheus-retention-time = 0

# GlobalLabels defines a global set of name/value label tuples applied to all
# metrics emitted using the wrapper functions defined in telemetry package.
#
# Example:
# [["chain_id", "cosmoshub-1"]]
global-labels = []

###############################################################################
###                           API Configuration                             ###
###############################################################################

[api]

# Enable defines if the API server should be enabled.
enable = false

# Swagger defines if swagger documentation should automatically be registered.
swagger = false

# Address defines the API server to listen on.
address = "tcp://localhost:1317"

# MaxOpenConnections defines the number of maximum open connections.
max-open-connections = 1000

# RPCReadTimeout defines the Tendermint RPC read timeout (in seconds).
rpc-read-timeout = 10

# RPCWriteTimeout defines the Tendermint RPC write timeout (in seconds).
rpc-write-timeout = 0

# RPCMaxBodyBytes defines the Tendermint maximum request body (in bytes).
rpc-max-body-bytes = 1000000

# EnableUnsafeCORS defines if CORS should be enabled (unsafe - use it at your own risk).
enabled-unsafe-cors = false

###############################################################################
###                           Rosetta Configuration                         ###
###############################################################################

[rosetta]

# Enable defines if the Rosetta API server should be enabled.
enable = false

# Address defines the Rosetta API server to listen on.
address = ":8080"

# Network defines the name of the blockchain that will be returned by Rosetta.
blockchain = "app"

# Network defines the name of the network that will be returned by Rosetta.
network = "network"

# Retries defines the number of retries when connecting to the node before failing.
retries = 3

# Offline defines if Rosetta server should run in offline mode.
offline = false

# EnableDefaultSuggestedFee defines if the server should suggest fee by default.
# If 'construction/medata' is called without gas limit and gas price,
# suggested fee based on gas-to-suggest and denom-to-suggest will be given.
enable-fee-suggestion = false

# GasToSuggest defines gas limit when calculating the fee
gas-to-suggest = 200000

# DenomToSuggest defines the default denom for fee suggestion.
# Price must be in minimum-gas-prices.
denom-to-suggest = "uatom"

###############################################################################
###                           gRPC Configuration                            ###
###############################################################################

[grpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the gRPC server address to bind to.
address = "localhost:9090"

# MaxRecvMsgSize defines the max message size in bytes the server can receive.
# The default value is 10MB.
max-recv-msg-size = "10485760"

# MaxSendMsgSize defines the max message size in bytes the server can send.
# The default value is math.MaxInt32.
max-send-msg-size = "2147483647"

###############################################################################
###                        gRPC Web Configuration                           ###
###############################################################################

[grpc-web]

# GRPCWebEnable defines if the gRPC-web should be enabled.
# NOTE: gRPC must also be enabled, otherwise, this configuration is a no-op.
enable = true

# Address defines the gRPC-web server address to bind to.
address = "localhost:9091"

# EnableUnsafeCORS defines if CORS should be enabled (unsafe - use it at your own risk).
enable-unsafe-cors = false

###############################################################################
###                        State Sync Configuration                         ###
###############################################################################

# State sync snapshots allow other nodes to rapidly join the network without replaying historical
# blocks, instead downloading and applying a snapshot of the application state at a given height.
[state-sync]

# snapshot-interval specifies the block interval at which local state sync snapshots are
# taken (0 to disable).
snapshot-interval = 0

# snapshot-keep-recent specifies the number of recent snapshots to keep and serve (0 to keep all).
snapshot-keep-recent = 2

###############################################################################
###                         Store / State Streaming                         ###
###############################################################################

[store]
streamers = []

[streamers]
[streamers.file]
keys = ["*"]
write_dir = ""
prefix = ""

# output-metadata specifies if output the metadata file which includes the abci request/responses 
# during processing the block.
output-metadata = "true"

# stop-node-on-error specifies if propagate the file streamer errors to consensus state machine.
stop-node-on-error = "true"

# fsync specifies if call fsync after writing the files.
fsync = "false"

###############################################################################
###                         Mempool                                         ###
###############################################################################

[mempool]
# Setting max-txs to 0 will allow for a unbounded amount of transactions in the mempool.
# Setting max_txs to negative 1 (-1) will disable transactions from being inserted into the mempool.
# Setting max_txs to a positive number (> 0) will limit the number of transactions in the mempool, by the specified amount.
#
# Note, this configuration only applies to SDK built-in app-side mempool
# implementations.
max-txs = 5000### Gas Prices ###
minimum-gas-prices = "0.025ibc/8E27BA2D5493AF5636760E354E46004562C46AB7EC0CC4C1CA14E9E20E2545B5,12500000000$NATIVE_TOKEN_DENOM"

### Pruning ###
pruning = "custom"
# Small numbers >= "2" for validator nodes.
# Larger numbers could be used for full-nodes if they are used for historical queries.
pruning-keep-recent = "7"
# Any prime number between "13" and "97", inclusive.
pruning-interval = "17"dydxprotocold start --p2p.seeds="..." --bridge-daemon-eth-rpc-endpoint="<eth rpc endpoint>" --non-validating-full-node=true# For the deployment by DYDX token holders
# CHAIN_ID=dydx-mainnet-1

# For testnet
CHAIN_ID=dydx-testnet-4{
  "$schema": "../chain.schema.json",
  "chain_name": "dydx",
  "status": "live",
  "website": "https://dydx.trade/",
  "network_type": "mainnet",
  "pretty_name": "dYdX Protocol",
  "chain_id": "dydx-mainnet-1",
  "bech32_prefix": "dydx",
  "daemon_name": "dydxprotocold",
  "node_home": "$HOME/.dydxprotocol",
  "key_algos": [
    "secp256k1"
  ],
  "slip44": 118,
  "fees": {
    "fee_tokens": [
      {
        "denom": "adydx",
        "fixed_min_gas_price": 12500000000,
        "low_gas_price": 12500000000,
        "average_gas_price": 12500000000,
        "high_gas_price": 20000000000
      },
      {
        "denom": "ibc/8E27BA2D5493AF5636760E354E46004562C46AB7EC0CC4C1CA14E9E20E2545B5",
        "fixed_min_gas_price": 0.025,
        "low_gas_price": 0.025,
        "average_gas_price": 0.025,
        "high_gas_price": 0.03
      }
    ]
  },
  "staking": {
    "staking_tokens": [
      {
        "denom": "adydx"
      }
    ]
  },
  "codebase": {
    "git_repo": "https://github.com/dydxprotocol/v4-chain/",
    "recommended_version": "protocol/v3.0.0",
    "compatible_versions": [
      "protocol/v3.0.0"
    ],
    "binaries": {
      "linux/amd64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv3.0.0/dydxprotocold-v3.0.0-linux-amd64.tar.gz",
      "linux/arm64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv3.0.0/dydxprotocold-v3.0.0-linux-arm64.tar.gz"
    },
    "cosmos_sdk_version": "dydxprotocol/cosmos-sdk v0.47.5-0.20240111163003-128eb0a555af",
    "ibc_go_version": "v7.3.0",
    "consensus": {
      "type": "cometbft",
      "version": "dydxprotocol/cometbft v0.37.3-0.20230908230338-65f7a2f25c18"
    },
    "genesis": {
      "genesis_url": "https://raw.githubusercontent.com/dydxopsdao/networks/main/dydx-mainnet-1/genesis.json"
    },
    "versions": [
      {
        "name": "v2",
        "recommended_version": "protocol/v2.0.0",
        "compatible_versions": [
          "protocol/v2.0.0"
        ],
        "binaries": {
          "linux/amd64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv2.0.0/dydxprotocold-v2.0.0-linux-amd64.tar.gz",
          "linux/arm64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv2.0.0/dydxprotocold-v2.0.0-linux-arm64.tar.gz"
        },
        "cosmos_sdk_version": "dydxprotocol/cosmos-sdk v0.47.5-0.20231011192538-b95c66dedbd5",
        "ibc_go_version": "v7.3.0",
        "consensus": {
          "type": "cometbft",
          "version": "dydxprotocol/cometbft v0.37.3-0.20230908230338-65f7a2f25c18"
        },
        "next_version_name": "v3.0.0"
      },
      {
        "name": "v3.0.0",
        "proposal": 7,
        "height": 7147832,
        "recommended_version": "protocol/v3.0.0",
        "compatible_versions": [
          "protocol/v3.0.0"
        ],
        "binaries": {
          "linux/amd64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv3.0.0/dydxprotocold-v3.0.0-linux-amd64.tar.gz",
          "linux/arm64": "https://github.com/dydxprotocol/v4-chain/releases/download/protocol%2Fv3.0.0/dydxprotocold-v3.0.0-linux-arm64.tar.gz"
        },
        "cosmos_sdk_version": "dydxprotocol/cosmos-sdk v0.47.5-0.20240111163003-128eb0a555af",
        "ibc_go_version": "v7.3.0",
        "consensus": {
          "type": "cometbft",
          "version": "dydxprotocol/cometbft v0.37.3-0.20230908230338-65f7a2f25c18"
        },
        "next_version_name": ""
      }
    ]
  },
  "logo_URIs": {
    "png": "https://raw.githubusercontent.com/cosmos/chain-registry/master/dydx/images/dydx.png",
    "svg": "https://raw.githubusercontent.com/cosmos/chain-registry/master/dydx/images/dydx.svg"
  },
  "description": "Our goal is to build open source code that can power a first class product and trading experience.",
  "peers": {
    "seeds": [
      {
        "id": "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0",
        "address": "seeds.polkachu.com:23856",
        "provider": "Polkachu"
      },
      {
        "id": "65b740ee326c9260c30af1f044e9cda63c73f7c1",
        "address": "seeds.kingnodes.net:23856",
        "provider": "KingNodes"
      },
      {
        "id": "f04a77b92d0d86725cdb2d6b7a7eb0eda8c27089",
        "address": "dydx-mainnet-seed.bwarelabs.com:36656",
        "provider": "Bware Labs"
      },
      {
        "id": "20e1000e88125698264454a884812746c2eb4807",
        "address": "seeds.lavenderfive.com:23856",
        "provider": "Lavender.Five Nodes ðŸ"
      },
      {
        "id": "c2c2fcb5e6e4755e06b83b499aff93e97282f8e8",
        "address": "tenderseed.ccvalidators.com:26401",
        "provider": "CryptoCrew"
      },
      {
        "id": "4f20c3e303c9515051b6276aeb89c0b88ee79f8f",
        "address": "seed.dydx.cros-nest.com:26656",
        "provider": "Crosnest"
      },
      {
        "id": "a9cae4047d5c34772442322b10ef5600d8e54900",
        "address": "dydx-mainnet-seednode.allthatnode.com:26656",
        "provider": "DSRV"
      },
      {
        "id": "802607c6db8148b0c68c8a9ec1a86fd3ba606af6",
        "address": "64.227.38.88:26656",
        "provider": "Luganodes"
      },
      {
        "id": "400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc",
        "address": "dydx.rpc.kjnodes.com:17059",
        "provider": "kjnodes"
      },
      {
        "id": "8542cd7e6bf9d260fef543bc49e59be5a3fa9074",
        "address": "seed.publicnode.com:26656",
        "provider": "Allnodes âš¡ï¸ Nodes & Staking"
      },
      {
        "id": "4c30c8a95e26b07b249813b677caab28bf0c54eb",
        "address": "rpc.dydx.nodestake.top:666",
        "provider": "NodeStake"
      },
      {
        "id": "ebc272824924ea1a27ea3183dd0b9ba713494f83",
        "address": "dydx-mainnet-seed.autostake.com:27366",
        "provider": "AutoStake | StakeDrops"
      },
      {
        "id": "09ba537d6563018b97c502979c3478df4decf426",
        "address": "dydxprotocol-seed.genznodes.dev:22656",
        "provider": "genznodes"
      }
    ],
    "persistent_peers": [
      {
        "id": "ebc272824924ea1a27ea3183dd0b9ba713494f83",
        "address": "dydx-mainnet-peer.autostake.com:27366",
        "provider": "AutoStake | StakeDrops"
      }
    ]
  },
  "apis": {
    "rpc": [
      {
        "address": "https://dydx-dao-rpc.polkachu.com",
        "provider": "Polkachu"
      },
      {
        "address": "https://dydx-mainnet-full-rpc.public.blastapi.io",
        "provider": "Bware Labs"
      },
      {
        "address": "https://dydx-rpc.kingnodes.com:443",
        "provider": "Kingnodes"
      },
      {
        "address": "https://dydx-rpc.lavenderfive.com:443",
        "provider": "Lavender.Five Nodes ðŸ"
      },
      {
        "address": "https://dydx-mainnet-rpc.autostake.com:443",
        "provider": "AutoStake | StakeDrops"
      },
      {
        "address": "https://rpc-dydx.ecostake.com:443",
        "provider": "ecostake"
      },
      {
        "address": "https://rpc.dydx.nodestake.top:443",
        "provider": "NodeStake"
      },
      {
        "address": "https://rpc-dydx.cosmos-spaces.cloud",
        "provider": "Cosmos Spaces"
      },
      {
        "address": "https://dydx-rpc.publicnode.com:443",
        "provider": "Allnodes âš¡ï¸ Nodes & Staking"
      },
      {
        "address": "https://rpc-dydx.cros-nest.com:443",
        "provider": "Crosnest"
      },
      {
        "address": "https://dydx-rpc.enigma-validator.com",
        "provider": "Enigma"
      },
      {
        "address": "https://community.nuxian-node.ch:6797/dydx/trpc",
        "provider": "PRO Delegators"
      }
    ],
    "rest": [
      {
        "address": "https://community.nuxian-node.ch:6797/dydx/crpc",
        "provider": "PRO Delegators"
      },
      {
        "address": "https://dydx-dao-api.polkachu.com",
        "provider": "Polkachu"
      },
      {
        "address": "https://dydx-mainnet-full-lcd.public.blastapi.io",
        "provider": "Bware Labs"
      },
      {
        "address": "https://dydx-rest.kingnodes.com:443",
        "provider": "Kingnodes"
      },
      {
        "address": "https://dydx-api.lavenderfive.com:443",
        "provider": "Lavender.Five Nodes ðŸ"
      },
      {
        "address": "https://dydx-mainnet-lcd.autostake.com:443",
        "provider": "AutoStake | StakeDrops"
      },
      {
        "address": "https://rest-dydx.ecostake.com:443",
        "provider": "ecostake"
      },
      {
        "address": "https://api-dydx.cosmos-spaces.cloud",
        "provider": "Cosmos Spaces"
      },
      {
        "address": "https://api.dydx.nodestake.top:443",
        "provider": "NodeStake"
      },
      {
        "address": "https://dydx-rest.publicnode.com",
        "provider": "Allnodes âš¡ï¸ Nodes & Staking"
      },
      {
        "address": "https://rest-dydx.cros-nest.com:443",
        "provider": "Crosnest"
      },
      {
        "address": "https://dydx-lcd.enigma-validator.com",
        "provider": "Enigma"
      }
    ],
    "grpc": [
      {
        "address": "dydx-dao-grpc-1.polkachu.com:23890",
        "provider": "Polkachu (1)"
      },
      {
        "address": "dydx-dao-grpc-2.polkachu.com:23890",
        "provider": "Polkachu (2)"
      },
      {
        "address": "dydx-dao-grpc-3.polkachu.com:23890",
        "provider": "Polkachu (3)"
      },
      {
        "address": "dydx-dao-grpc-4.polkachu.com:23890",
        "provider": "Polkachu (4)"
      },
      {
        "address": "dydx-dao-grpc-5.polkachu.com:23890",
        "provider": "Polkachu (5)"
      },
      {
        "address": "dydx-mainnet-full-grpc.public.blastapi.io:443",
        "provider": "Bware Labs"
      },
      {
        "address": "https://dydx-grpc.kingnodes.com:443",
        "provider": "Kingnodes"
      },
      {
        "address": "https://dydx-grpc.lavenderfive.com",
        "provider": "Lavender.Five Nodes ðŸ"
      },
      {
        "address": "dydx-mainnet-grpc.autostake.com:443",
        "provider": "AutoStake | StakeDrops"
      },
      {
        "address": "https://grpc.dydx.nodestake.top",
        "provider": "NodeStake"
      },
      {
        "address": "dydx.grpc.kjnodes.com:443",
        "provider": "kjnodes"
      },
      {
        "address": "grpc-dydx.cosmos-spaces.cloud:4990",
        "provider": "Cosmos Spaces"
      },
      {
        "address": "dydx-grpc.publicnode.com:443",
        "provider": "Allnodes âš¡ï¸ Nodes & Staking"
      }
    ]
  },
  "explorers": [
    {
      "kind": "mintscan",
      "url": "https://www.mintscan.io/dydx",
      "tx_page": "https://www.mintscan.io/dydx/txs/${txHash}",
      "account_page": "https://www.mintscan.io/dydx/account/${accountAddress}"
    },
    {
      "kind": "ezstaking",
      "url": "https://ezstaking.app/dydx",
      "tx_page": "https://ezstaking.app/dydx/txs/${txHash}",
      "account_page": "https://ezstaking.app/dydx/account/${accountAddress}"
    },
    {
      "kind": "NodeStake",
      "url": "https://explorer.nodestake.top/dydx/",
      "tx_page": "https://explorer.nodestake.top/dydx/txs/${txHash}",
      "account_page": "https://explorer.nodestake.top/dydx/account/${accountAddress}"
    },
    {
      "kind": "TC Network",
      "url": "https://explorer.tcnetwork.io/dydx",
      "tx_page": "https://explorer.tcnetwork.io/dydx/transaction/${txHash}",
      "account_page": "https://explorer.tcnetwork.io/dydx/account/${accountAddress}"
    }
  ],
  "images": [
    {
      "png": "https://raw.githubusercontent.com/cosmos/chain-registry/master/dydx/images/dydx.png",
      "svg": "https://raw.githubusercontent.com/cosmos/chain-registry/master/dydx/images/dydx.svg"
    }
  ]
}