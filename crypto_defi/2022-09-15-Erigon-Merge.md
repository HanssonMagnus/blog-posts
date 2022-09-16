---
layout: post
title: Running an Erigon After the Merge
---

# Running Erigon After the Merge

## Resources
* [Ethereum Beacon Chain Checkpoint Sync Endpoints](https://eth-clients.github.io/checkpoint-sync-endpoints/)
* [Ethereum: Spin Up Your Own Ethereum Node](https://ethereum.org/en/developers/docs/nodes-and-clients/run-a-node/)
* [Erigon](https://github.com/ledgerwatch/erigon)
* [Lighthouse docs](https://lighthouse-book.sigmaprime.io/intro.html)
* [Mainnet Merge Announcement](https://blog.ethereum.org/2022/08/24/mainnet-merge-announcement)

The above resources hold valuable information and links to further resources, and is recommended
reading.

## Introduction
```
Sep 15 08:42:44 e erigon[31611]: [INFO] [09-15|08:42:44.174] Commit cycle                             in=818.880865ms
Sep 15 08:42:44 e erigon[31611]: [INFO] [09-15|08:42:44.174] RPC Daemon notified of new headers       from=15537391 to=15537392 header sending=7.323µs log sending=261ns
Sep 15 08:42:44 e erigon[31611]: [INFO] [09-15|08:42:44.174] [1/15 Headers] Waiting for headers...    from=15537392
Sep 15 08:42:50 e erigon[31611]: [INFO] [09-15|08:42:50.880]
Sep 15 08:42:50 e erigon[31611]:     ,,,         ,,,                                               ,,,         ,,,
Sep 15 08:42:50 e erigon[31611]:   ;"   ^;     ;'   ",                                           ;"   ^;     ;'   ",
Sep 15 08:42:50 e erigon[31611]:   ;    s$$$$$$$s     ;                                          ;    s$$$$$$$s     ;
Sep 15 08:42:50 e erigon[31611]:   ,  ss$$$$$$$$$$s  ,'  ooooooooo.    .oooooo.   .oooooo..o     ,  ss$$$$$$$$$$s  ,'
Sep 15 08:42:50 e erigon[31611]:   ;s$$$$$$$$$$$$$$$     '888   'Y88. d8P'  'Y8b d8P'    'Y8     ;s$$$$$$$$$$$$$$$
Sep 15 08:42:50 e erigon[31611]:   $$$$$$$$$$$$$$$$$$     888   .d88'888      888Y88bo.          $$$$$$$$$$$$$$$$$$
Sep 15 08:42:50 e erigon[31611]:  $$$$P""Y$$$Y""W$$$$$    888ooo88P' 888      888 '"Y8888o.     $$$$P""Y$$$Y""W$$$$$
Sep 15 08:42:50 e erigon[31611]:  $$$$  p"$$$"q  $$$$$    888        888      888     '"Y88b    $$$$  p"$$$"q  $$$$$
Sep 15 08:42:50 e erigon[31611]:  $$$$  .$$$$$.  $$$$     888        '88b    d88'oo     .d8P    $$$$  .$$$$$.  $$$$
Sep 15 08:42:50 e erigon[31611]:   $$DcaU$$$$$$$$$$      o888o        'Y8bood8P' 8""88888P'      $$DcaU$$$$$$$$$$
Sep 15 08:42:50 e erigon[31611]:     "Y$$$"*"$$$Y"                                                 "Y$$$"*"$$$Y"
Sep 15 08:42:50 e erigon[31611]:         "$b.$$"                                                       "$b.$$"
Sep 15 08:42:50 e erigon[31611]:        .o.                   .   o8o                         .                 .o8
Sep 15 08:42:50 e erigon[31611]:       .888.                .o8   '"'                       .o8                "888
Sep 15 08:42:50 e erigon[31611]:      .8"888.     .ooooo. .o888oooooo oooo    ooo .oooo.  .o888oo .ooooo.  .oooo888
Sep 15 08:42:50 e erigon[31611]:     .8' '888.   d88' '"Y8  888  '888  '88.  .8' 'P  )88b   888  d88' '88bd88' '888
Sep 15 08:42:50 e erigon[31611]:    .88ooo8888.  888        888   888   '88..8'   .oP"888   888  888ooo888888   888
Sep 15 08:42:50 e erigon[31611]:   .8'     '888. 888   .o8  888 . 888    '888'   d8(  888   888 .888    .o888   888
Sep 15 08:42:50 e erigon[31611]:  o88o     o8888o'Y8bod8P'  "888"o888o    '8'    'Y888""8o  "888"'Y8bod8P''Y8bod88P"
Sep 15 08:42:50 e erigon[31611]:
Sep 15 08:42:50 e erigon[31611]: [INFO] [09-15|08:42:50.882] [1/15 Headers] Processed                 highest inserted=15537393 age=8s
Sep 15 08:42:50 e erigon[31611]: [INFO] [09-15|08:42:50.933] [6/15 Execution] Completed on            block=15537393
```

This morning Ethereum merged with the Beacon Chain and is now running with PoS. With this, there is
one important change if you run an archive node,

- Node operators must now run both a consensus layer (CL) client and an execution layer (EL)
  client to remain on the network after the Merge.

Previously, on PoW, it was sufficient to run an EL client to run an archive or full node. However,
after the Merge an Ethereum full node is a combination of a CL client and an EL client, these
communicate with each other through a new
[Engine JSON-RPC API](https://github.com/ethereum/execution-apis/tree/main/src/engine), and
authenticate each other using a JWT secret. The clients should run on the same machine for
stability.

From Ethereum Foundation: "In other words, if you were already running a node on the Beacon Chain,
you now also need to run an execution layer client. Similarly, if you were running a node on the
current proof-of-work network, you will need to run a consensus layer client."

As of now, there are 5 CL clients that support the Merge: Lighthouse, Lodestar, Nimbus, Prysm, and
Teku. In this post I will run a `Lighthouse v3.1.0` consensus client.


## Install Lighthouse consensus client

#### Install Rust
First, you have to install the [Rust programming language](https://www.rust-lang.org/tools/install),

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

In the Rust development environment, all tools are installed to the ~/.cargo/bin directory, and
we need to update the search path,

```bash
source $HOME/.cargo/env
```

#### Install Lighthouse
First, we need to install the following packages,

```bash
sudo apt install -y git gcc g++ make cmake pkg-config llvm-dev libclang-dev clang
```

Then we can build Lighthouse from source by,

```bash
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
git checkout stable
make
```

and we can check that we have the correct version with `lighthouse --version`,
```
Lighthouse v3.1.0-aa022f4
```

In the future if we want to update Lighthouse we can run,
```bash
cd lighthouse
git fetch
git checkout ${VERSION}
make
```

The Lighthouse node must be connected to an execution client for the execution client (Erigon)
to process transactions beyond block `15537393`. The authentication that is used to connect the
clients is [JWT](https://jwt.io/). The EL client must expose the JWT to a port
that is not used before, the default port used here is `8551`.

From Erigon's Github: "Once Erigon is running, you need to point your CL client to <erigon address>:8551,
where <erigon address> is either localhost or the IP address of the device running Erigon, and also
point to the JWT secret path created by Erigon."

#### Sync lighthouse
In order for Lighthouse to work with Erigon it needs to be synced. This will take a day or so, and
if you set up Lighthouse for the first time you can use a `checkpoint` to sync from. You will add
the flag `--checkpoint-sync-url <URL>` to your Lighthouse command, where `<URL>` is an endpoint.
You can find endpoints at [this website](https://eth-clients.github.io/checkpoint-sync-endpoints/).
An example is `--checkpoint-sync-url https://beaconstate.ethstaker.cc`.

If you get the following in the logs `INFO Starting checkpoint sync, remote_url:
https://beaconstate.ethstaker.cc/, service:...` it means that the sync started from the checkpoint.
However, if you get, e.g, `INFO Refusing to checkpoint sync, msg: database already exists, use
--purge-db to force checkpoint sync`, the checkpoint sync did not work, and as the message says you
can add the flag `--purge-db` such that you do not have to sync from scratch.

From Lighhouse:
Once forwards sync completes, Lighthouse will commence a "backfill sync" to download the blocks
from the checkpoint back to genesis. The beacon node will log messages similar to the following
each minute while it completes backfill sync,
```
INFO Downloading historical blocks  est_time: 5 hrs 0 mins, speed: 111.96 slots/sec, distance:
2020451 slots (40 weeks 0 days), service: slot_notifier
```

## Install Erigon
For building the latest alpha release (this will be suitable for most users just wanting to run a
node),

```
git clone https://github.com/ledgerwatch/erigon.git
cd erigon
git checkout alpha
make erigon
./build/bin/erigon
```

If you have previously run the `beta` version of Erigon (you can find releases
[here](https://github.com/ledgerwatch/erigon/releases)) then you will have to re-sync the DB, and
this will take a few days depending on your system.

## Set up Lighthouse and Erigon as Systemd Services
Most Linux distributions run `systemd` as init system, and it handles services on the machine. We
can set up Erigon and Lighthouse as services such that they run in the background. If you are
unfamiliar with Systemd Services you can check out my
[blog post](https://magnushansson.xyz/blog_posts/software/2022-01-08-Systemd-service) about it.

We will create two files called `erigon.service` and `lighthouse.service` in `/etc/systemd/system`,
```bash
cd /etc/systemd/system
sudo touch erigon.service
sudo touch lighthouse.service
```

We will then edit (`sudo vim erigon.service` and `sudo vim lighthouse.service`) the files in the
following ways,

```bash
[Unit]
Description=Erigon Node
After=network.target network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=/media/node/erigon/erigon/
ExecStart=/media/node/erigon/erigon/build/bin/erigon \
    --datadir=mainnet \
    --private.api.addr=localhost:9090 \
    --authrpc.jwtsecret=/media/node/erigon/erigon/mainnet/jwt.hex \
    --http \
    --ws \
    --http.api=engine,eth,erigon,web3,net,debug,trace,txpool,shh

User=magnus
Restart=always
RestartSec=5s

# Output to syslog
StandardOutput=syslog
StandardError=syslog
#Change this to find app logs in /var/log/syslog
SyslogIdentifier=erigon

[Install]
WantedBy=multi-user.target
```

Note that the newer version of Erigon has the http server activated by default. If you previously
ran on Erigon beta, you can now put some flags from the RPC daemon into Erigon, and the stand alone
RPC daemon is no longer necessary to `curl` the node.

```bash
[Unit]
Description=Lighthouse CL Client
After=network.target network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/magnus/.cargo/bin/lighthouse beacon_node \
	--datadir /media/node/erigon/erigon/mainnet \
	--network mainnet \
	--http \
	--execution-endpoint http://localhost:8551 \
	--execution-jwt /media/node/erigon/erigon/mainnet/jwt.hex \
        --checkpoint-sync-url https://beaconstate.ethstaker.cc \

User=magnus
Restart=always
RestartSec=5s

# Output to syslog
StandardOutput=syslog
StandardError=syslog
#Change this to find app logs in /var/log/syslog
SyslogIdentifier=lighthouse

[Install]
WantedBy=multi-user.target
```

Now we can start Lighthouse and Erigon with,

```bash
sudo systemctl start erigon
sudo systemctl start lighthouse
```

## Monitor processes
We can monitor the logs continuously with,
```bash
sudo journalctl -u erigon -f
```
and,
```bash
sudo journalctl -u lighthouse -f
```

If you want to monitor your complete `syslog` you can run,
```
sudo tail -f /var/log/syslog
```

## Check if Erigon is up to date
To check if the archive node is fully synced we can query the latest block, convert it from hex to
int, and compare it to the latest validated block at [Etherscan](https://etherscan.io/),
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method":
"eth_blockNumber", "params": [], "id":1}' localhost:8545 | jq -r ".result" | mawk '{ printf "%d\n",
$1 }'
```
Here we can see that my node's latest block is `15544659` and Etherscan shows `15544659`, and we
are thus up to date.
