---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Tasks"
permalink: "docs/tasks/"
---

## What is a Task?

:::note[Tasks]
Tasks replace the [core adapters](/chainlink-nodes/oracle-jobs/v1/adapters/) from v1 jobs.
:::

Tasks are a replacement for core adapters that is more flexible. Tasks can be composed in arbitrary order into [pipelines](#writing-pipelines). Pipelines consist of one or more threads of execution where tasks are executed in a well-defined order.

You can use Chainlink's [built-in tasks](/chainlink-nodes/oracle-jobs/all-tasks), or you can create your own [external adapters](/chainlink-nodes/external-adapters/external-adapters/) for tasks which are accessed through a `bridge`.

## Shared attributes

All tasks share a few common attributes:

`index`: when a task has more than one input (or the pipeline overall needs to support more than one final output), and the ordering of the values matters, the index parameter can be used to specify that ordering.

```toml
data_1 [type="http" method=GET url="https://chain.link/eth_usd"       index=0]
data_2 [type="http" method=GET url="https://chain.link/eth_dominance" index=1]
multiword_abi_encode [type="eth_abi_encode" method="fulfill(uint256,uint256)"]

data_1 -> multiword_abi_encode
data_2 -> multiword_abi_encode
```

`timeout`: The maximum duration that the task is allowed to run before it is considered to be errored. Overrides the `maxTaskDuration` value in the job spec.

## Writing pipelines

Pipelines are composed of tasks arranged in a DAG (directed acyclic graph). Pipelines are expressed in [DOT syntax](https://en.wikipedia.org/wiki/DOT_%28graph_description_language%29#Directed_graphs).

Each node in the graph is a task with a user-specified ID and a set of configuration parameters and attributes:

```toml
my_fetch_task [type="http" method=GET url="https://chain.link/eth_usd"]
```

The edges between tasks define how data flows from one task to the next. Some tasks can have multiple inputs, such as `median`. Other tasks are limited to 0 (`http`) or 1 (`jsonparse`).

```toml
data_source_1  [type="http" method=GET url="https://chain.link/eth_usd"]
data_source_2  [type="http" method=GET url="https://coingecko.com/eth_usd"]
medianize_data [type="median"]
submit_to_ea   [type="bridge" name="my_bridge"]

data_source_1 -> medianize_data
data_source_2 -> medianize_data
medianize_data -> submit_to_ea
```

![DAG Example](/images/dag_example.png)
