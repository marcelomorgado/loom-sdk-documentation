---
id: transfer-gateway
title: Transfer Gateway
sidebar_label: Transfer Gateway
---

## Overview

The Transfer Gateway allows tokens to be transferred between Loom DAppChains and Ethereum networks.

The Transfer Gateway consists of four main components:

- Gateway Solidity contract on Ethereum (Mainnet Gateway)
- Gateway Go contract on the Loom DAppChain (DAppChain Gateway)
- Address Mapper Go contract on the Loom DAppChain
- Gateway Oracle (can run in-process on a DAppChain node, or as a standalone process)

## Transferring a token from Ethereum to DAppChain

When a user wishes to transfer a token from their Ethereum account to their DAppChain account they must:

- Transfer it to the Mainnet Gateway,
- In turn, the Mainnet Gateway emits a deposit event.
- The deposit event is picked up by the Gateway Oracle which forwards it onto the DAppChain Gateway.
- The DAppChain Gateway then transfers the token to the DAppChain account of the user that deposited the token into the Mainnet Gateway.

![Diagram of ERC721 Transfer to DAppChain](/developers/img/transfer-gateway-erc721-to-dappchain.png)

## Transferring a token from DAppChain to Ethereum

To get that same token back into their Ethereum account the user must:

- Transfer the token back to the DAppChain Gateway, which creates a pending withdrawal.
- The pending withdrawal is picked up by the Gateway Oracle, which signs the withdrawal, and notifies the DAppChain Gateway.
- The DAppChain Gateway emits an event to let the user know they can withdraw their token from the Mainnet Gateway
to their Ethereum account by providing the signed withdrawal record.

![Diagram of ERC721 Transfer to Ethereum](/developers/img/transfer-gateway-erc721-to-ethereum.png)

If you're a hands-on learner you might want to jump straight into the [Transfer Gateway Example][] example project before reading any further.


## Setting up ERC721 contracts

To transfer an ERC721 token from Ethereum to the DAppChain you'll need two of your own ERC721 contracts, one on Ethereum (Mainnet ERC721), and the other on the DAppChain (DAppChain ERC721).

Your Mainnet ERC721 contract doesn't need anything special to work with the Transfer Gateway.
Though you might want to add something like the `depositToGateway` method below to make it a bit easier to transfer tokens into the Mainnet Gateway:

```solidity
pragma solidity ^0.4.24;

import "openzeppelin-solidity/contracts/token/ERC721/ERC721Token.sol";

contract MyAwesomeToken is ERC721Token("MyAwesomeToken", "MAT") {
    // Mainnet Gateway address
    address public gateway;

    constructor(address _gateway) public {
        gateway = _gateway;
    }

    function depositToGateway(uint tokenId) public {
        safeTransferFrom(msg.sender, gateway, tokenId);
    }

}
```

Your DAppChain ERC721 contract must provide a public `mintToGateway` method to allow the DAppChain Gateway to mint tokens that are transferred from Ethereum:

```solidity
pragma solidity ^0.4.24;

import "openzeppelin-solidity/contracts/token/ERC721/ERC721Token.sol";

/**
 * @title Full ERC721 Token for Loom DAppChains
 * This implementation includes all the required and some optional functionality of the ERC721
 * standard, it also contains some required functionality for Loom DAppChain compatibility.
 */
contract MyAwesomeToken is ERC721Token {
    // DAppChain Gateway address
    address public gateway;

    constructor(address _gateway) ERC721Token("MyAwesomeToken", "MAT") public {
        gateway = _gateway;
    }

    // Used by the DAppChain Gateway to mint tokens that have been deposited to the Mainnet Gateway
    function mintToGateway(uint256 _uid) public
    {
        require(msg.sender == gateway);
        _mint(gateway, _uid);
    }
}
```

> The DAppChain Gateway will only attempt to mint tokens it doesn't already own, so if you'd rather
> manage the token supply yourself you can pre-mint them to the DAppChain Gateway instead of
> implementing the `mintToGateway` function.

When you're happy with your contracts, you can deploy them with Truffle to Ethereum and the DAppChain. Before doing so, you may want to take a look at [loom-truffle-doc].


## Mapping Mainnet contracts to DAppChain contracts

Once you've deployed your contracts you'll need to send a request to the DAppChain Gateway to create a mapping between them. When the DAppChain Gateway is notified that a token from your Mainnet ERC721 contract has been deposited to the Mainnet Gateway, it will mint a matching token in your DAppChain ERC721 contract (unless the token already exists on the DAppChain). The DAppChain Gateway will refuse to create a contract mapping unless you provide proof that you deployed both contracts.

To prove that you deployed the Mainnet ERC721 contract you must provide a signature generated by signing a message using the Ethereum private key you used to deploy the contract and a hash of the Mainnet transaction that deployed the contract.

To prove that you deployed the DAppChain ERC721 contract you simply need to sign the request sent to the DAppChain Gateway using the DAppChain private key you used to deploy the contract. The DAppChain Gateway will verify that the sender of the request deployed the contract by looking up
the contract creator in the DAppChain contract registry.

After a contract mapping request is received by the DAppChain Gateway, there will be a small delay before it is picked up by the Gateway Oracle. The Gateway Oracle will lookup the transaction that deployed the Mainnet contract to find out who really deployed it, and will then submit its findings back to the DAppChain Gateway, which will either approve the requested mapping or simply throw it out.

## ERC721 token transfer to the DAppChain

Alice has managed to acquire one of your awesome tokens on Mainnet and now wants to transfer it to her DAppChain account. Before she can do so, she must send a request to the Address Mapper contract to create a mapping between her Ethereum and DAppChain accounts. Like the DAppChain Gateway, the Address Mapper will refuse to create an account mapping unless Alice provides proof that she is the owner of both accounts.

To prove that she owns her Mainnet account, Alice must provide a signature generated by signing a message using the Ethereum private key associated with the account. And to prove she owns her DAppChain account, she just needs to sign the request she sends to the DAppChain Gateway using the DAppChain private key associated with her account. As soon as the Address Mapper receives the request it will create the requested account mapping. At this point, Alice can start transferring tokens between her Ethereum and DAppChain accounts.


## ERC721 token transfer to Ethereum

Alice has had her fun on the DAppChain so she wants to transfer her token from her DAppChain account back to her Mainnet account. First, she must grant approval to the DAppChain Gateway to take over ownership of the token she wants to transfer. She can do this by sending a request to the DAppChain ERC721 contract.
Next, Alice should send a request to the DAppChain Gateway to start the token withdrawal process.
When the DAppChain Gateway receives the request it creates a pending withdrawal record for Alice. Then, it waits for the Gateway Oracle to sign the pending withdrawal. After a small delay, the Gateway Oracle signs the pending withdrawal and submits the signature to the DAppChain Gateway. In turn, the DAppChain Gateway emits an event to notify Alice that her pending withdrawal has been signed.

To complete the withdrawal process Alice must provide the withdrawal signature (generated by the Gateway Oracle) to the Mainnet Gateway, which then transfers the corresponding token to Alice's Mainnet account.


## Summary

You should now have a basic understanding of how the Transfer Gateway works, though we haven't presented nor explained any of the actual API yet. If you haven't already, take a look at the following pages:

 - [Transfer Gateway Testnet Tutorial](extdev-transfer-gateway.html).
 - [Transfer Gateway Example Projects](transfer-gateway-example).
