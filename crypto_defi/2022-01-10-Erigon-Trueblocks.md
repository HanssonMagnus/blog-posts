---
layout: post
title: How to set up Erigon, Erigon's RPC and TrueBlocks as services
---

# How to set up Erigon, Erigon's RPC and TrueBlocks (scrape and API) as services

## Update 2022-09-10 (Be ready for the Merge)
Ethereum Foundation recently announced that "It is strongly advised to stay up-to-date on the
latest available versions of clients being used. This can be done by continue monitoring client
repositories, client mailing lists and client social media channels for updates."

It is thus recommended to use
[Erigon v2022.09.01-alpha](https://github.com/ledgerwatch/erigon/releases/tag/v2022.09.01) as of
now. If you have previously been on Erigon `beta` you will have to re-sync the DB from scratch,
otherwise it should be compatible.

Further,
I got an email from Aristide Pottier that in the later version of Erigon the http server is
activated by default and we have to deactivate it otherwise the RPC daemon won't be able to deploy
the http API at port 8045. Aristide provided the fix,

```bash
ExecStart=/bin/erigon --datadir=mainnet --private.api.addr=localhost:9090 --http=false
```

A big thank you for helping this post to be up to date!

## Update 2022-07-13
Erigon has had several updates since this blog post was first written. The previous problems with
the alpha branch seems to be resolved, and the latest beta release is depreciated. It is
recommended to run the latest [alpha release](https://github.com/ledgerwatch/erigon/releases). When
you change from beta to alpha you will have to re-sync the DB.
Further, Erigon now has a built in RPC server. The standalone RPC daemon still works, but you need
to change ports from was originally written in this post. I changed the ports in the systemd
service for the RPC daemon from 9090 to [9094](https://github.com/ledgerwatch/erigon#grpc-ports) in
order for it to work.

## Resources
* [Erigon](https://github.com/ledgerwatch/erigon)
* [TrueBlocks](https://trueblocks.io/)

It is highly recommended to read the documentation of Erigon and TrueBlocks. This blog post assumes
that you have Erigon and TrueBlocks installed. These previous blog posts might be of interest:
* [2022-01-09 TrueBlocks](./2022-01-09-Trueblocks)
* [2022-01-08 Systemd Services](../software/2022-01-08-Systemd-service)
* [2021-12-27 Run an Erigon Archive Node](./2021-12-27-Run-Erigon-Archive-Node)
* [2021-11-11 Running an Ethereum Node](./2021-11-11-Running-an-Ethereum-node)

## Introduction
In this blog post the Erigon archive node, the Erigon RPC daemon, TrueBlocks' index scraper and
TrueBlocks' API are set up as Systemd services.

Erigon is a resource efficient Ethereum archive node that you can run on a standard local machine
if you have 2TB or more of storage. TrueBlocks is a software that indexes the historical
data of an archive node such that you can "list every appearance of an address
anywhere on the chain".

## Set up Erigon and Erigon's RPC as Systemd services
One way to set up Erigon and its RPC daemon as services on Linux is via Systemd. Create the files
`erigon.service` and `erigon-rpc.service` in `/etc/systemd/system/`. Set up respectively file as,

```bash
[Unit]
Description=Erigon Node
After=network.target network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=/media/node/erigon/erigon/
ExecStart=/bin/erigon --datadir=mainnet --private.api.addr=localhost:9090
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
and
```bash
[Unit]
Description=Erigon RPC Daemon
After=erigon.service

[Service]
WorkingDirectory=/media/node/erigon/erigon/
ExecStart=/bin/rpcdaemon --datadir=mainnet --txpool.api.addr=localhost:9090
    +++--private.api.addr=localhost:9090 --http.api=eth,erigon,web3,net,
    +++debug,trace,txpool --ws
User=magnus
Restart=always
RestartSec=5s

# Output to syslog
StandardOutput=syslog
StandardError=syslog
#Change this to find app logs in /var/log/syslog
SyslogIdentifier=erigon-rpc

[Install]
WantedBy=multi-user.target
```
Note that the "trace" option is enabled for the RPC daemon.

Now start the services with `sudo systemctl start erigon.service` and `sudo systemctl start
erigon-rpc.service`. The status of the services can be viewed with `sudo systemctl status erigon`
and `sudo systemctl status erigon-rpc`. The logs of the services will appear in `/var/log/syslog`.

There are more options that might be useful depending on how the services are to be run. E.g.,
depending on what network manager you are using the "After=" value might be different. This could
be important if you want to run `sudo systemctl enable <service>` such that the service is started
upon booting.

## Connect TrueBlocks to Erigon
In TrueBlocks' config file `~/.local/share/trueblocks/trueblocks.toml` the "rpcProvider" value
point to the RPC endpoint and it by default set to "http://localhost:8545". Thus, if you run a
local Erigon node you do not need to change this value.

To see if the connection between Erigon and TrueBlocks is correctly established type `chifra status
--terse` the output should be without errors and look something like,
```bash
TB_CONFIG_PATH: /home/magnus/.local/share/trueblocks/
TB_CACHE_PATH:  /media/node/trueblocks/cache/mainnet/
TB_INDEX_PATH:  /media/node/trueblocks/index/unchained/mainnet/
TB_CACHE_PATH: /media/node/trueblocks/cache/mainnet/
TB_INDEX_PATH: /media/node/trueblocks/index/unchained/mainnet/
2022/01/11 16:02:01 Client:       erigon/2021.11.3/linux-amd64/go1.17.5 (archive, tracing)
2022/01/11 16:02:01 TrueBlocks:   GHC-TrueBlocks//0.23.7-alpha-4b5fbdfe3-20220111 (eskey, no
pinkey)
2022/01/11 16:02:01 Config Path:  /home/magnus/.local/share/trueblocks/
2022/01/11 16:02:01 Cache Path:   /media/node/trueblocks/cache/
2022/01/11 16:02:01 Index Path:   /media/node/trueblocks/index/unchained/
2022/01/11 16:02:01 RPC Provider: http://localhost:8545
2022/01/11 16:02:01 Progress:     13984914, 13883211, 0, 13896584
```
In order for TrueBlocks to connect to the archive node both Erigon and its RPC daemon need to be
running. If only the RPC client is running errors such as,
`transport: Error while dialing dial tcp [::1]:9090: connect: connection refused` will appear.

## Creating the index
The index is created with the `chifra scrape` command that has a few options, most notably the
"sleep" and "block_cnt" flags which are set to 14 and 2000 by default. When creating the index from
scratch it might be a good idea to have a low sleep value and a high block count value. Later when the
index is up-to-date these values can be changed. To build the index,
```bash
chifra scrape --sleep 1 --block_cnt 20000 indexer
```
This requires that the connection between Erigon, Erigon's RPC daemon and TrueBlocks is well
established. If not error such as `This tool requires Parity traces. Add [requires]\nparity=false
to $CONFIG/blockScrape.toml to turn this restriction off. Quitting...` might appear, in which
TrueBlocks is not able to talk to the node correctly.

If the index will be used for historical tx analysis where an always updated index is not important
it is fine to run the above command in the terminal. However, if you want a continuously updated
index it is possible to create a service for this as well. First create a script that runs the
`chifra scrape` command,
```bash
#!/bin/bash
export PATH="$PATH:/media/node/trueblocks/trueblocks-core/bin"
chifra scrape --sleep 300 indexer
```
`--sleep` is set to `300` such that the script scrapes every 5 minutes. A block is created every 14
seconds on average and when the index is up to date it is unnecessary to run `scrape` continuously.
Make the script executable with `chmod +x chifra-scrape.sh`.
Note that Sysetemd services are not run in user login, meaning that if you have just added the
TrueBlocks path to your `.profile` or `.bashrc` `exit code 127` (command not found) will be thrown if
you don't export the path in the script file.

Then create a service running the script `/etc/systemd/system/trueblock-scrape.service`,
```bash
[Unit]
Description=TrueBlocks Scrape
After=erigon-rpc.service
Requires=erigon-rpc.service

[Service]
WorkingDirectory=/home/magnus/opt/bin/
ExecStart=/home/magnus/opt/bin/chifra-scrape.sh
User=magnus
Restart=always
RestartSec=5s

# Output to syslog
StandardOutput=syslog
StandardError=syslog
#Change this to find app logs in /var/log/syslog
SyslogIdentifier=trueblocks-chifra-scrape

[Install]
WantedBy=multi-user.target
```

## TrueBlocks' API
The `chifra` command can be used directly in the terminal and in bash scripts. However, there is
also a TrueBlocks [API](https://trueblocks.io/api/) that allows you to easily interact with TrueBlocks
trough making calls to `localhost`. `chifra serve` creates a REST JSON API for `chifra` commands
through a flame server. `chifra serve --help`,
```bash
Purpose:
  Serve the TrueBlocks API using the flame server.

Usage:
  chifra serve [flags]

Flags:
  -p, --port string   specify the server's port (default ":8080")
  -x, --fmt string    export format, one of [none|json|txt|csv|api]
  -v, --verbose       enable verbose (increase detail with --log_level)
  -h, --help          display this help screen

Notes:
  - To start API open terminal window and run chifra serve.
```
By running `chifra serve` in a terminal it is possible to call the API and query data, e.g.,
```bash
curl "http://localhost:8080/blocks?blocks=100"
```
To create a Systemd service running the API as a daemon create a script `chifra-serve.sh`,
```bash
#!/bin/bash
export PATH="$PATH:/media/node/trueblocks/trueblocks-core/bin"
chifra serve
```
Make the script executable `chmod +x chifra-serve.sh` and create a service,
```bash
sudo vim /etc/systemd/system/trueblock-serve.service
```
```bash
[Unit]
Description=TrueBlocks serve API
After=erigon-rpc.service
Requires=erigon-rpc.service

[Service]
WorkingDirectory=/home/magnus/opt/bin/
ExecStart=/home/magnus/opt/bin/chifra-serve.sh
User=magnus
Restart=always
RestartSec=5s

# Output to syslog
StandardOutput=syslog
StandardError=syslog
#Change this to find app logs in /var/log/syslog
SyslogIdentifier=trueblocks-chifra-serve

[Install]
WantedBy=multi-user.target
```

## Management of the services
All services can be stopped at any time an restarted in the same fashion as `ctrl + c` on the
command line. E.g., `chifra scrape` will contiue from the last bloom filter and Erigon will start
to sync from the last block.

Useful commands:
- If changes are made to the service files they can reloaded with `sudo systemctl daemon-reload`.
- Listen to the syslog file in real time with `sudo tail -f /var/log/syslog`.
- Check status of services with `systemctl status <name.service>`.
- `systemctl enable <name.service>` to start service on boot.
- `sudo journalctl -u <name.service>` to show logs of service.
