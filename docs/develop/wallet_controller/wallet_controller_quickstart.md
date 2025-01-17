![image](/img/develop/wallet_controller/vault_blueprint_1.png)


# Lamden Wallet Controller Starter Template

If it's easier you can clone and run this starter template to get a website up and running quickly with the LWC.

[LWC Starter Template](https://github.com/Lamden/arko-svelte-template)

# Using the Lamden Vault Controller (LWC)

> This package should work with all javascript implementations including node.js and vanilla javascript

## What is it?
The Lamden Vault Controller (LWC) is a helper package for interacting with the <u>[Lamden Vault API](/docs/develop/wallet_api/overview)</u> via a webpage.
You can interact with the <u>[Lamden Vault API](/docs/develop/wallet_api/overview)</u> without this package, but the intent here is to replace the process of 
managing listeners and events with a more modern callback/Promise workflow.

- Check out the <u>[GITHUB REPO](https://github.com/Lamden/lamden_wallet_controller)</u>
- Download the raw <u>[walletController.js](https://raw.githubusercontent.com/Lamden/lamden_wallet_controller/master/walletController.js)</u> file
- Install the <u>[NPM PACKAGE](https://www.npmjs.com/package/lamden_wallet_controller)</u>

## Install

### NPM

1. Install via NPM
```bash
npm install lamden_wallet_contoller
```
2. Import into your project
```javascript
import WalletController from 'lamden_wallet_controller';
```

### Vanilla JS
1. add <u>[walletController.js](https://raw.githubusercontent.com/Lamden/lamden_wallet_controller/master/walletController.js)</u> to your project.
2. import into project
```javascript
import WalletController from './walletController.js'; // path to the walletController file.
```

## Create Connection Request Object
Creating a connection request is the same as <u>[Lamden Vault API - Create a Connection](docs/develop/wallet_api/overview)</u>.

Below is the mandatory information for a connection request but <u>[more customization options](/docs/develop/wallet_api/customize_connection)</u>  are available.
```javascript
const connectionRequest = {
    appName: 'My Killer dApp', // Your Dapps's name
    version: '1.0.0', // any version to start, increment later versions to update connection info
    logo: 'images/logo.png', // or whatever the location of your logo
    contractName: 'con_killer_app', // Will never change
    networkType: 'testnet', // other option is 'mainnet'
    networkName: 'arko' // If this is not included it will attempt to connect to Legacy Lamden network
}
```

## Create an Instance of the LWC
```javascript
const lwc = new WalletController(connectionRequest)
```

## Respond to Events from the LWC
```javascript
// Create event handlers
const handleWalletInfo = (walletInfo) => console.log(walletInfo) 
const handleTxResults = (txInfo) => console.log(txInfo) 

//Connect to event emitters
lwc.events.on('newInfo', handleWalletInfo) // Wallet Info Events, including errors
lwc.events.on('txStatus', handleTxResults) // Transaction Results
```

- See information returned from `newInfo` <u>[here](/docs/develop/wallet_api/get_wallet_info#request-wallet-info)</u>
- See information returned from `txStatus` <u>[here](/docs/develop/wallet_api/send_transactions#listen-for-transaction-result)</u>

## Check if Wallet is Installed
Calling `walletIsInstalled` will return a promise with the resolver being a boolean value as to the installed status of the wallet.

**If the wallet is installed the LWC will automatically send your connection request** and the user will see the <u>[New Linked Account](/docs/wallet/accounts_linked_create#linked-account-creation)</u> popup process. 

Once the user completes the popup process you will get the wallet Info on the `newInfo` event emitter.

```javascript
lwc.walletIsInstalled()
    .then(installed => {
        if (installed) // Connection request will automatically be sent.
        else // inform user to install the wallet
    })
```
So to recap, the `walletIsInstalled` method will accomplish 3 things:

1. Reports back the installed status of the wallet.
2. If installed is true and a connection request was provided to the constructor then it will automatically create the `lamdenWalletConnect` event and provide your connection request to the users Lamden Vault
3. If the user has a <u>[Linked Account](/docs/wallet/accounts_linked_overview)</u> for your Dapp you will be sent the Wallet Information on the `newInfo` listener; or else the user will complete the New Linked Account process and you will be provided the Wallet Information at that point.

## Send Connection Request
> ** Not required if you have already called `walletIsInstalled`**

`sendConnection` can be used if you didn't supply the connectionRequest information to the wallet previously or if you want to submit another connection request.

If the user has a <u>[Linked Account](/docs/wallet/accounts_linked_overview)</u> for your Dapp you will be sent the Wallet Information on the `newInfo` listener; or else the user will complete the New Linked Account process and you will be provided the Wallet Information at that point.

```javascript
lwc.sendConnection(connectionRequest)  
```


## Send A Transaction
Create the transaction object as per the <u>[Lamden Vault API](/docs/develop/wallet_api/send_transactions#create-transaction-detail)</u> specification.

> `contractName` is automatically filled in by the LamdenWallet with the value approved by the user in your connection request.

```javascript
const txInfo = {
    networkType: 'mainnet', // other option is 'testnet'
    networkName: 'arko', // If not included tx will go to old legacy Lamden network
    methodName: 'do_something', 
    kwargs: {
        Str: 'awesome', //send a string
        Float: {'__fixed__': '1000.000000006'} //send a float
        Int: 1000 //send and int
    }, 
    stampLimit: 100
};

const handleResults = (txResults) => console.log(txResults)

lwc.sendTransaction(transaction, handleResults) // callback is optional
```

You always get the transaction results on the `txStatus` event emitter even if a callback is defined or not.