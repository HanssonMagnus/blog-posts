---
layout: post
title: TrueBlocks
---

# TrueBlocks

## Resources
* [TrueBlocks](https://trueblocks.io/)
* [TrueBlocks: A long winded explanation](https://trueblocks.io/blog/a-long-winded-explanation-of-trueblocks/)

## Introduction
TrueBlocks is a software that creates an index of the transactions in an Ethereum archive node
(works best with Erigon).
This gives access to the data in a way not otherwise possible, in that you can in an instance query
all transactions of a given address. If you run an archive node (e.g., Erigon) a common way to
access its data is through a web socket connection using a web3 application such as Ethers.js. It is
possible to query transaction and block data, but if you would like to create a data set with
all transactions using the Uniswap smart contract there is no fast way to do this. However,
Trueblocks solves this issue by letting us query "every appearance of every address in the block",
this includes "from" and "to" but also traces.

## Installation
Up-to-date installation instructions can be found
[here](https://trueblocks.io/docs/install/install-trueblocks/#how-do-i-install-packages-on-the-command-line)
and below an overview of the installation process is given.

Install Go and dependencies,
```
sudo apt install build-essential git cmake python python-dev libcurl3-dev clang-format jq
```
Clone and compile TrueBlocks,
```
git clone -b develop https://github.com/trueblocks/trueblocks-core
cd trueblocks-core
git checkout develop
mkdir build && cd build
cmake ../src
make
```
Now add `trueblocks-core/bin` to your shell PATH. Go to TrueBlocks configuration file at
`~/.local/share/trueblocks` and edit `trueblocks.toml` to add your RPC provider. This is by default
set to `http://localhost:8545`. In this config file you can also add an Etherscan key and change
the `indexPath` (the index is around 90GB).
Now we can test if the installation has worked,
```
chifra blocks 12345
```

## Build or/and download the index
You can build your own index or you can download an already existing index from IPFS or you can
mix and match these two options. To download
the index can be a good option if you do not run your own node or if you want to get started
faster. One drawback to downloading the index is that you can only query data up until the point
the index have been uploaded. However this can be updated by your own indexing.
`chifra init` and `chifra pins` are two commands that let you download the index. If you want to
download the whole index and not just the Bloom filters you can use,
```
chifra init --all
```

`chifra scrape` allows you to build your own index and it takes a few days.

## CLI tool Chifra
`chifra` is the TrueBlocks cli command that let us interact with the software. `chifra --help`
gives a list of flags and descriptions.

From the TrueBlocks documentation, if we want to search for all appearances of an address we first
remove the address from cash such that we do not search from a known place, then we search,
```
chifra monitors --delete 0xf503017d7baf7fbc0fff7492b751025c6a78179b
chifra list 0xf503017d7baf7fbc0fff7492b751025c6a78179b
```
This will create a small "monitor" file consisting of the block number and transaction index of
each appearance. In order to export the transaction details including the receipts and logs we run,
```
chifra export 0xf503017d7baf7fbc0fff7492b751025c6a78179b
```

## TrueBlocks explorer
TrueBlocks has an [GUI explorer](https://github.com/TrueBlocks/trueblocks-explorer) that
provides a front end for the TrueBlocks backend.
