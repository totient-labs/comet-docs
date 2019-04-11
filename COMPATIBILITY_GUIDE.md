# Comet Compatibility Guide

While Comet exposes the [standard (thorified) web3 API](https://github.com/vechain/thorify#web3-method-supported), there are few things to keep in mind. Below are requirements for Comet support as well as some best practices.

## Requirements

### Thor - Creating your bridge to the blockchain

Comet injects `thor` object into the javascript context.
Look for this before using your fallback strategy

```js
import { thorify } from 'thorify'
import { extend } from 'thorify/dist/extend'
const Web3 = require("web3"); // Recommend using require() instead of import here

window.addEventListener('load', function() {
  // Checking if Thor has been injected by the browser
  if (typeof thor !== 'undefined') {
    // Use thor provider
    web3js = new Web3(thor);
    // Extend web3 to connect to VeChain Blockchain
    extend(web3js)
  } else {
    // Fall back to default thorified construction
    web3js = thorify(new Web3(), "http://localhost:8669");
  }

  // Now you can start your dApp
  startApp()
})
```
See more at the [thorify docs on "usage"](https://github.com/vechain/thorify#usage)

### Request Account Access

Comet does not expose user account addresses by default; instead a dApp must first request access which the user must approve. We recommend only requesting access after the user has interacted with your dApp for a better user experience (see best practices below).
```javascript
async function enableThor () {
  try {
    const [cometAccount] = await thor.enable();
    return cometAccount;
  } catch (e) {
    console.log(`User rejected request ${e}`);
    // handle error
  }
}
```

### Comet Handles User Authorization

As Comet handles all authorization, your dApp likely won't need to call `sendRawTransaction` anymore.

Any time you make a call that requires a private key to sign something (`sendTransaction`, `personal.sign`), Comet will automatically prompt the user for permission, and then forward the signed request on to the blockchain (or return it to you, if it was a call to `personal.sign`).

`web3.eth.sendTransaction` will return a PromiseEvent which will `resolve` when the transaction is confirmed. It will also emit events like `transactionHash` and `receipt` when they happen. Read more at the [thorify API docs](https://vechain.github.io/thorify/#/?id=send-transaction-1)

### All Async

The user does not have the full blockchain on their machine, so data lookups can be a little slow.
For this reason, we are unable to support most synchronous methods.

Using synchronous calls is both a technical limitation and a user experience issue. They block the user's interface. So using them is a bad practice, anyway. Think of this API restriction as a gift to your users.

## Best Practices

### Requesting Account Information

Comet will only provide account information after user authorization to protect user privacy. When a dApp requests access, a confirmation screen will appear to the user similar to a transaction or sign flow. In order to avoid popups randomly appearing, your dApp should only call `thor.enable` upon user interaction with your dApp.

### Network check

When a user interacts with a dApp via Comet, they may be on the mainnet or testnet. As a best practice, your dApp should inspect the current network via the `getChainTag` call. Then, the dApp can use the correct deployed contract addresses for the network, or show which network is expected in a message.

For example:
```javascript
web3.eth.getChainTag().then(chainTagHex => {
  const chainTag = parseInt(chainTagHex, 16)
  switch (chainTag) {
    case 74:
      console.log('This is mainnet')
      break
    case 39:
      console.log('This is testnet')
      break
    case 199:
      console.log('This is localhost.')
      break
    default:
      console.log('This is an unknown network.')
  }
})
```

### Account management and transaction signing is managed externally to the dApp

Many dApps have a built-in identity management solution as a fallback.
When a Thor Browser environment has been detected, the user interface should reflect that the accounts are being managed externally.

### Account List Reflects User Preference

When a user selects an account in Comet, that account becomes available via the `web3.eth.getAccounts` async call. It is the only account returned by that call.
```javascript
web3.eth.getAccounts(([cometAccount]) => {
  console.log(`The user address is ${cometAccount}`)
})
```

### Listening for Selected Account Changes

Since these variables reflect user intention, but do not (currently) have events representing their values changing, we recommend using an interval to check for account changes.

For example, if your application only cares about the `web3.eth.getAccounts[0]` value, you might add some code like this somewhere in your application:
```javascript
const [cometAccount] = await web3.eth.getAccounts();
const accountInterval = setInterval(function() {
  const [currentAccount] = await web3.eth.getAccounts();
  if (currentAccount !== cometAccount) {
    cometAccount = currentAccount
    updateInterface();
  }
}, 300);
