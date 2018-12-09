# VeChain Development Environment
This guide will cover the process of setting up your local environment for developing on the VeChainThor blockchain using the standard `web3` stack.

- [Setting up your local VeChain Thor node](#vechain-thor-local-node)
- [Setting up `web3-gear` to use `truffle` with VeChain](#web3-gear)
- [Setting up truffle to work with VeChain](#truffle)

### Overview of development stack
![VeChain Development Stack](https://user-images.githubusercontent.com/747165/49694457-54432c80-fb3f-11e8-81dc-1940c12f8891.png)

## VeChain Thor Local Node

First we'll need to set up a node running locally for development. This will be your own local testnet.

### Requirements

- Thor requires `Go` 1.10+ and `C` compiler to build. To install `Go`, follow this [link](https://golang.org/doc/install). 
- Install [dep](https://github.com/golang/dep) dependency manager for `Go`

### Getting the source

If you haven't already, setup your go filesystem:
```
mkdir -p go/src
cd go/src
```

Clone the Thor repo:

```
git clone https://github.com/vechain/thor.git
cd thor
```
### Build Thor

Install dependencies
```
dep ensure
```

To build the main app `thor`, just run
```
make
```

### Run node
Once built you will want to run the node in solo mode
```
./bin/thor solo
```

The node should now be up and running and ready to connect. To reset the local blockchain, simply kill the service and restart.

### Further Info
Full guide: https://github.com/vechain/thor/blob/master/README.md

## web3-gear
web3-gear is a proxy server that surfaces an ethereum RPC interface and converts requests to vechain RPC request for your local node. You will need to run web3-gear so that truffle can talk to your local vechain node.
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
### Other options
| Flag | Description |
| --- | --- |
| --host | rpc service host, eg: ``--host 127.0.0.1`` |
| --port | rpc service port, eg: ``--port 8545`` |
| --keystore | keystore file path, eg: ``--keystore /Users/(username)/keystore)``, default=thor stand-alone(solo) built-in accounts |
| --passcode | passcode of keystore, eg: ``--passcode xxxxxxxx`` |

### Further Info
Full Guide: https://github.com/vechain/web3-gear/blob/master/README.rst

## Truffle
Truffle is your development environment for building and deploying contract code. Truffle is an Ethereum project however works with VeChain using web3-gear.

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
To deploy your project's contracts to your local node run
```
truffle migrate
```

- Deploying to testnet or mainnet can be accomplish by pointing your `web3-gear` at a testnet or mainnet node using the `--host` and `--port` flags
- When deploying to testnet or mainnet you'll need to load your own wallet into `web3-gear` using the `--keystore` and `--passcode` flags. Make sure to also use this address in your truffle migration scripts

### Debug
To debug a transaction run
```
truffle debug <TxID>
```

Note that this only works in the latest `master` of thor. You must run your local node in solo mode and not use the `--persist` or `--on-demand` flags.

### Further Info
[Truffle Documentation](https://truffleframework.com/docs/truffle/overview)