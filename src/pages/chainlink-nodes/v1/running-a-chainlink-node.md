---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Running a Chainlink Node"
permalink: "docs/running-a-chainlink-node/"
whatsnext:
  {
    "Fulfilling Requests": "/chainlink-nodes/v1/fulfilling-requests/",
    "Requirements": "/chainlink-nodes/resources/requirements/",
    "Optimizing EVM Performance": "/chainlink-nodes/resources/evm-performance-configuration/",
    "Performing System Maintenance": "/chainlink-nodes/resources/performing-system-maintenance/",
    "Miscellaneous": "/chainlink-nodes/resources/miscellaneous/",
    "Security and Operation Best Practices": "/chainlink-nodes/resources/best-security-practices/",
  }
metadata:
  title: "Running a Chainlink Node locally"
  description: "Run your own Chainlink node using this guide which explains the requirements and basics for getting started."
setup: |
  import { Tabs } from "@components/Tabs"
---

This guide will teach you how to run a Chainlink node locally using [Docker](#using-docker). The Chainlink node will be configured to connect to the Ethereum Sepolia or Goerli testnet.

:::note[Running from source]
To run a Chainlink node from source, use the [following instructions](https://github.com/smartcontractkit/chainlink#install). However, it’s recommended to run the Chainlink node with Docker. This is because we continuously build and deploy the code from our repository on Github, which means you don’t need a complete development environment to run a node.
:::

:::note[Other Supported Networks]
Chainlink is blockchain agnostic technology. The [LINK Token Contracts](/resources/link-token-contracts/) page details networks which support the LINK token. You can setup your node to provide data to any of these blockchains.
:::

:::note[Running Chainlink Node on Ganache]
Ganache is a mock testnet. Although you can run nodes on Ganache, it is not officially supported. Most node operators should use one of the supported [testnets](/resources/link-token-contracts/) for development and testing.
:::

## Requirements

- As explained in the [requirements page](/chainlink-nodes/resources/requirements/), make sure there are enough resources to run a Chainlink node and a PostgreSQL database.
- Install [Docker Desktop](https://docs.docker.com/get-docker/). You will run the Chainlink node and PostgreSQL in Docker containers.
- Chainlink nodes must be able to connect to an Ethereum client with an active websocket connection. See [Running an Ethereum Client](/chainlink-nodes/resources/run-an-ethereum-client/) for details. In this tutorial, you can [use an external service](/chainlink-nodes/resources/run-an-ethereum-client/#external-services) as your client.

## Using Docker

### Run PostgreSQL

1. Run PostgreSQL in a Docker container. You can replace `mysecretpassword` with your own password.

   ```shell
   docker run --name cl-postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
   ```

1. Confirm that the container is running. Note the `5432` port is [published](https://docs.docker.com/config/containers/container-networking/#published-ports) `0.0.0.0:5432->5432/tcp` and therefore accessible outside of Docker.

   ```shell
   docker ps -a -f name=cl-postgres

   CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                    NAMES
   dc08cfad2a16   postgres   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:5432->5432/tcp   cl-postgres
   ```

### Run Chainlink node

#### Configure your node

1. Create a local directory to hold the Chainlink data:

   <Tabs client:visible>
      <Fragment slot="tab.1">Sepolia</Fragment>
      <Fragment slot="tab.2">Goerli</Fragment>
      <Fragment slot="panel.1">
      ```shell Sepolia
      mkdir ~/.chainlink-sepolia
      ```
      </Fragment>
      <Fragment slot="panel.2">
      ```shell Goerli
      mkdir ~/.chainlink-goerli
      ```
      </Fragment>
   </Tabs>

1. Run the following as a command to create a `config.toml` file and populate with variables specific to the network you're running on. For a full list of available configuration variables, see the [Node Config](/chainlink-nodes/v1/node-config) page.
   Be sure to update the value for `CHANGEME` to the value given by your [external Ethereum provider](/chainlink-nodes/resources/run-an-ethereum-client/#external-services).

   <Tabs client:visible>
      <Fragment slot="tab.1">Sepolia</Fragment>
      <Fragment slot="tab.2">Goerli</Fragment>
      <Fragment slot="panel.1">
      ```shell Sepolia
      echo "[Log]
      Level = 'warn'

   [WebServer]
   AllowOrigins = '\*'
   SecureCookies = false

   [WebServer.TLS]
   HTTPSPort = 0

   [[EVM]]
   ChainID = '11155111'

   [[EVM.Nodes]]
   Name = 'Sepolia'
   WSURL = 'wss://CHANGE_ME'
   HTTPURL = 'https://CHANGE_ME'
   " > ~/.chainlink-sepolia/config.toml

   ````
   </Fragment>
   <Fragment slot="panel.2">
   ```shell Goerli
   echo "[Log]
   Level = 'warn'

   [WebServer]
   AllowOrigins = '*'
   SecureCookies = false

   [WebServer.TLS]
   HTTPSPort = 0

   [[EVM]]
   ChainID = '5'

   [[EVM.Nodes]]
   Name = 'Goerli'
   WSURL = 'wss://CHANGE_ME'
   HTTPURL = 'https://CHANGE_ME'
   " > ~/.chainlink-goerli/config.toml
   ````

      </Fragment>
   </Tabs>

1. Create a `secrets.toml` file with a keystore password and the URL to your database. Update the value for `mysecretpassword` to the chosen password in [Run PostgreSQL](#run-postgresql). Specify a complex keystore password. This will be your wallet password that you can use to unlock the keystore file generated for you.

   <Tabs client:visible>
      <Fragment slot="tab.1">Sepolia</Fragment>
      <Fragment slot="tab.2">Goerli</Fragment>
      <Fragment slot="panel.1">
      ```shell Sepolia
      echo "[Password]
      Keystore = 'mysecretkeystorepassword'

   [Database]
   URL = 'postgresql://postgres:mysecretpassword@host.docker.internal:5432/postgres?sslmode=disable'
   " > ~/.chainlink-sepolia/secrets.toml

   ````
   </Fragment>
   <Fragment slot="panel.2">
   ```shell Goerli
   echo "[Password]
   Keystore = 'mysecretkeystorepassword'

   [Database]
   URL = 'postgresql://postgres:mysecretpassword@host.docker.internal:5432/postgres?sslmode=disable'
   " > ~/.chainlink-goerli/secrets.toml
   ````

      </Fragment>
   </Tabs>

   :::tip[Important]
   Because you are testing locally, add `?sslmode=disable` to the end of your
   `DATABASE_URL`. However you should _never_ do this on a production node.
   :::

1. Start the Chainlink Node. Now you can run the Docker image. Replace `<version>` with your desired version. Tag versions are available in the [Chainlink docker hub](https://hub.docker.com/r/smartcontract/chainlink/tags). _The `latest` version does not work._

   <Tabs client:visible>
      <Fragment slot="tab.1">Sepolia</Fragment>
      <Fragment slot="tab.2">Goerli</Fragment>
      <Fragment slot="panel.1">
      ```shell Sepolia
      cd ~/.chainlink-sepolia && docker run --platform linux/x86_64/v8 --name chainlink -v ~/.chainlink-sepolia:/chainlink -it -p 6688:6688 --add-host=host.docker.internal:host-gateway smartcontract/chainlink:<version> -config /chainlink/config.toml -secrets /chainlink/secrets.toml node start
      ```
      </Fragment>
      <Fragment slot="panel.2">
      ```shell Goerli
      cd ~/.chainlink-goerli && docker run --platform linux/x86_64/v8 --name chainlink -v ~/.chainlink-goerli:/chainlink -it -p 6688:6688 --add-host=host.docker.internal:host-gateway smartcontract/chainlink:<version> -config /chainlink/config.toml -secrets /chainlink/secrets.toml node start
      ```
      </Fragment>
   </Tabs>

   The first time running the image, the node asks you to enter an API Email and Password. This will be used to expose the API for the GUI interface, and will be used every time you log into your node.

1. Check the container is running. Note the `6688` port is [published](https://docs.docker.com/config/containers/container-networking/#published-ports) `0.0.0.0:6688->6688/tcp` and therefore accessible outside of Docker.

   ```shell
   docker ps -a -f name=chainlink

   CONTAINER ID   IMAGE                            COMMAND                CREATED         STATUS                   PORTS                    NAMES
   feff39f340d6   smartcontract/chainlink:1.13.0   "chainlink node start" 4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:6688->6688/tcp   chainlink
   ```

1. You can now connect to your Chainlink node's UI interface by navigating to [http://localhost:6688](http://localhost:6688). If using a VPS, you can create an [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/) to your node for `6688:localhost:6688` to enable connectivity to the GUI. Typically this is done with `ssh -i $KEY $USER@$REMOTE-IP -L 6688:localhost:6688 -N`. An SSH tunnel is recommended over opening public-facing ports specific to the Chainlink node. See the [Security and Operation Best Practices](/chainlink-nodes/resources/best-security-practices/) page for more details on securing your node.
