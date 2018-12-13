# VeChain Development Environment
This guide will cover the process of setting up your local environment for developing on the VeChainThor blockchain using the standard `web3` stack.

- [Setting up your local VeChain Thor node](#vechain-thor-local-node)
- [Setting up `web3-gear` to use `truffle` with VeChain](#web3-gear)
- [Setting up truffle to work with VeChain](#truffle)

### Overview of development stack
![VeChain Development Stack](https://user-images.githubusercontent.com/747165/49694457-54432c80-fb3f-11e8-81dc-1940c12f8891.png)

## Running the VeChainThor local node

First we'll need to set up a VeChainThor node running locally for development. This will be your own local testnet that you can easiliy spin up or down without having to worry about affecting the rest of the network. 

### Requirements

- Thor requires `Go` 1.10+ and `C` compiler to build. To install `Go`, follow this [link](https://golang.org/doc/install). 
- Install [dep](https://github.com/golang/dep) dependency manager for `Go`

### Getting the source

If you haven't already, setup your `go` filesystem:
```
mkdir -p go/src
cd go/src
```

Clone the thor repo:

```
git clone https://github.com/vechain/thor.git
cd thor
```
### Build thor

Install dependencies
```
dep ensure
```

To build the `thor` node application run
```
make
```

### Run node
Once built you will want to run the node in `solo` mode. This instructs the node to run locally as its own network instead of connecting to `testnet` or `mainnet`.
```
./bin/thor solo --api-cors "*"
```

The `--api-cors` flag enables secure websocket connections from web3 and the Comet extension.

The node should now be up and running and ready to connect. Unless specified with further options, all blockchain data will be stored in-memory and will be reset when you kill the process. This is useful for continual debugging.

### Further Info
Full guide: https://github.com/vechain/thor/blob/master/README.md

## web3-gear
web3-gear is a proxy server that converts Ethereum RPC calls to VeChainThor RPC calls. You will need to run web3-gear so that truffle can talk to your local VeChainThor node.

### Installation
web3-gear requires OpenSSL
```
brew install openssl
```
Install web3-gear with `pip`
```
pip3 install web3-gear
```
### Run
```
web3-gear
```
### Other useful options
| Flag | Description |
| --- | --- |
| --host | rpc service host, eg: `--host 127.0.0.1` |
| --port | rpc service port, eg: `--port 8545` |
| --keystore | custom keystore filepath, eg: `--keystore /path/to/keystore)` |
| --passcode | passcode of custom keystore, eg: `--passcode xxxxxxxx` |

### Further Info
Full Guide: https://github.com/vechain/web3-gear/blob/master/README.rst

## Truffle
Truffle is your development environment for building and deploying contract code. Truffle is an Ethereum project however using web3-gear, can work with VeChainThor.

### Initialize
If you have not yet initialized truffle in your project run
```
truffle init
```

### Configure
Configure your project to talk to `web3-gear` in your `truffle.js` file
```
module.exports = {
    networks: {
        development: {
            host: "localhost",
            port: 8545,
            network_id: "*" // Match any network id
        }
    }
};
```

### Build
To build your project run
```
truffle build
```

### Deploy
To deploy your project's contracts to your local VeChainThor node run
```
truffle migrate
```

#### Deploying to full network
- Deploying to `testnet` or `mainnet` can be accomplish by pointing your `web3-gear` at a `testnet` or `mainnet` node using the `--host` and `--port` flags
- When deploying to testnet or mainnet you'll need to load your own wallet into `web3-gear` using the `--keystore` and `--passcode` flags. Make sure to also use this address in your truffle migration scripts

### Debugging
To debug a transaction run
```
truffle debug <TxID>
```

Note that this only works in the latest `master` of `thor`. You must run your local node in `solo` mode and not use the `--persist` or `--on-demand` flags. This feature is a work in progress so you may experience bugs while using.

### Further Info
[Truffle Documentation](https://truffleframework.com/docs/truffle/overview)

## Setting up Comet
Comet enables `web3` within VeChainThor dApps. Comet can easily be configured to connect to your local node for development purposes.
### Connecting to local node
In `Settings > Network`, ensure that you are connected to `Localhost`. Your node will need to be running on the default port: `8669` for Comet to connect.

![screen shot 2018-12-13 at 11 49 10 am](https://user-images.githubusercontent.com/747165/49963539-1f333300-fecd-11e8-9284-ed108a36968a.png)

### Importing private keys
Upon launch of your local node, you are presented with the set of hardcoded local node wallet addresses:

![screen shot 2018-12-13 at 11 55 50 am](https://user-images.githubusercontent.com/747165/49964152-a208bd80-fece-11e8-9d4e-1ba848cdb863.png)

These can be imported into Comet in `Accounts > Add`:

![screen shot 2018-12-13 at 12 05 46 pm](https://user-images.githubusercontent.com/747165/49964473-70442680-fecf-11e8-8eac-47d1a1add967.png)
