# Authenticating Users

Comet provides ways to authenticate users using both Web3 and Connex. The overall flow for user authentication is as follows:

1. Generate an Auth message
2. Present the message to the user for signing
3. Retrieve address and signature from the user
4. Recover the address from the signature and verify


Note: This guide assumes you are working with an Express Server with Express Sessions middleware
## 1. Generate an Auth message
```js
function generateChallengeMessage () {
  return `dApp Authentication (${Math.random().toString(36).substring(2)})`
}
```

## 2. Present the message to the user for signing
```js
// server.js
const challengeCache = {}

// GET /auth
function issueChallenge (request, response) {
  const sessionID = request.sessionID
  const challenge = generateChallengeMessage()
  challengeCache[sessionID] = challenge

  return response.json({ challenge })
}
```
```js
// client.js
const challenge = await fetch('/auth').then(res => res.json()).then(({ challenge }) => challenge)

// Connex
const signingService = connex.vendor.sign('cert')
const signatureData = await signingService.request({
  purpose: 'identification',
  payload: {
    type: 'text',
    content: challenge
  }
})

// Web3
const addresses = await web3.eth.getAccounts()
const address = addresses[0]
const signature = await web3.eth.personal.sign(challenge, address)
const signatureData = { signature, address }
```

## 3. Retrieve address and signature from the user
```js
// client.js
const response = await fetch('/auth', {
  method: 'POST',
  body: JSON.stringify({ signatureData })
})
```

## 4. Recover the address from the signature and verify
```js
// server.js
import { toBuffer } from 'ethereumjs-util'
import { cry, Certificate } from 'thor-devkit'

// POST /auth
function verifyChallenge (request, response) {
  const sessionID = request.sessionID
  const { signatureData } = request.body

  const challenge = challengeCache[sessionID]
  const address = recoverAddress(challenge, signatureData)
  if (!address) {
    return response.json({ success: false })
  }

  request.session.userdata = { address, authenticated: true }
  return request.session.save(err => {
    if (err) { /* handle error */ }
    return response.json({ success: true })
  })
}

// Connex
function recoverAddress (challenge, signatureData) {
  const { signature, annex } = signatureData
  const cert = {
    purpose: 'identification',
    payload: {
      type: 'text',
      content: challenge
    },
    domain: annex.domain,
    timestamp: annex.timestamp,
    signer: annex.signer,
    signature: `0x${signature.toString('hex')}`,
  }

  try {
    Certificate.verify(cert)
    return annex.signer
  } catch (e) {
    return ''
  }
}

// Web3
// Note: These helper functions are necessary until the Personal Sign VIP has been approved. See https://github.com/vechain/VIPs/pull/3. Once the VIP is approved, `web3.eth.accounts.recover` will replace these functions
function recoverAddress (challenge, signatureData) {
  const { signature, address } = signatureData
  const signatureBuffer = toBuffer(signature)
  const challengeBuffer = toBuffer(challenge)

  const pubKey = cry.secp256k1.recover(generatePersonalSignMessage(challengeBuffer), signatureBuffer)
  const recoveredAddress = cry.publicKeyToAddress(pubKey).toString('hex')

  if (`0x${recoveredAddress.toLowerCase()}` !== address.toLowerCase()) {
    return ''
  }

  return address
}

const VET_SIGN_PREFIX = "\u0019VeChain Signed Message:\n"

function generatePersonalSignMessage (messageBuffer) {
  const prefix = `${VET_SIGN_PREFIX}${messageBuffer.length}`
  const prefixBuffer = toBuffer(prefix)
  return cry.blake2b256(Buffer.concat([prefixBuffer, messageBuffer]))
}
```