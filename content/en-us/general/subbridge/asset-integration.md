---
title: 'Asset Integration Guide'
weight: 1004
menu:
  general:
    parent: "general-bridge"
---

## Overview

SubBridge supports the integration of both parachain assets and EVM assets, making it possible to transfer them between parachains and EVM chains. All supported assets are whitelisted, which means it would be failed if you transfer assets through SubBridge without registration. Check [**Supported Assets**](../supported-assets) to find all the integrated assets on SubBridge so far.

> A parachain asset means the asset is reserved on a parachian. For example, both KAR and aUSD are reserved on Karura.
> 

> An EVM asset means the asset is reserved on an EVM chain, in general, it's an ERC20 smart contract deployed on EVM. **Note we currently only support assets deployed on Ethereum and Moonriver EVM.**
> 

You need to do the following steps to finish integration with SubBridge:

- Open an issue on [subbridge-integration](https://github.com/Phala-Network/subbridge-integration) repo to provide assets registry information
- Test registration locally or on Rococo
- Register assets on the Khala network
- Integrate SubBridge into your app[Optional]

## Open an issue to provide assets registry information

If you are interested in integrating your asset into SubBridge, please go to [integration repo](https://github.com/Phala-Network/subbridge-integration) to open an issue with the template below:

```markdown
### Who we are

*** Brief introduction of your project and team ***

### Asset registry informations

- name
- symbol
- decimals
- location(represented as a MultiLocation)
- contract address(EVM assets only)
- is mintable(EVM assets only)
```

> `location` must be a MultiLocation, for example, the location of `KAR` is `MultiLocation::new(1, X2(Parachain(2000), General(0x0080)))`. If your asset is an EVM asset, the `location` should be `MultiLocation::new(1, X3(Parachain(2004), GeneralIndex(chain_id), GeneralKey(erc20_address)))`. `chain_id` should match the chain your asset deployed on. So far chain id of Ethereum is `0`, and chain id of Moonriver EVM is `2`.
> 

> `is mintable` represents whether your ERC20 smart contract support `mint`   and `burn`, if set as true, your users will need to give the mint authority to our contract when the first time they use SubBridge. Check [**Assets Registration**](../technical-details/#asset-registration) find more details about what we have done on registration.
> 

After the issue opened, feel free to notify the team on our Discord channel if we haven't given you a response for a long time. Now you are ready to work with the team to finish the Integration stuff when we approve the request on the issue.

## Test registration locally or on Rococo

Before onboarding your asset to the Khala network, we must make sure everything works properly on the [Rhala network](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frhala-api.phala.network%2Fws#/explorer), which is our test parachain on Rococo. For parachain assets, you can choose to test the registration locally before running a test with the Phala team on Rococo. We have provided a [config file](https://github.com/Phala-Network/subbridge-integration/blob/main/thala_karura.config.json) which contains the information on how to run our test runtime `Thala` with `polkadot-launch`. Check [here](https://github.com/paritytech/polkadot-launch) to find more about `polkadot-launch`.

For both parachain assets and EVM assets, the necessary step is to test the whole integration stuff on Rococo with the Phala team. The parachain registration on Rococo is maintained by Parity, if you are a parachain asset and you haven't been a Rococo parachain, please **notice the Parity team to onboard your chain on Rococo, also don't forget to let them open HRMP channels between your parachain and Rhala network**. If you are an EVM asset, please **deploy an ERC20 asset on Rinkeby in advance, which will be used to config the EVM bridge.** The architecture of the test net is as follow:

<p>
    <img src="/images/general/subbridge-rhala.png" style="background-color:white;" alt>
    <figcaption align = "center">Rhala Testnet Architecture</figcaption>
</p>

As shown, we have deployed a test ChainBridge that bridges the Rhala network and Rinkeby, contracts information is as below:

| Contract      | Address |
| ----------- | ----------- |
| bridge      | 0x0712Cf53B9fA1A33018d180a4AbcC7f1803F55f4       |
| erc20Handler   | 0x30f8549A46afa69f3F9b46e3C28F456796E6Cf37        |

To start integration testing, please t**ransfer some assets to our test account in advance**

- For parachain asset, please transfer 10000 of your asset to account `0x7804e66ec9eea3d8daf6273ffbe0a8af25a8879cf43f14d0ebbb30941f578242` on your parachain
- For EVM asset, please transfer 10000 of your asset(ERC20) to account `0xA29D4E0F035cb50C0d78c8CeBb56Ca292616Ab20` on Rinkeby

Now it's time to let us know that you have everything ready to do the test with the Phala team on Rococo. We will register your asset on the Rhala network and enable ChainBridge transfer if you are an EVM asset or you certainly want to bridge your parachain asset to EVM chains. Check [**Assets Registration**](../technical-details/#asset-registration) to find more details.

## Register assets on the Khala network

After we have confirmed all functions are working properly on Rococo, we will start integrating your asset on the Khala network. If your asset is a parachian asset, please refer to [**Parachain assets integration guide**](#parachain-assets-integration-guide) and if your assets is an EVM asset, please refer to [**EVM assets integration guide**](#evm-assets-integration-guide).

### Parachain assets integration guide

SubBridge contains an XCM-based bridge that can let assets transferred between parachians. If the parachian your asset reserved hasn't opened HRMP channel with Khala network, the first thing we should do is follow these steps to open bi-direction HRMP channels:

<aside>
💡 Step1 - Step3 actually is the stuff to open HRMP channels between our two parachains
</aside>

**Step1: Construct the call to send a request to open the HRMP channel to the Khala network**

- Head to: [https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkusama.api.onfinality.io%2Fpublic-ws#/extrinsics](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkusama.api.onfinality.io%2Fpublic-ws#/extrinsics)
- Choose extrinsic with parameters: *hrmp -> hrmpInitOpenChannel { recipient: 2004 , proposedMaxCapacity: 1000, proposedMaxMessageSize: 102400}*
- Copy the encoded call data, it should be `0x3c00d4070000e803000000900100`

<p>
    <img src="/images/general/subbridge-openhrmp.png" style="background-color:white;" alt>
    <figcaption align = "center">Send HRMP Open Request</figcaption>
</p>

**Step2: Construct an XCM message and send it to the relay chain(Kusama)**

The XCM message you constructed should contain the following 5 instructions:

- WithdrawAsset
- BuyExecution
- Transact
- RefundSurplus
- DepositAsset

Here is an example to send the request to Moonriver from Khala `0x03000821040101009d1f02000000210001010002140004000000000700e876481713000000000700e876481700060002286bee383c00e7070000e803000000900100140d0100040001010070617261d4070000000000000000000000000000000000000000000000000000`, you can decode the call at page [https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkhala.api.onfinality.io%2Fpublic-ws#/extrinsics/decode](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkhala.api.onfinality.io%2Fpublic-ws#/extrinsics/decode)

**Tips:**

- Make sure the XCM message is sent from the root account (either SUDO or via governance)
- Make sure your parachian sovereign account has enough KSM on the Kusama network
- The destination should be (1, Here)
- Asset id(MultiLocation) should be set to (0, Here)
- Recipient of `DepositAsset` should be an `AccountId32` and put your parachain sovereign account public key to id field.

<aside>
💡 Please let us know when the XCM message is executed successfully on Kusama, then we will propose to send an XCM message to receive the request and send open the request to your parachain at the same time.

</aside>

**Step3: Receive HRMP open request from Khala network**

HRMP channel is a one-way channel, which means you also need to receive our open request on your side or your parachain will not be able to receive messages from our parachain. The steps to receive the request from the the Khala network are similar to the open request we did before. When the Khala network successfully sends the open request, you should do as follow:

- Head to [https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkusama.api.onfinality.io%2Fpublic-ws#/extrinsics](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkusama.api.onfinality.io%2Fpublic-ws#/extrinsics)
- Choose extrinsic with parameters: *hrmp -> hrmpAcceptOpenChannel { sender: 2004 }*
- Copy the encoded call data, it should be 0x3c01d4070000


<p>
    <img src="/images/general/subbridge-accepthrmp.png" style="background-color:white;" alt>
    <figcaption align = "center">Accept HRMP Open Request</figcaption>
</p>

**Step4: Register PHA on your parachain**

Since the HRMP channel already opened between our two parachains, the last thing we need to do is to register your asset into our parachain and register our asset(PHA) into your parachain. You can refer to [**Assets Registration**](../technical-details/#asset-registration) to see what we have done to register an asset. Please note that the location of `PHA` is `**MultiLocation::new(1, X1(Parachain(2004))**`. You may want to add it to your runtime XCM trader config according to your implementation.

### EVM assets integration guide

If the smart contract of your asset is ERC20 compatible, we can easily integrate it into SubBridge. Currently, we have integrated [ChainBridge](https://chainbridge.chainsafe.io/) as our EVM bridge. It already deployed on both Ethereum mainnet and Moonriver EVM, here is the deployed ChainBridge contracts information:

**Ethereum mainnet:**

| Contract      | Address |
| ----------- | ----------- |
| bridge      | 0x8F92e7353b180937895E0C5937d616E8ea1A2Bb9       |
| erc20Handler   | 0xEEc0fb4913119567cDfC0c5fc2Bf8f9F9B226c2d        |

**Moonriver EVM:**

| Contract      | Address |
| ----------- | ----------- |
| bridge      | 0xCe6652551A989C13B41f70cac504341A5F711c8d       |
| erc20Handler   | 0xf88337a0db6e24Dff0fCD7F92ab0655B97A68d38        |

**Relayers:**

| Relayer      | Address |
| ----------- | ----------- |
| relayer1      | 0xA97Dc452Ca3699c4Eb62171FE2f994ff7aE48400       |
| relayer2   | 0xdca0f5B3686cc87415100808a2568879fE74E01a        |
| relayer2   | 0x4EE535bE2ce432151916E36B3c684E1dB8Cbf8c1        |

Compare with the registration of parachain assets, registration of EVM assets is more convenient. You should be relaxed because we will do everything need with the registration information provided by you. If you are aware of things we did when registering an asset, please refer to [**Assets Registration**](../technical-details/#asset-registration) for more information.

<aside>
💡 You should receive the update when we finish the registration.

</aside>

## Integrate SubBridge into your app

It's better to integrate SubBridge into your App if you'd like to let your users use it easily. Integrating SubBridge into your app is super easy, the only thing you need to do is follow the `dest` specification we defined previously on your client. Different between doing crosschain transfer on a parachain and on an EVM chain is that the former you will need to interact with the XCM implementation pallet on that parachain while in the latter you will need to interact with the smart contract deployed by Phala.

- Issuing crosschain transfer on other parachains or Khala network

For example, when issuing a crosschain transfer to the Khala network or to EVM chains from Karura, you should use the [xtoken>>transfer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkarura.polkawallet.io#/extrinsics), while issuing a crosschain transfer to other parachains or EVM chains from Khala, you should use the [xtransfer>>transfer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkhala.api.onfinality.io%2Fpublic-ws#/extrinsics). So interfaces may be different between different parachains depends on its implementation. 

- Issuing crosschain transfer on EVM chains with ChainBridge

So far we only support ChainBridge as an EVM bridge(will integrate more EVM bridges in the future), so the following is based on interaction with our ChainBridge contract. According to the mechanism of the ERC20 protocol, before calling `Bridge.deposit` which is the interface to issue crosschain transfer we need to call [ERC20 approve](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-approve-address-uint256-) method to let user approve the specific amount of assets to our `ERC20Handler` contract. This operation will allow `ERC20Handler` to spend at most of the number of asset from the user account. According to the `deposit` method defined in our [bridge contract](https://github.com/Phala-Network/chainbridge-solidity/blob/5eef3073ccc75b48e06ce44eee522c2023da974e/contracts/Bridge.sol#L312), there are three parameters you need to provide:

> The first one is `destinationChainID`, so far you can only transfer to the Khala network from EVM, the value would always be `1`.
> 

> The second one is `resourceID`, this is a 32 bytes indentation representing the EVM asset on a specific chain, which means the same asset with a different chain will have a different `resourceId`. The way we generate the `resourceId` can be found at [here](https://github.com/Phala-Network/khala-parachain/blob/5ab4f77163c811fb4a02d337791ce669b41481ad/pallets/assets-registry/src/lib.rs#L141).
> 

> The third one is `data`, it is a combination of 32 bytes transfer `amount` and dynamic recipient bytes. The recipient should be an encoded `MultiLocation`.
> 

We prepared the following 6 examples that almost covered all of the scenarios. If the transaction is issued on parachain, we have provided an encoded call you can decode on https://polkadot.js.app, and if the transaction is issued on EVM chains, we have provided a code snippet.

**Example1: transfer an asset from Khala network to other parachain**

An example of transferring 100 PHA from the Khala network to Karura, the recipient is `Alice`:

```jsx
0x5200000000000b00407a10f35a010200411f0100d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d0100bca06501000000
```

**Example2: transfer an asset from Khala network to another EVM chain**

An example of transferring 400 PHA from Khala network to Ethereum, recipient is `0xA29D4E0F035cb50C0d78c8CeBb56Ca292616Ab20`. Note 300 PHA will be deducted as the fee of ChainBridge:

```jsx
0x5200000000000b00407a10f35a00030608636205000650a29d4e0f035cb50c0d78c8cebb56ca292616ab2000
```

**Example3: transfer an asset from parachain to Khala network**

An example of transferring 100 PHA from Karura to Khala network, the recipient is `Alice`. Karura XCM transfer based on the implementation of [xtoken pallet]https://github.com/open-web3-stack/open-runtime-module-library/tree/master/xtokens):

```jsx
0x360000aa00407a10f35a0000000000000000000001010200511f0100d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d00bca06501000000
```

**Example4: transfer an asset from parachain to EVM chain**

An example of transferring 400 PHA from Karura to Ethereum, recipient is `0xA29D4E0F035cb50C0d78c8CeBb56Ca292616Ab20`. Note 300 PHA will be deducted as the fee of ChainBridge:

```jsx
0x360000aa00407a10f35a0000000000000000000001010400511f0608636205000650a29d4e0f035cb50c0d78c8cebb56ca292616ab2000bca06501000000 
```

**Example5: transfer an asset from EVM to Khala network**

An example of transferring 100 PHA from Ethereum to the Khala network, the recipient is Alice. Note example code is interact with Bridge contract deployed on Rinkeby test net. Check the [ChainBridge deployment] for more information.

[View snippet code](https://gist.github.com/tolak/20d71a78ae650cb9f54dc7d4eaaab11f)

**Example6: transfer an asset from EVM chain to parachain**

An example of transferring 100 PHA from Ethereum to Karura, the recipient is `Alice`. 

[View snippet code](https://gist.github.com/tolak/ce6b12af0a22b994ed65b533edf4a4ce)

<aside>
💡 🥳Cheer! After all of these steps, your asset is ready to be transferred between parachains and EVM chains.

</aside>
