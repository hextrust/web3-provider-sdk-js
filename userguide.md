# Gryfyn Web3 API

Gryfyn provides a Web3 API which allows developers to connect to an account and use it in a fashion similar to how they can integrate other Web3 providers such as MetaMask. For existing dApps, this allows the integration of Gryfyn without the need for large modifications of their existing code bases. At this point in time, the integration may be done via our Web3 Javascript library, and there is no typing in place.

It complies the EIP-1193 standard `https://eips.ethereum.org/EIPS/eip-1193`, which allow access from web3 libraries like `ethers.js` or `web3.js`.

This document provides a walk-through of how one may connect to the Gryfyn Web3 API, and subsequently, provides code samples as to the relevant calls.

Please note that this version is accurate as of 21st September 2022, and may be subject to change from time-to-time. Gryfyn is not responsible for any losses which may be incurred during the usage of our API, which is provided without warranty.

## Step 1: Including the API using the Javascript library

### Using javascript

To start, you can first import the API's JS file, `https://loader.uat-testnet.metazens.xyz/api.min.js` in the header, shown in the following HTML script:

```html
<script src="https://<% hosting domain %>/api.min.js"></script>
```

or using our UAT testnet

```html
<script src="https://loader.uat-testnet.metazens.xyz/api.min.js"></script>
```

Following the above code snippet, the provider initiator `window.GryFyn` will be injected.

### using NPM package:

For React projects, `@gryfyn/gryfyn-web3-api-js` will be provided.

You can import by the following script

```javascript
import GryFyn from '@gryfyn/gryfyn-web3-api-js';
```

## Step 2: Initiating the provider with an API key and a Web3 object

Now that the library has been included, it is now possible to initiate the provider with an API key, as well as a new Web3 object.
The provider can be instantiated as follows:

To create the EIP 1193 Provider, use the following script.

We will use `getProvider(<YOUR_API_KEY>)` to initiate the Wallet object;

The Wallet object is EIP 1193 competable. You can use it like Metamask.

### For using `api.min.js`

```typescript
const Wallet = GryFyn.default.getProvider('<YOUR_API_KEY>');
```

### For using NPM package

```javascript
const Wallet = GryFyn.getProvider(apiKey);
```

It will then be possible to extend the `Web3` object using the following snippet:

```typescript
const web3 = new Web3(Wallet);
```

## Step 3: Calling the Gryfyn wallet API

Once the provider has been instantiated with the injected Web3, it is then possible to connect the wallet with the `eth_requestAccounts` call. This allows the user to connect to the Wallet.

**Note that this must be the first API call, and is a prerequisite before other calls may be performed**
The initial `eth_requestAccounts` call may be performed as follows:

```typescript
const account = await Wallet.request({ method: 'eth_requestAccounts' });

const Address = account[0];
```

## Different calls:

After the wallet is connected by calling the `eth_requestAccounts` call, it is now possible to perform various calls in a manner which you would call an API using the `web3.js` library. See the code block below for examples:

```typescript
const address = await Wallet.request({ method: 'eth_requestAccounts' })[0];

await web3.eth.getBalance(address);
// decimal values in Wei
await web3.eth.getAccounts();
// address
await web3.eth.getChainId();
// chain id in decimal

// ...
```

## Signing calls

There are three ways of signing that can be performed with the Gryfyn API. These are:

- `signMessage` - Allows the signing of an input message using the private keys stored by Gryfyn
- `signTypedData` - Allows the signing of typed data using the private keys stored by the Gryfyn, in accordance with EIP-712 standards
- `signTx` - Allows the signing of a blockchain transaction using the private keys stored by Gryfyn
  The code snippets for the above calls are shown in this section.

### `signMessage` code snippet

```typescript
const signature = await web3.eth.personal.sign(<string>message, address, undefined);
// final result after interacting with wallet UI
console.log('Signed message: ', signature); // signature of the input message

// We can use ecRecover to verify the signing.
const recoveredAddress = await web3.eth.personal.ecRecover(message, signature);
console.log('ecRecover', recoveredAddress);
// It shound return the same address
```

## Sending Transactions

There are multiple methods which may be used to send transactions:

- Using the Web3.js API, in line with what was shown in the above examples
- Using a MetaMask-like interface

### Using the Web3.js API to Send Transactions

A transaction can be sent in a method similar to the way it is done in the Web3.js library as follows:

```typescript
let handleSendTransaction = async (amount, toAddress) => {
    let web3 = new Web3(Wallet);

    // Get Address
    const [{ address }] = await Wallet.request({ method: "eth_requestAccounts" });


    // Get nonce
    const nonce = await web3.eth.getTransactionCount(address);

    // Send Tx
    web3.eth.sendTransaction({
        from: address,
        to: destAddress,
        value: web3.utils.toWei(amount, 'ether'),
        gasLimit: "0x5208",
        nonce: nonce,
        maxFeePerGas: "0x174e6cc900",
        maxPriorityFeePerGas: "0x174e6cc900",
	});
```

#### Using a MetaMask Interface

Transactions can also be sent using a fashion similar to how it is integrated with MetaMask. Using the API key mentioned above, this will be shown as follows:

```typescript
function sendETH() {
        // Get Address
        const [{ address }] = await Wallet.request({ method: "eth_requestAccounts" });

        // Get chainID
        // NOTE: This is done in a hex format rather than a decimal string
        const { chainId } = await Wallet.request({ method: 'eth_chainId' });
        // Get nonce
        const nonce = await Wallet.request({
          "method": "eth_getTransactionCount",
          "params": [address, "latest"],
        });
        const destAddress = "0x3f009302978CC82C6dAF06AAc49B5716be79cCb9";
        // Send Tx
        const params = [
        {
            from: address,
            to: destAddress,
            gas: '0x760c0',
            gasPrice: '0x9172a000',
            value: '0x9184e72a000',
            nonce: nonce,
        }
        Wallet.request({
            method: 'eth_sendTransaction',
            params,
        }
    }
```

## GryFyn Custom calls

In order to programmatically open or close the wallet UI, the following code snippet may be used:

```typescript
Wallet.openWallet();
Wallet.closeWallet();

Wallet.isGryFyn = true; // Always true
Wallet.isMetaMask = false; // Always False

// ...
```

### Get Account KYC Status

Level 1 - Email Address verified
Level 2 - KYC verified
Level 3 - MFA configured

```javascript
Wallet.getUserLevel(); // Get User Level.

// Open kyc Uer page
Wallet.request({ method: 'gryfyn_requestKYC' });
```
