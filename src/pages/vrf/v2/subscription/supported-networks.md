---
layout: ../../../../layouts/MainLayout.astro
section: vrf
date: Last Modified
title: "Configuration"
permalink: "docs/vrf/v2/subscription/supported-networks/"
metadata:
  title: "Chainlink VRF v2 Supported Networks"
  linkToWallet: true
  image:
    0: "/files/OpenGraph_V3.png"
setup: |
  import VrfCommon from "@features/vrf/v2/common/VrfCommon.astro"
  import ResourcesCallout from "@features/resources/callouts/ResourcesCallout.astro"
---

<VrfCommon callout="subscription"/>

Chainlink VRF allows you to integrate provably fair and verifiably random data in your smart contract.

For implementation details, read [Introduction to Chainlink VRF](/vrf/v2/introduction/).

## Coordinator Parameters

These parameters are configured in the coordinator contract. You can view these values by running `getConfig` on the coordinator or by viewing the coordinator contracts in a blockchain explorer.

- `uint16 minimumRequestConfirmations`: The minimum number of confirmation blocks on VRF requests before oracles respond
- `uint32 maxGasLimit`: The maximum gas limit supported for a `fulfillRandomWords` callback.
- `uint32 stalenessSeconds`: How long the coordinator waits until we consider the ETH/LINK price used for converting gas costs to LINK is stale and use `fallbackWeiPerUnitLink`
- `uint32 gasAfterPaymentCalculation`: How much gas is used outside of the payment calculation. This covers the additional operations required to decrement the subscription balance and increment the balance for the oracle that handled the request.

## Fee Parameters

Fee parameters are configured in the coordinator contract and specify the premium you pay per request in addition to the gas cost for the transaction. You can view them by running `getFeeConfig` on the coordinator. The `uint32 fulfillmentFlatFeeLinkPPMTier1` parameter defines the fees per request specified in millionths of LINK.
The details for calculating the total transaction cost can be found [here](/vrf/v2/subscription/#request-and-receive-data).

## Configurations

VRF v2 coordinators for subscription funding are available on several networks. To see a list of coordinators for direct funding, see the [Direct Funding Configurations](/vrf/v2/direct-funding/supported-networks) page.

<ResourcesCallout callout="bridgeRisks" />

### Ethereum Mainnet

| Item                  | Value                                                                                                                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| LINK Token            | <a class="erc-token-address" id="1_0x514910771AF9Ca656af840dff83E8264EcF986CA" href="https://etherscan.io/token/0x514910771AF9Ca656af840dff83E8264EcF986CA">`0x514910771AF9Ca656af840dff83E8264EcF986CA`</a> |
| VRF Coordinator       | [`0x271682DEB8C4E0901D1a1550aD2e64D568E69909`](https://etherscan.io/address/0x271682DEB8C4E0901D1a1550aD2e64D568E69909)                                                                                      |
| 200 gwei Key Hash     | `0x8af398995b04c28e9951adb9721ef74c74f93e6a478f39e7e0777be13527e7ef`                                                                                                                                         |
| 500 gwei Key Hash     | `0xff8dedfbfa60af186cf3c830acbc32c05aae823045ae5ea7da1e45fbfaba4f92`                                                                                                                                         |
| 1000 gwei Key Hash    | `0x9fe0eebf5e446e3c998ec9bb19951541aee00bb90ea201ae456421a2ded86805`                                                                                                                                         |
| Premium               | 0.25 LINK                                                                                                                                                                                                    |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                    |
| Minimum Confirmations | 3                                                                                                                                                                                                            |
| Maximum Confirmations | 200                                                                                                                                                                                                          |
| Maximum Random Values | 500                                                                                                                                                                                                          |

### Sepolia testnet

:::note[Sepolia Faucets]
Testnet LINK and ETH are available from <a href="https://faucets.chain.link/sepolia">faucets.chain.link</a>.<br/>
Testnet ETH is also available from several public <a href="https://faucetlink.to/sepolia">ETH faucets</a>.
:::

| Item                  | Value                                                                                                                                                                                                                       |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="11155111_0x779877A7B0D9E8603169DdbD7836e478b4624789" href="https://sepolia.etherscan.io/token/0x779877A7B0D9E8603169DdbD7836e478b4624789">`0x779877A7B0D9E8603169DdbD7836e478b4624789`</a> |
| VRF Coordinator       | [`0x8103B0A8A00be2DDC778e6e7eaa21791Cd364625`](https://sepolia.etherscan.io/address/0x8103B0A8A00be2DDC778e6e7eaa21791Cd364625)                                                                                             |
| 30 gwei Key Hash      | `0x474e34a077df58807dbe9c96d3c009b23b3c6d0cce433e59bbf5b34f823bc56c`                                                                                                                                                        |
| Premium               | 0.25 LINK                                                                                                                                                                                                                   |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                                   |
| Minimum Confirmations | 3                                                                                                                                                                                                                           |
| Maximum Confirmations | 200                                                                                                                                                                                                                         |
| Maximum Random Values | 500                                                                                                                                                                                                                         |

### Goerli testnet

:::note[Goerli Faucets]
Testnet LINK is available from https://faucets.chain.link/goerli<br/>
Testnet ETH is available from https://goerlifaucet.com/ or faucets listed at https://faucetlink.to/goerli
:::

| Item                  | Value                                                                                                                                                                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="5_0x326C977E6efc84E512bB9C30f76E30c160eD06FB" href="https://goerli.etherscan.io/token/0x326C977E6efc84E512bB9C30f76E30c160eD06FB">`0x326C977E6efc84E512bB9C30f76E30c160eD06FB`</a> |
| VRF Coordinator       | [`0x2Ca8E0C643bDe4C2E08ab1fA0da3401AdAD7734D`](https://goerli.etherscan.io/address/0x2Ca8E0C643bDe4C2E08ab1fA0da3401AdAD7734D)                                                                                      |
| 150 gwei Key Hash     | `0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15`                                                                                                                                                |
| Premium               | 0.25 LINK                                                                                                                                                                                                           |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                           |
| Minimum Confirmations | 3                                                                                                                                                                                                                   |
| Maximum Confirmations | 200                                                                                                                                                                                                                 |
| Maximum Random Values | 500                                                                                                                                                                                                                 |

### BNB Chain

:::tip[Important]
The LINK provided by the [BNB Chain Bridge](https://www.bnbchain.world/en/bridge) is not ERC-677 compatible, so cannot be used with Chainlink oracles. However, it can be [**converted to the official LINK token on BNB Chain using Chainlink's PegSwap service**](https://pegswap.chain.link/).
:::

| Item                  | Value                                                                                                                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| LINK Token            | <a class="erc-token-address" id="56_0x404460C6A5EdE2D891e8297795264fDe62ADBB75" href="https://bscscan.com/token/0x404460C6A5EdE2D891e8297795264fDe62ADBB75">`0x404460C6A5EdE2D891e8297795264fDe62ADBB75`</a> |
| VRF Coordinator       | [`0xc587d9053cd1118f25F645F9E08BB98c9712A4EE`](https://bscscan.com/address/0xc587d9053cd1118f25F645F9E08BB98c9712A4EE)                                                                                       |
| 200 gwei Key Hash     | `0x114f3da0a805b6a67d6e9cd2ec746f7028f1b7376365af575cfea3550dd1aa04`                                                                                                                                         |
| 500 gwei Key Hash     | `0xba6e730de88d94a5510ae6613898bfb0c3de5d16e609c5b7da808747125506f7`                                                                                                                                         |
| 1000 gwei Key Hash    | `0x17cd473250a9a479dc7f234c64332ed4bc8af9e8ded7556aa6e66d83da49f470`                                                                                                                                         |
| Premium               | 0.005 LINK                                                                                                                                                                                                   |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                    |
| Minimum Confirmations | 3                                                                                                                                                                                                            |
| Maximum Confirmations | 200                                                                                                                                                                                                          |
| Maximum Random Values | 500                                                                                                                                                                                                          |

### BNB Chain testnet

:::note[BNB Chain Faucet]
Testnet LINK is available from https://faucets.chain.link/chapel
:::

| Item                  | Value                                                                                                                                                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="97_0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06" href="https://testnet.bscscan.com/address/0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06">`0x84b9B910527Ad5C03A9Ca831909E21e236EA7b06`</a> |
| VRF Coordinator       | [`0x6A2AAd07396B36Fe02a22b33cf443582f682c82f`](https://testnet.bscscan.com/address/0x6A2AAd07396B36Fe02a22b33cf443582f682c82f)                                                                                         |
| 50 gwei Key Hash      | `0xd4bb89654db74673a187bd804519e65e3f71a52bc55f11da7601a13dcf505314`                                                                                                                                                   |
| Premium               | 0.005 LINK                                                                                                                                                                                                             |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                              |
| Minimum Confirmations | 3                                                                                                                                                                                                                      |
| Maximum Confirmations | 200                                                                                                                                                                                                                    |
| Maximum Random Values | 500                                                                                                                                                                                                                    |

### Polygon (Matic) mainnet

:::tip[Important]
The LINK provided by the [Polygon (Matic) Bridge](https://wallet.polygon.technology/polygon/bridge) is not ERC-677 compatible, so cannot be used with Chainlink oracles. However, it can be [**converted to the official LINK token on Polygon (Matic) using Chainlink's PegSwap service**](https://pegswap.chain.link/)
:::

| Item                  | Value                                                                                                                                                                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="137_0xb0897686c545045aFc77CF20eC7A532E3120E0F1" href="https://polygonscan.com/address/0xb0897686c545045aFc77CF20eC7A532E3120E0F1">`0xb0897686c545045aFc77CF20eC7A532E3120E0F1`</a> |
| VRF Coordinator       | [`0xAE975071Be8F8eE67addBC1A82488F1C24858067`](https://polygonscan.com/address/0xAE975071Be8F8eE67addBC1A82488F1C24858067)                                                                                          |
| 200 gwei Key Hash     | `0x6e099d640cde6de9d40ac749b4b594126b0169747122711109c9985d47751f93`                                                                                                                                                |
| 500 gwei Key Hash     | `0xcc294a196eeeb44da2888d17c0625cc88d70d9760a69d58d853ba6581a9ab0cd`                                                                                                                                                |
| 1000 gwei Key Hash    | `0xd729dc84e21ae57ffb6be0053bf2b0668aa2aaf300a2a7b2ddf7dc0bb6e875a8`                                                                                                                                                |
| Premium               | 0.0005 LINK                                                                                                                                                                                                         |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                           |
| Minimum Confirmations | 3                                                                                                                                                                                                                   |
| Maximum Confirmations | 200                                                                                                                                                                                                                 |
| Maximum Random Values | 500                                                                                                                                                                                                                 |

### Polygon (Matic) Mumbai testnet

:::note[Mumbai Faucet]
Testnet LINK and MATIC are available from [the Polygon faucet](https://faucet.polygon.technology/) and https://faucets.chain.link/mumbai.
:::

| Item                  | Value                                                                                                                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="80001_0x326C977E6efc84E512bB9C30f76E30c160eD06FB" href="https://mumbai.polygonscan.com/address/0x326C977E6efc84E512bB9C30f76E30c160eD06FB">`0x326C977E6efc84E512bB9C30f76E30c160eD06FB `</a> |
| VRF Coordinator       | [`0x7a1BaC17Ccc5b313516C5E16fb24f7659aA5ebed`](https://mumbai.polygonscan.com/address/0x7a1BaC17Ccc5b313516C5E16fb24f7659aA5ebed)                                                                                             |
| 500 gwei Key Hash     | `0x4b09e658ed251bcafeebbc69400383d49f344ace09b9576fe248bb02c003fe9f`                                                                                                                                                          |
| Premium               | 0.0005 LINK                                                                                                                                                                                                                   |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                                     |
| Minimum Confirmations | 3                                                                                                                                                                                                                             |
| Maximum Confirmations | 200                                                                                                                                                                                                                           |
| Maximum Random Values | 500                                                                                                                                                                                                                           |

### Avalanche mainnet

| Item                  | Value                                                                                                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| LINK Token            | <a class="erc-token-address" id="43114_0x5947BB275c521040051D82396192181b413227A3" href="https://snowtrace.io/address/0x5947BB275c521040051D82396192181b413227A3">`0x5947BB275c521040051D82396192181b413227A3`</a> |
| VRF Coordinator       | [`0xd5D517aBE5cF79B7e95eC98dB0f0277788aFF634`](https://snowtrace.io/address/0xd5D517aBE5cF79B7e95eC98dB0f0277788aFF634)                                                                                            |
| 200 gwei Key Hash     | `0x83250c5584ffa93feb6ee082981c5ebe484c865196750b39835ad4f13780435d`                                                                                                                                               |
| 500 gwei Key Hash     | `0x89630569c9567e43c4fe7b1633258df9f2531b62f2352fa721cf3162ee4ecb46`                                                                                                                                               |
| 1000 gwei Key Hash    | `0x06eb0e2ea7cca202fc7c8258397a36f33d88568d2522b37aaa3b14ff6ee1b696`                                                                                                                                               |
| Premium               | 0.005 LINK                                                                                                                                                                                                         |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                          |
| Minimum Confirmations | 1                                                                                                                                                                                                                  |
| Maximum Confirmations | 200                                                                                                                                                                                                                |
| Maximum Random Values | 500                                                                                                                                                                                                                |

### Avalanche Fuji testnet

:::note[Avax Fuji Faucet]
Testnet LINK is available from https://faucets.chain.link/fuji
:::

| Item                  | Value                                                                                                                                                                                                                      |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="43113_0x0b9d5D9136855f6FEc3c0993feE6E9CE8a297846" href="https://testnet.snowtrace.io/address/0x0b9d5D9136855f6FEc3c0993feE6E9CE8a297846">`0x0b9d5D9136855f6FEc3c0993feE6E9CE8a297846`</a> |
| VRF Coordinator       | [`0x2eD832Ba664535e5886b75D64C46EB9a228C2610`](https://testnet.snowtrace.io/address/0x2eD832Ba664535e5886b75D64C46EB9a228C2610)                                                                                            |
| 300 gwei Key Hash     | `0x354d2f95da55398f44b7cff77da56283d9c6c829a4bdf1bbcaf2ad6a4d081f61`                                                                                                                                                       |
| Premium               | 0.005 LINK                                                                                                                                                                                                                 |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                                  |
| Minimum Confirmations | 1                                                                                                                                                                                                                          |
| Maximum Confirmations | 200                                                                                                                                                                                                                        |
| Maximum Random Values | 500                                                                                                                                                                                                                        |

### Fantom mainnet

:::tip[Important]
You must use ERC-677 LINK on Fantom. ERC-20 LINK will not work with Chainlink services.<br/>
Use [bridge.multichain.org](https://bridge.multichain.org/#/router) to send LINK to the Fantom network and be sure to select LINK-ERC677 as the token you will receive on the Fantom mainnet.
:::

| Item                  | Value                                                                                                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="250_0x6F43FF82CCA38001B6699a8AC47A2d0E66939407" href="https://ftmscan.com/token/0x6F43FF82CCA38001B6699a8AC47A2d0E66939407">`0x6F43FF82CCA38001B6699a8AC47A2d0E66939407`</a> |
| VRF Coordinator       | [`0xd5D517aBE5cF79B7e95eC98dB0f0277788aFF634`](https://ftmscan.com/address/0xd5d517abe5cf79b7e95ec98db0f0277788aff634)                                                                                        |
| 4000 gwei Key Hash    | `0xb4797e686f9a1548b9a2e8c68988d74788e0c4af5899020fb0c47784af76ddfa`                                                                                                                                          |
| 10000 gwei Key Hash   | `0x5881eea62f9876043df723cf89f0c2bb6f950da25e9dfe66995c24f919c8f8ab`                                                                                                                                          |
| 20000 gwei Key Hash   | `0x64ae04e5dba58bc08ba2d53eb33fe95bf71f5002789692fe78fb3778f16121c9`                                                                                                                                          |
| Premium               | 0.0005 LINK                                                                                                                                                                                                   |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                     |
| Minimum Confirmations | 1                                                                                                                                                                                                             |
| Maximum Confirmations | 200                                                                                                                                                                                                           |
| Maximum Random Values | 500                                                                                                                                                                                                           |

### Fantom testnet

:::note[Fantom Testnet Faucet]
Testnet LINK is available from https://faucets.chain.link/fantom-testnet
:::

| Item                  | Value                                                                                                                                                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| LINK Token            | <a class="erc-token-address" id="4002_0xfaFedb041c0DD4fA2Dc0d87a6B0979Ee6FA7af5F" href="https://testnet.ftmscan.com/address/0xfaFedb041c0DD4fA2Dc0d87a6B0979Ee6FA7af5F">`0xfaFedb041c0DD4fA2Dc0d87a6B0979Ee6FA7af5F`</a> |
| VRF Coordinator       | [`0xbd13f08b8352A3635218ab9418E340c60d6Eb418`](https://testnet.ftmscan.com/address/0xbd13f08b8352a3635218ab9418e340c60d6eb418)                                                                                           |
| 3000 gwei Key Hash    | `0x121a143066e0f2f08b620784af77cccb35c6242460b4a8ee251b4b416abaebd4`                                                                                                                                                     |
| Premium               | 0.0005 LINK                                                                                                                                                                                                              |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                                |
| Minimum Confirmations | 1                                                                                                                                                                                                                        |
| Maximum Confirmations | 200                                                                                                                                                                                                                      |
| Maximum Random Values | 500                                                                                                                                                                                                                      |

### Klaytn Baobab testnet

:::note[Klaytn Testnet Faucet]
Testnet LINK is available from [faucets.chain.link](https://faucets.chain.link/klaytn-testnet). Use the [KLAY Faucet](https://baobab.wallet.klaytn.foundation/faucet) to obtain testnet KLAY.
:::

| Item                  | Value                                                                                                                                                                                                                      |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LINK Token            | <a class="erc-token-address" id="1001_0x04c5046A1f4E3fFf094c26dFCAA75eF293932f18" href="https://baobab.scope.klaytn.com/token/0x04c5046A1f4E3fFf094c26dFCAA75eF293932f18">`0x04c5046A1f4E3fFf094c26dFCAA75eF293932f18`</a> |
| VRF Coordinator       | [`0x771143FcB645128b07E41D79D82BE707ad8bDa1C`](https://baobab.scope.klaytn.com/address/0x771143FcB645128b07E41D79D82BE707ad8bDa1C)                                                                                         |
| 750 gwei Key Hash     | `0x9be50e2346ee6abe000e6d3a34245e1d232c669703efc44660a413854427027c`                                                                                                                                                       |
| Premium               | 0.005 LINK                                                                                                                                                                                                                 |
| Max Gas Limit         | 2,500,000                                                                                                                                                                                                                  |
| Minimum Confirmations | 1                                                                                                                                                                                                                          |
| Maximum Confirmations | 200                                                                                                                                                                                                                        |
| Maximum Random Values | 500                                                                                                                                                                                                                        |
