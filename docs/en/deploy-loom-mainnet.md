## First Steps with the Developer Dashboard

This page shows you how to deploy your first smart contract to Loom mainnet. Note that you only need to do this once you go live, and it costs $99 a year to deploy to mainnet.

First, head over to our [Developer Dashboard](https://dashboard.dappchains.com/login). We offer 3 different ways in which you can connect and sign transactions:

![Select Wallet](/developers/img/deploy-to-loom-select-wallet.png)

Depending on your preference click "Ledger", "Metamask" or "Trezor via Metamask".

Next, if you already have a Relentless Marketplace account, you will be asked if you want to link it:

![Link Marketplace Account](/developers/img/deploy-to-loom-link-marketplace-account.png)

If so, click on the "Link my account" button and follow the instructions.

If you want to use a different account, just click on the "Nope, I'm not from Relentless Marketplace" button.

Say you choose to create a new account. Once you've selected this option, you'll see a Metamask popup like this:

<img src="/developers/img/deploy-to-loom-map-accounts.png" alt="Map Accounts" width="400"/>

Click "Sign" and, under the hood, the dashboard will map your accounts. Next, you will be redirected to your dashboard. It'll look similar to this:

![Map Accounts](/developers/img/deploy-to-loom-dashboard.png)


## Deposit Loom to Plasmachain

From the left pane, click on the "Deploy to Plasmachain". You will be redirected to a page similar to the one below:

![Deposit Loom to Plasma Chain](/developers/img/deploy-to-loom-deposit-loom.png)

Notice that, in order to deploy to Plasmachain, you need to stake some LOOM. Click on the "Deposit more Loom to Plasmachain" and follow the instructions.

## Adding the Public Keys

From here you have two options. You either create a new account, or you use an existing address:

Let's explore both a bit further.

#### Option 1 - Creating a New Account

Say you clicked "Generate New Key". Next, a new window will pop showing the details of the new account:

![New Account Details](/developers/img/deploy-to-loom-create-new-account.png)

We can't emphasize this more: **store the mnemonic somewhere safe**.

Next, copy the public address and paste it into the "Your Loom Public Address" textbox:

![Public Address](/developers/img/deploy-to-loom-paste-public-address.png)

Lastly, click "Add Key" and you're set.

#### Option 2 - Use an Existing Address

Now, as an example, let's suppose you already have a Loom address. If so, all you have to do is to paste it into the "Your Loom Public Address".

Next, click "Add Key" and you're ready to deploy to Loom mainnet.

## Deploying to Loom mainnet

If in the previous step, you chose to use an existing address, chances are that you already know how to use your private key to deploy to Loom. Thus, we won't describe that scenario.

However, if you chose to generate a new address, here's what you have to do:

 - Fire up your favorite text editor and open `truffle-config.js`

 - Start by importing a few things:

```js
const LoomTruffleProvider = require('loom-truffle-provider')
const { sha256 } = require ('js-sha256')
const { CryptoUtils } = require ('loom-js')
const { mnemonicToSeedSync } = require ('bip39')
```

 - Next, we'll have to compute the private key from mnemonic as follows:

 ```js
const seed = mnemonicToSeedSync(mnemonic)
const privateKeyUint8ArrayFromSeed = CryptoUtils.generatePrivateKeyFromSeed(new Uint8Array(sha256.array(seed)))
const privateKeyB64 = CryptoUtils.Uint8ArrayToB64(privateKeyUint8ArrayFromSeed)
 ```

- Lastly, we can instantiate `loomProvider` like this:

```js
new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKeyB64)
```

Wrapping it up, your `truffle-config.js` file should look something like this:

```js

const LoomTruffleProvider = require('loom-truffle-provider')
const { sha256 } = require ('js-sha256')
const { CryptoUtils } = require ('loom-js')
const { mnemonicToSeedSync } = require ('bip39')

const mnemonic = readFileSync(path.join(__dirname, 'loom_mnemonic'), 'utf-8')

module.exports = {
  networks: {
    loom_mainnet: {
      provider: function () {
        const chainId = 'default'
        const writeUrl = 'http://plasma.dappchains.com/rpc'
        const readUrl = 'http://plasma.dappchains.com/query'
        const seed = mnemonicToSeedSync(mnemonic)
        const privateKeyUint8ArrayFromSeed = CryptoUtils.generatePrivateKeyFromSeed(new Uint8Array(sha256.array(seed)))
        const privateKeyB64 = CryptoUtils.Uint8ArrayToB64(privateKeyUint8ArrayFromSeed)
        return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKeyB64)
      },
      network_id: '*'
    }
  }
}
```

Well done!👏🏻👏🏻👏🏻

You are now ready to deploy your first smart contract to Loom mainnet!

In order to get a better feel for it, check out our [truffle-dappchain-example](https://github.com/loomnetwork/truffle-dappchain-example).