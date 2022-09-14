---
layout: post
title: Running an Ethereum node
---

# Running an Ethereum node

## Resources
- [Erigon](https://github.com/ledgerwatch/erigon)
- [Ethereum Nodes and Clients](https://ethereum.org/en/developers/docs/nodes-and-clients/)
- [Etherscan Node Tracker](https://etherscan.io/nodetracker#)
- [The Difference Between TrueBlocks and theGraph](https://trueblocks.io/papers/2021/the-difference-between-trueBlocks-and-rotki-and-trueBlocks-and-thegraph.pdf)
- [Towards the first release of Turbo-geth](https://ledgerwatch.github.io/turbo_geth_release.html#)
- [TrueBlocks](https://trueblocks.io/)
- [TrueBlocks Explorer](https://github.com/TrueBlocks/trueblocks-explorer)

## Nodes
Ethereum is a transaction based distributed state machine that is running on "nodes". Nodes verify
transactions and data on the network and you need a "client" on your computer to run a node.

From the Etherscan node tracker one can see that at the moment there are 2,927 nodes running.
97.8% of the nodes are running Geth on Linux.

All clients implement the
[JSON-RPC](https://playground.open-rpc.org/?schemaUrl=https://raw.githubusercontent.com/ethereum/eth1.0-apis/assembled-spec/openrpc.json&uiSchema%5BappBar%5D%5Bui:splitView%5D=true&uiSchema%5BappBar%5D%5Bui:input%5D=false&uiSchema%5BappBar%5D%5Bui:examplesDropdown%5D=false)
specification to standardize endpoints that other applications can rely on.

There are three ways of running an Ethereum node: Full node, light node and archive node. These
different types consume data differently. The nodes can also have different (faster or slower) ways
of syncing with the network.

It is important with diversification in both nodes and clients. Diversification in nodes means
that, e.g., there are enough full nodes to support light nodes and make the network secure.
Diversification in clients means that different code bases are run all following the Ethereum
protocol but have independent designs. This makes sure that if there is a bug in one client the
network is still secure running on the others. E.g., Geth (Go), OpenEthereum (Rust), Nethermind
(C#,.NET), Besu (Java) and Erigon (Go).

## Full node
A full node is a node that can interact with the network, verify blocks, post transactions and look
at the current state. However, a full node does not allow you to search the blockchain for
historical paths of addresses and fully query the blockchain in an efficient way. Technically you
can derive all states from a full node but it takes time.

## Light node
A light node is good for weaker devices such as mobile phones and only stores the header chain.
Light nodes partially verify data via the state roots in the block headers and they can request more
information from other nodes if needed.

## Archive node
An archive node has all information as a full node plus it keeps all blocks of the blockchain and
builds a history of all the states. Archive nodes are good for chain analysis as it lets you query
the blochchain.

#### Erigon
Erigon (previously named TurboGeth) is a fork of GoEthereum (Geth) focusing on speed and disk
efficiency. Erigon has provided a solution that creates an archive node with under `2TB` disc
space and under 3 days of syncing.

## Hardware requirements
A regular computer with enough disk space is enough to run a node. Even ARM architecture such as a
Raspberry Pi works.

As of writing, Recommended requirements: Fast CPU with 4+ cores, `16+GB` RAM, SSD and fast internet
connection. The faster the RAM, disk and connection the faster the sync time.
As of disk space it varies depending on which client you use, if you are using Erigon
`2TB` is enough, whereas if you run Geth or OpenEthereum you would need `6TB`.

If you have a server or desktop computer you will be able to run a node in the background that will
be synced as long as the computer is on. This is a great way to be able to interact with Ethereum
without relying on others, e.g., you can point MetaMask to your node instead of having it point to
publicly hosted services. You will also be contributing to the network. Another option is to have
a dedicated computer running a node. There are also companies selling pre configured nodes:
DappNode and Avado.

## Running a cloud node
If you do not want to run a local node there is the option to run a node in the cloud. There are
dedicated services for this including: QuikNode, Blockdaemon and LunaNode.

## Node as a Service
[Node as a Service](https://ethereum.org/en/developers/docs/nodes-and-clients/nodes-as-a-service/)
can be an alternative if you are not able to run your own node. These services usually provide an
API such that you can interact with the node and the network. Popular Node as a Service options include:
Alchemy, BlockDaemon, Chainstack, GetBlock, InfStones, Infura, Moralis, Pocket Network, QuikNode
and Rivet.

## Layers on top of a node
It is difficult to get good quality and understandable data from the Ethereum blockchain.
Once you have an Ethereum archive node and you want to analyze the data it contains it is a good
idea to use an indexing tool. An analogy is to the internet where Google is indexing the data such
that you can more easily query it.

#### TrueBlock
[TrueBlocks](https://trueblocks.io/) is an open source tool
that builds an index of the Ethereum blockchain such that you can analyze it on your local machine.
TrueBlocks works great with an archive node such as Erigon.

Block explorers such as Etherscan allows you to access the blockchain but comes with drawbacks
including: limited number of queries, privacy concerns and data quality. One of the core design
values of TrueBlocks is that "No one should have to rely on anyone other than themselves for data,
this including users who want data at the very end of the data journey".

From a research point of view it is beneficial to be able to index your own blockchain data for
reproducibility.
