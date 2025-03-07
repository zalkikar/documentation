---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Forwarder tutorial"
metadata:
  title: "Chainlink Node Operators: Forwarder tutorial"
  description: "Use a forwarder contract for more security and flexibility."
setup: |
  import CodeSample from "@components/CodeSample/CodeSample.astro"
  import ClickToZoom from "@components/ClickToZoom.astro"
---

:::note[Prerequisites]
This guide assumes you have a running Chainlink node. See the [Running a Chainlink Node locally](/chainlink-nodes/v1/running-a-chainlink-node) guide to learn how to run a Chainlink node. **Note**: For this example to work, you must use Chainlink node version _1.12.0_ or above.

Also, you must be familiar with these concepts:

- [Forwarder](/chainlink-nodes/contracts/forwarder) contracts.
- [Operator](/chainlink-nodes/contracts/operator) contracts.
- [OperatorFactory](/chainlink-nodes/contracts/operatorfactory) contracts.
  :::

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forwarder-directrequest-example.png' />

In this tutorial, you will configure your Chainlink node with a simple transaction-sending strategy on the Sepolia testnet:

- Your node has two externally owned accounts (EOA).
- Your node has two [direct request](/chainlink-nodes/oracle-jobs/job-types/direct_request) jobs. One job returns _uint256_, and the other returns _string_.
- Each job uses a different EOA.
- You use a [forwarder](/chainlink-nodes/contracts/forwarder) contract to fulfill requests with two EOAs that look like a single address.

:::note
Here you are using a forwarder contract and two EOAs for two [direct request](/chainlink-nodes/oracle-jobs/job-types/direct_request) jobs. You can use the same strategy for different job types, VRF and OCR. Supporting different job types on the same Chainlink node reduces your infrastructure and maintenance costs.
:::

## Check your Chainlink node is running

On your terminal, check that your Chainlink node is running:

```shell
 docker ps -a -f name=chainlink

 CONTAINER ID   IMAGE                            COMMAND               CREATED         STATUS                   PORTS                    NAMES
 feff39f340d6   smartcontract/chainlink:1.12.0   "chainlink local n"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:6688->6688/tcp   chainlink
```

Your Chainlink Operator Interface is accessible on [http://localhost:6688](http://localhost:6688). If using a VPS, you can create a [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/) to your node for `6688:localhost:6688` to enable connectivity to the GUI. Typically this is done with `ssh -i $KEY $USER@$REMOTE-IP -L 6688:localhost:6688 -N`. A SSH tunnel is recommended over opening up ports specific to the Chainlink node to be public facing. See the [Security and Operation Best Practices](/chainlink-nodes/resources/best-security-practices/) page for more details on how to secure your node.

If you don't have a running Chainlink node, follow the [Running a Chainlink Node Locally](/chainlink-nodes/v1/running-a-chainlink-node) guide.

## Set up multiple EOAs

1. Access the shell of your Chainlink node container:

   ```shell
   docker exec -it chainlink /bin/bash
   ```

   ```shell
   chainlink@1d095e4ceb09:~$
   ```

1. You can now log in by running:

   ```shell
   chainlink admin login
   ```

   You will be prompted to enter your API email and password. If successful, the prompt will appear again.

1. Check the number of available EOA:

   ```shell
   chainlink keys eth list
   ```

   You should see one EOA:

   ```shell
   🔑 ETH keys
   -------------------------------------------------------------------------------------------------
   Address:           0x71a1Eb6534054E75F0D6fD0A3B0A336228DD5cFc
   EVM Chain ID:      11155111
   Next Nonce:        0
   ETH:               0.000000000000000000
   LINK:              0
   Disabled:          false
   Created:           2023-03-02 09:28:26.872791 +0000 UTC
   Updated:           2023-03-02 09:28:26.872791 +0000 UTC
   Max Gas Price Wei: 115792089237316195423570985008687907853269984665640564039457584007913129639935chainlink@22480bec8986
   ```

1. Create a new EOA for your Chainlink node:

   ```shell
   chainlink keys eth create
   ```

   ```shell
   ETH key created.

   🔑 New key
   -------------------------------------------------------------------------------------------------
   Address:           0x259c49E65644a020C2A642260a4ffB0CD862cb24
   EVM Chain ID:      11155111
   Next Nonce:        0
   ETH:               0.000000000000000000
   LINK:              0
   Disabled:          false
   Created:           2023-03-02 11:36:48.717074 +0000 UTC
   Updated:           2023-03-02 11:36:48.717074 +0000 UTC
   Max Gas Price Wei: 115792089237316195423570985008687907853269984665640564039457584007913129639935
   ```

1. At this point, there are two EOAs:

   ```shell
   chainlink keys eth list
   ```

   ```shell
   🔑 ETH keys
   -------------------------------------------------------------------------------------------------
   Address:           0x259c49E65644a020C2A642260a4ffB0CD862cb24
   EVM Chain ID:      11155111
   Next Nonce:        0
   ETH:               0.000000000000000000
   LINK:              0
   Disabled:          false
   Created:           2023-03-02 11:36:48.717074 +0000 UTC
   Updated:           2023-03-02 11:36:48.717074 +0000 UTC
   Max Gas Price Wei: 115792089237316195423570985008687907853269984665640564039457584007913129639935
   -------------------------------------------------------------------------------------------------
   Address:           0x71a1Eb6534054E75F0D6fD0A3B0A336228DD5cFc
   EVM Chain ID:      11155111
   Next Nonce:        0
   ETH:               0.000000000000000000
   LINK:              0
   Disabled:          false
   Created:           2023-03-02 09:28:26.872791 +0000 UTC
   Updated:           2023-03-02 09:28:26.872791 +0000 UTC
   Max Gas Price Wei: 115792089237316195423570985008687907853269984665640564039457584007913129639935chainlink@22480bec8986
   ```

1. Fund the two addresses with _0.5_ Sepolia ETH each. You can obtain testnet ETH from the faucets listed on the [Link Token Contracts](/resources/link-token-contracts/) page.
1. Note the two addresses, as you will need them later.

## Deploy operator and forwarder

Use the [operator factory](/chainlink-nodes/contracts/operatorfactory) to deploy both the [forwarder](/chainlink-nodes/contracts/forwarder) and the [operator](/chainlink-nodes/contracts/operator) contracts. You can find the factory address for each network on the [addresses](/chainlink-nodes/contracts/addresses) page.

1. Open contract [0x447Fd5eC2D383091C22B8549cb231a3bAD6d3fAf](https://sepolia.etherscan.io/address/0x447Fd5eC2D383091C22B8549cb231a3bAD6d3fAf) to display the factory in the Sepolia block explorer.
1. Click the _Contract_ tab. Then, click _Write Contract_ to display the _write_ transactions on the factory.

   <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/factorytransactions.jpg' />

1. Click the _Connect to Web3_ button to connect your wallet.
   <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/factorytransactionsconnected.jpg' />

1. Click the `deployNewOperatorAndForwarder` function to expand it and then click the _Write_ button to run the function. Metamask prompts you to confirm the transaction.

1. Click _View your transaction_. _Etherscan_ will open a new tab. Wait for the transaction to be successful.
   <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/factoryviewtransaction.jpg' />

      <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/deployfromfactorysuccess.jpg' />

1. On the _Transaction Details_ page, click _Logs_ to display the list of transaction events. Notice the `OperatorCreated` and `AuthorizedForwarderCreated` events.

   <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/factorycreatetransactions.jpg' />

1. Right-click on each contract address and open it in a new tab.
1. At this point, you should have one tab displaying the operator contract and one tab displaying the forwarder contract.

   <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/operator.jpg' />
      <ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forwarder.jpg' />

1. Record the operator and forwarder addresses. You will need them later.

## Access control setup

As explained in the [forwarder](/chainlink-nodes/contracts/forwarder) page:

- The owner of a forwarder contract is an [operator](/chainlink-nodes/contracts/operator) contract. The owner of the operator contract is a more secure address, such as a hardware wallet or a multisig wallet. Therefore, node operators can manage a set of forwarder contracts through an operator contract using a secure account such as hardware or a multisig wallet.
- Forwarder contracts distinguish between owners and authorized senders. Authorized senders are hot wallets (Chainlink nodes' EOAs).

For this example to run, you will have to:

- Allow the forwarder contract to call the operator's [fulfillOracleRequest2](/chainlink-nodes/contracts/operator#fulfilloraclerequest2) function by calling the [setauthorizedsenders](/chainlink-nodes/contracts/receiver#setauthorizedsenders) function on the operator contract. Specify the forwarder address as a parameter.
- Allow the two Chainlink node EOAs to call the forwarder's [forward](/chainlink-nodes/contracts/forwarder#forward) function. Because the operator contract owns the forwarder contract, call [acceptAuthorizedReceivers](/chainlink-nodes/contracts/operator#acceptauthorizedreceivers) on the operator contract. Specify the forwarder contract address and the two Chainlink node EOAs as parameters. This call makes the operator contract accept ownership of the forwarder contract and authorizes the Chainlink node EOAs to call the forwarder contract by calling [setauthorizedsenders](/chainlink-nodes/contracts/receiver#setauthorizedsenders).

### Whitelist the forwarder

In the blockchain explorer, view the operator contract and call the `setAuthorizedSenders` method with the address of your forwarder contract. The parameter is an array. For example, `["0xA3f07D6773514480b918C2742b027b3acD9E44fA"]`. Metamask prompts you to confirm the transaction.

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/operatorsetauthorizedsenders.jpg' />

### Whitelist the Chainlink node EOAs

In the blockchain explorer, view the operator contract and call the `acceptAuthorizedReceivers` method with the following parameters:

- `targets`: Specify an array of forwarder addresses. For example, `["0xA3f07D6773514480b918C2742b027b3acD9E44fA"]`
- `sender`: Specify an array with the two Chainlink node EOAs. For example, `["0x259c49E65644a020C2A642260a4ffB0CD862cb24","0x71a1Eb6534054E75F0D6fD0A3B0A336228DD5cFc"]`.

Metamask prompts you to confirm the transaction.

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/acceptAuthorizedReceivers.jpg' />

## Activate the forwarder

1. In the shell of your Chainlink node, enable the forwarder using the Chainlink CLI. Replace `forwarderAddress` with the forwarder address that was created by the factory. Replace `chainId` with the EVM chain ID. (`11155111` for Sepolia):

   ```shell
   chainlink forwarders track --address forwarderAddress --evmChainID chainId
   ```

   In this example, the command is:

   ```shell
   chainlink forwarders track --address 0xA3f07D6773514480b918C2742b027b3acD9E44fA --evmChainID 11155111
   ```

   ```shell
   Forwarder created
   ------------------------------------------------------
   ID:         1
   Address:    0xA3f07D6773514480b918C2742b027b3acD9E44fA
   Chain ID:   11155111
   Created At: 2023-03-02T11:41:43Zchainlink@22480bec8986
   ```

1. Exit the shell of your Chainlink node:

   ```shell
   exit
   ```

1. Stop your Chainlink node:

   ```shell
   docker stop chainlink && docker rm chainlink
   ```

1. Update your environment file. If you followed [Running a Chainlink Node locally](/chainlink-nodes/v1/running-a-chainlink-node) guide then add `ETH_USE_FORWARDERS=true` and `FEATURE_LOG_POLLER=true` to your environment file:

   ```shell
   echo "ETH_USE_FORWARDERS=true
   FEATURE_LOG_POLLER=true" >> ~/.chainlink-sepolia/.env
   ```

   - `ETH_USE_FORWARDERS` enables sending transactions through forwarder contracts.
   - `FEATURE_LOG_POLLER` enables polling forwarder contracts logs to detect any changes to the authorized senders.

1. Start the Chainlink node:

   ```shell
   cd ~/.chainlink-sepolia && docker run --platform linux/x86_64/v8 --name chainlink  -v ~/.chainlink-sepolia:/chainlink -it --env-file=.env -p 6688:6688 smartcontract/chainlink:1.12.0 local n
   ```

1. You will be prompted to enter the key store password:

   ```shell
   2022-12-22T16:18:04.706Z [WARN]  P2P_LISTEN_PORT was not set, listening on random port 59763. A new random port will be generated on every boot, for stability it is recommended to set P2P_LISTEN_PORT to a fixed value in your environment config/p2p_v1_config.go:84       logger=1.10.0@aeb8c80.GeneralConfig p2pPort=59763
   2022-12-22T16:18:04.708Z [DEBUG] Off-chain reporting disabled                       chainlink/application.go:373     logger=1.10.0@aeb8c80
   2022-12-22T16:18:04.709Z [DEBUG] Off-chain reporting v2 disabled                    chainlink/application.go:422     logger=1.10.0@aeb8c80
   Enter key store password:
   ```

1. Enter the key store password and wait for your Chainlink node to start.

## Create directRequest jobs

This section is similar to the [Fulfilling Requests](/chainlink-nodes/v1/fulfilling-requests) guide.

1. Open the Chainlink Operator UI.
1. On the **Jobs** tab, click **New Job**.
1. Create the `uint256` job. Replace:

   - `YOUR_OPERATOR_CONTRACT_ADDRESS` with the address of your deployed operator contract address.
   - `EOA_ADDRESS` with the **first** Chainlink node EOA.

     <CodeSample src="samples/ChainlinkNodes/forwarder/get-uint256.toml"/>

1. Create the `string` job. Replace:

   - `YOUR_OPERATOR_CONTRACT_ADDRESS` with the address of your deployed operator contract address.
   - `EOA_ADDRESS` with the **second** Chainlink node EOA.

     <CodeSample src="samples/ChainlinkNodes/forwarder/get-string.toml"/>

1. After you create the jobs, record the two job IDs.

:::note
Note that both jobs have the attribute `forwardingAllowed = true`. This attribute will make the jobs attempt to run through the forwarder before falling back to default mode. For example, if the Chainlink node tracks no forwarder, the job falls back to the default mode.
:::

## Test the transaction-sending strategy

### Create API requests

1. Open [APIConsumerForwarder.sol](https://remix.ethereum.org/#url=https://docs.chain.link/samples/APIRequests/APIConsumerForwarder.sol) in the Remix IDE.

1. Note that `setChainlinkToken(0x779877A7B0D9E8603169DdbD7836e478b4624789)` is configured for _Sepolia_.

1. On the **Compiler** tab, click the **Compile** button for `APIConsumerForwarder.sol`.

1. On the **Deploy and Run** tab, configure the following settings:

   - Select _Injected Provider_ as your environment. Make sure your metamask is connected to Sepolia.
   - Select _APIConsumerForwarder_ from the **Contract** menu.

1. Click **Deploy**. MetaMask prompts you to confirm the transaction.

1. Fund the contract by sending LINK to the contract's address. See the [Fund Your Contracts](/resources/fund-your-contract/) page for instructions. The address for the `ATestnetConsumer` contract is on the list of your deployed contracts in Remix. You can fund your contract with 1 LINK.

1. After you fund the contract, create a request. Input your operator contract address and the job ID for the `Get > Uint256` job into the `requestEthereumPrice` request method **without dashes**. The job ID is the `externalJobID` parameter, which you can find on your job's definition page in the Node Operators UI.

1. Click the **transact** button for the `requestEthereumPrice` function and approve the transaction in Metamask. The `requestEthereumPrice` function asks the node to retrieve `uint256` data specifically from [https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD](https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD).

1. After the transaction processes, open the **Runs** page in the Node Operators UI. You can see the details for the completed the job run.

1. In Remix, click the `currentPrice` variable to see the current price updated on your consumer contract.

1. Input your operator contract address and the job ID for the `Get > String` job into the `requestFirstId` request method **without dashes**. The job ID is the `externalJobID` parameter, which you can find on your job's definition page in the Node Operators UI.

1. Click the **transact** button for the `requestFirstId` function and approve the transaction in Metamask. The `requestFirstId` function asks the node to retrieve the first `id` from [https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&per_page=10](https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&per_page=10).

1. After the transaction processes, you can see the details for the complete the job run the **Runs** page in the Node Operators UI.

1. In Remix, click the `id` variable to see the current price updated on your consumer contract.

### Check the forwarder

Confirm the following information:

- The Chainlink node submitted the callbacks to the forwarder contract.
- Each callback used a different account.
- The forwarder contract forwarded the callbacks to the operator contract.

Open your forwarder contract in the block explorer. Two [forward](/chainlink-nodes/contracts/forwarder#forward) transactions exist.

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forward-transactions.jpg' />

Click on both transaction hashes and note that the `From` addresses differ. In this example, `0x259c49E65644a020C2A642260a4ffB0CD862cb24` is the EOA used in the `uint256` job, and `0x71a1Eb6534054E75F0D6fD0A3B0A336228DD5cFc` is the EOA used in the `string` job:

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forward1.jpg' />

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forward2.jpg' />

Then click on the `Logs` tab to display the list of events. Notice the [OracleResponse](/chainlink-nodes/contracts/operator#oracleresponse) events. The operator contract emitted them when the Forward contract called it

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forward1-logs.jpg' />

<ClickToZoom src='/images/chainlink-nodes/node-operators/forwarder/forward2-logs.jpg' />
