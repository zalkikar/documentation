---
layout: ../../../layouts/MainLayout.astro
section: dataFeeds
date: Last Modified
title: "Proof of Reserve Feeds"
isIndex: true
permalink: "docs/data-feeds/proof-of-reserve/"
whatsnext: { "Find contract addresses for Proof of Reserve Feeds": "/data-feeds/proof-of-reserve/addresses/" }
setup: |
  import ClickToZoom from "@components/ClickToZoom.astro"
  import { Aside } from "~/components"
---

Chainlink Proof of Reserve Feeds provide the status of the reserves for several assets. You can use these feeds the same way that you read other Data Feeds. Specify the [Proof of Reserve Feed Address](/data-feeds/proof-of-reserve/addresses/) that you want to read instead of specifying a Price Feed address. See the [Using Data Feeds](/data-feeds/using-data-feeds/) page to learn more.

To find a list of available Proof of Reserve Feeds, see the [Proof of Reserve Feed Addresses](/data-feeds/proof-of-reserve/addresses/) page.

## Types of Proof of Reserve Feeds

Reserves are available for both cross-chain assets and off-chain assets. This categorization describes the data attestation variations of Proof of Reserve feeds and helps highlight some of the inherent market risks surrounding the data quality of these feeds.

Reserves are available for both off-chain assets and cross-chain assets. Token issuers prove the reserves for their assets through several different methods.

### Off-chain reserves

Off-Chain reserves are sourced from APIs through an [external adapter](/chainlink-nodes/external-adapters/external-adapters).

<ClickToZoom src='/images/data-feed/off-chain-reserves.webp' />

Off-chain reserves provide their data using the following methods:

- Third-party: An auditor, accounting firm, or other third party attests to reserves. This is done by combining both fiat and investment assets into a numeric value that is reported against the token.
- Custodian: Reserves data are pulled directly from the bank or custodian. The custodian has direct access to the bank or vault holding the assets. Generally, this works when the underlying asset pulled requires no additional valuation and is simply reported on-chain.
- ⚠️ Self-attested: Reserve data is read from an API that the token issuer hosts. Self-attested feeds carry additional risk.

### Cross-chain reserves

Cross-chain reserves are sourced from the network where the reserves are held. Chainlink node operators can report cross-chain reserves by running an [external adapter](/chainlink-nodes/external-adapters/external-adapters) and querying the source-chain client directly. In some instances, the reserves are composed of a dynamic list of IDs or addresses using a composite adapter.

<ClickToZoom src='/images/data-feed/cross-chain-reserves.webp' />

Cross-chain reserves provide their data using the following methods:

- Wallet address manager: The project uses the [IPoRAddressList](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/interfaces/PoRAddressList.sol) wallet address manager contract and self-attests to which addresses they own.
- Wallet address: The project attests which addresses they own through a self-hosted API.

## Using Proof of Reserve Feeds

Read answers from Proof of Reserve Feeds the same way that you read other Data Feeds. Specify the [Proof of Reserve Feed Address](/data-feeds/proof-of-reserve/addresses/) that you want to read instead of specifying a Price Feed address. See the [Using Data Feeds](/data-feeds/using-data-feeds/) page to learn more.

Using Solidity, your smart contract should reference [`AggregatorV3Interface`](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol), which defines the external functions implemented by Data Feeds.

::solidity-remix[samples/PriceFeeds/ReserveConsumerV3.sol]

<Aside title="Disclaimer" type="caution">
  <p>
    Proof of Reserve feeds can vary in their configurations. Please be careful with the configuration of the feeds
    used by your smart contracts. You are solely responsible for reviewing the quality of the data (e.g., a Proof
    of Reserve feed) that you integrate into your smart contracts and assume full responsibility for any damage,
    injury, or any other loss caused by your use of the feeds used by your smart contracts.
    <a href="/data-feeds/selecting-data-feeds#risk-mitigation">
    Learn more about making responsible data quality decisions.
    </a>
  </p>
</Aside>
