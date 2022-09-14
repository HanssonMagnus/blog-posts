---
layout: post
title: Running an Erigon Archive Node
---

# Running an Erigon Archive Node

## Resources
* [Erigon](https://github.com/ledgerwatch/erigon)
* [Erigon Programmer's Guide](https://github.com/ledgerwatch/erigon/blob/devel/docs/programmers_guide/guide.md)
* [Erigon YouTube channel](https://www.youtube.com/channel/UC706WC--fhg4O-o6eHs2Wvg)
* [Getting Started with Erigon on Ubuntu](https://chasewright.com/getting-started-with-turbo-geth-on-ubuntu/)
* [Otterscan](https://github.com/wmitsuda/otterscan)
* [Towards the first release of Turbo-geth](https://ledgerwatch.github.io/turbo_geth_release.html)
* [TrueBlocks](https://github.com/TrueBlocks/trueblocks-core)

## Update 2022-09-14 (Be ready for the Merge)
Ethereum Foundation recently announced that "It is strongly advised to stay up-to-date on the
latest available versions of clients being used. This can be done by continue monitoring client
repositories, client mailing lists and client social media channels for updates."

It is thus recommended to use
[Erigon v2022.09.01-alpha](https://github.com/ledgerwatch/erigon/releases/tag/v2022.09.01) as of
now. If you have previously been on Erigon `beta` you will have to re-sync the DB from scratch,
otherwise it should be compatible.

Further, for an updated version of how to config Erigon as a service see this blog post:
* [How to set up Erigon, Erigon's RPC and TrueBlocks (scrape and API) as services](./2022-01-10-Erigon-Trueblocks)

## Hardware
Hardware recommendations are 2Tb storage space on a single partition (1.6Tb state and 200GB temp
files) and 16GB RAM. The more RAM you have the faster Erigon is able to sync. With 16GB of RAM the
sync takes around 6 days, with 32GB 5 days and with 64GB approximately 4 days.

## Documentation and support
```
./build/bin/erigon --help
```

## Erigon database
The Erigon database is a key-value store organised in tables (also commonly referred to as
"buckets"). This is the list of the main tables in the database,

Table | Description
--- | ---
Headers | This table stores information about block headers
Block Bodies | Transactions and Ommers
Header Numbers | Mapping of 32-byte header/block hashes to the corresponding block numbers
Receipts | Records the list of transaction receipts for each block
PlainState | Store together Accounts and Storage
History Of Accounts | The history of accounts records how the accounts changed at each block
Change Sets | Records the history of changes in accounts and contract storage
HashedState | Used to calculate Merkle Trie root hash of whole current state
IntermediateTrieHashes | Used to calculate Merkle Trie root hash of whole current state
Tx Senders | List of transaction sender addresses for each transaction

[This](https://github.com/ledgerwatch/erigon/blob/devel/docs/programmers_guide/db_walkthrough.MD)
document explains the main tables in more detail and how Erigon organize the database and how it is
different from Geth. There is also a
[DB FAQ](https://github.com/ledgerwatch/erigon/blob/devel/docs/programmers_guide/db_faq.md).

## Installation
Clone git repo,
```
git clone --recurse-submodules -j8 https://github.com/ledgerwatch/erigon.git
```

The default path that Erigon saves the chain data is `~/.local/share/erigon` and you can change
this with the `--datadir` option.
```
cd erigon
make erigon
./build/bin/erigon --datadir mainnet
```
Note the --datadir option that allows you to store Erigon files in a non-default location,
in this example, in mainnet subdirectory of the current directory. Name of the directory --datadir
does not have to match the name of the chain in --chain.

Erigon syncs mainnet by default and is an archive node by default. Erigon has a pruning option if
you do not want to sync the whole history.

When the syncing is starting the following info will show,
```
INFO[12-27|19:11:50.532] Build info                               git_branch=...
INFO[12-27|19:11:50.533] Starting Erigon on Ethereum mainnet...
INFO[12-27|19:11:50.536] Maximum peer count                       ETH=100 total=100
INFO[12-27|19:11:50.536] Set global gas cap                       cap=50000000
INFO[12-27|19:11:50.599] Opening Database                         label=chaindata
path=/media/node/erigon/erigon/mainnet/chaindata
```
After this the [staged sync](https://github.com/ledgerwatch/erigon/blob/devel/eth/stagedsync/README.md)
starts. The log of the `./build/bin/erigon --datadir mainnet` command will be displayed in the
console. The logs can be saved by piping them to a file, e.g.,
```
./build/bin/erigon --datadir mainnet 2>&1 | tee erigon.log
```

Once the sync is up to date the node continues to update the database with the
newest block.

## Monitor the sync
With `htop` you can monitor processes in real time.

## Shutting down Erigon and start syncing again
It is possible to shut down Erigon and then start syncing again by starting Erigon and pointing to
the database file. There is also a
[testing document](https://github.com/ledgerwatch/erigon/blob/devel/TESTING.md) with instructions
on how to proceed when Erigon is updated.

## Run Erigon and rpcdaemon as Systemd services
Both `./build/bin/erigon` and `./build/bin/rpcdaemon` can be run as services, e.g., Systemd or
supervisor. This can help
to connect the node to other applications. Setting up the rpcdaemon as a service we'll create a
file called `/etc/systemd/system/erigon-rpc.service`,
```
[Unit]
Description=Erigon RPC Daemon

[Service]
WorkingDirectory=/media/node/erigon/erigon/
ExecStart=/media/node/erigon/erigon/build/bin/rpcdaemon --datadir=mainnet
    +++--private.api.addr=localhost:9090 --http.api=eth,erigon,web3,net,
    +++debug,trace,txpool --ws
User=magnus
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Now you can start the service with `sudo systemctl start erigon-rpc` and check if the service is
working properly with `systemctl status erigon-rpc`.

Once the service is running you can send a `curl` command to see if things are working. `rpcdaemon`
is by default using `https://localhost:8045`,
```
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method":
"eth_blockNumber", "params": [], "id":1}' localhost:8545
```
should return something like,
```
{"jsonrpc":"2.0","id":1,"result":"0xd51b03"}
```

## Accessing the Ethereum node data
There are several ways of accessing data stored in Ethereum nodes,

- JSON RPC API seems to be the prevalent way.
- LMDB python bindings. If you want to access the data from the same computer where an
  Erigon process is running, you can use LDMB to open turbo-geth’s database in a read-only
  mode.
- Remote interface to the database.
- REST API daemon. Such daemon can be run on the same or a different computer from the
  node. REST API daemon accesses the live database locally or remotely, via read-only
  interface.

## Accessing the Erigon node with Ethers.js
You can also access the node data by Ethers, using the local node as a provider. Here is the code I
tried for that,
```
// Comment in console
console.log("Starting!");

// Import libraries
import { ethers } from "ethers";

// Set provider (Ethereum node)
const provider = new ethers.providers.WebSocketProvider('ws://127.0.0.1:8545');

// Set transaction
const tx_hash = '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4';

// Define function to retrive tx
async function localTx() {
    const tx = await provider.getTransaction(tx_hash)
    const receipt = await provider.getTransactionReceipt(tx_hash)
    console.log(tx)
    console.log(receipt)
    console.log('Complete!')
};

// Export main function
export const run = () => {
  localTx().catch(err => console.error(err));
};
```
This will only work if you use the `--ws` flag when starting the rpcdaemon service.

## Otterscan block explorer
Otterscan is an Ethereum block explorer designed to be run locally with Erigon. Otterscan is a
patched version of Erigon that you add as an additional remote to your existing git repo.
```
git remote add otterscan https://github.com/wmitsuda/erigon.git
```
Check which tag your repo has with,
```
git tag
```
For example, if you are running Erigon from v2021.07.01 tag, checkout the tag v2021.07.01-otterscan
and rebuild rpcdaemon,
```
git fetch --all
git fetch otterscan --tags
git checkout <version-tag-otterscan>
make rpcdaemon
```
