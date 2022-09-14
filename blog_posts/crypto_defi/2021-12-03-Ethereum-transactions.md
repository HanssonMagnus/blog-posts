---
layout: post
title: Ethereum transactions
---

# Ethereum Transactions

## Resources
* [Ethereum: Data and analytics](https://ethereum.org/en/developers/docs/data-and-analytics/)
* [Etherscan](https://etherscan.io/)
* [What is crypto data?](https://towardsdatascience.com/what-is-crypto-data-fef70ecadf7b)
* [Decoding Ethereum smart contracts](https://towardsdatascience.com/decoding-ethereum-smart-contract-data-eed513a65f76)

## Introduction

## The Ethereum Blockchain Explorers
The data from decentralized exchanges (DEXes) are on the Ethereum blockchain. Through block
explorers we can more easily access the data. Most block explorers have REST APIs for action based
calls of blocks, transactions, miners, accounts and other on-chain activity. The blockchain
explorers have their own ways of reading, indexing and storing the blockchain data such that it is
easier to query. Block explorers are great for individual transaction analysis but may lack in
functionality if more specific questions are asked.

#### Etherscan
Etherscan is the most popular and widely used block explorer.

#### The Graph
The Graph is a decentralized indexing protocol for querying information from blockchains. The
protocol is used when building dApps so that the developers can rely on a decentralized query
infrastructure.

GraphQL is the query language used to query data organized in sub-graphs. Each query costs a fee
paid in the token GRT.

## Ethereum block
A block contains block headers, a list of ommer block headers and a series of transactions.

#### Structure of block headers
For completeness the structure of the block headers is added here
(from [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)).
Notably it shows that the hash of the receiptRoot is in the block header.

Value | Description
--- | ---
parentHash | Hash of the parent block's header, in its entirety
ommersHash | Hash of the ommers list portion of this block
benefitiary | The 160-bit address to which all fees collected from the successful mining of this block be transferred
stateRoot | Hash of the root node of the state trie, after all transactions are executed and finalisations applied
transactinosRoot | Hash of the root node of the trie structure populated with each transaction in the transactions list portion of the block
receiptRoot | Hash of the root node of the trie structure populated with the receipts of each transaction in the transaction portion of the block
logsBloom | The Bloom filters composed from indexable information (logger address and log topics) contained in each log entry from the receipt of each transaction in the transactions list
difficulty | A scalar value corresponding to the difficulty level of this block. This can be calculated from the previous block's difficulty level and the timestamp
number | A scalar value equal to the number of ancestor blocks. The genesis block has a number of zero
gasLimit | A scalar value equal to the current limit of gas expenditure per block
gasUsed | A scalar value equal to the total gas used in transactions in this block
timestamp | A scalar value equal to the reasonable output of Unix's time() at this block's inception
extraData | An arbitrary byte array containing data relevant to this block. This must be 32 bytes or fewer
mixHash | A 256-bit hash which, combined with the nonce, proves that a sufficient amount of computation has been carried out on thi block
nonce | A 64-bit value which, combined with the mixhash, proves that a sufficient amount of computation has been carried out on this block

The hashes are Keccak 256-bit hashes.

## Ethereum transactions

#### The route of an Ethereum transaction
- An external account holder posts a transaction including: target smart contract address, target
  function, arguments for the function, potential payment and gas fee.
- The transaction is posted to the mempool and passed through the network (or sent directly to a
  mining node).
- The transaction is mined and if successful the smart contract emits an event for each
  "transaction action" in the transaction. The event data is structured in "logs".
- The smart contract may in turn call other smart contracts and thus initialize "internal
  transactions" which are called "traces". The traces may emit additional log events.
- Once the block is mined and all transactions in the block is executed, the state is updated and
  distributed to the whole network.


#### Structure of a transaction

Value | Description
--- | ---
hash | String, 32 Bytes, hash of the transaction
type | Number, 0 or 2, type of tx (legacy or new after EIP-1559)
accessList | List, optional list of addresses and storage keys
blockHash | String, 32 Bytes, hash of the block where this transaction was in
blockNumber | Number, block number where this transaction was in
transactionIndex | Number, integer of the transactions index position in the block
confirmations | Number, number of blocks added since the block was mined
from | String, 20 Bytes, initiating EOA address
gasPrice | String, Gas price provided by the sender in wei
maxPriorityFeePerGas | Number, the maximum amount of gas to be included as a tip to the miner
maxFeePerGas | Number, the maximum amount of gas to be paid (inclusive of baseFeePerGas and maxPriorityFeePerGas)
gasLimit | Number, the maximum amount of gas units that can be consumed by the transaction
to | String, 20 Bytes, target smart contract address, null when its contract creation
value | String, value transferred in wei
nonce | Number, count of transactions sent from the account
data | Hexadecimal encoded representation of the target function with its arguments
r | part of the signature
s | part of the signature
v | part of the signature
creates | (Shown in ethers.js but not in web3.js)
chainId | Number, which chain (e.g., 1 is mainnet)
wait | (Shown in ethers.js but not in web3.js)

Where the signature of the transaction consists of r, s and v and can be used to get the public key
of any Ethereum account. Thus, if and only if a transaction has been sent from an account you can
derive the public key. r and s are outputs of an Elliptic Curve Digital Signature Algorithm (ECDSA)
signature and v is the recovery id.

[The London hard fork](https://ethereum.org/en/history/#london) introduced
[EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md) which announced a new
"[EIP-2718](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2718.md) transaction type"
called "2" (the legacy transaction type is called "0").

[EIP-2930](https://eips.ethereum.org/EIPS/eip-2930)
added a transaction type which contains an access list. This is a list of addresses and
storage keys that the transaction plans to access. Accesses outside the list are more expensive.
The gas cost of accessing addresses and storages keys within the list is cheaper than accessing
outside the list. The rationale is to make transactions more predictable so that clients can pre
load data sets from databases etc.

chainId was introduced in [EIP-155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)
to prevent replay attacks. The list of chain IDs are specified in EIP-155 as,

CHAIN_ID | Chain(s)
--- | ---
1 | Ethereum mainnet
2 | Morden (disused), Expanse mainnet
3 | Ropsten
4 | Rinkeby
5 | Goerli
42 | Kovan
1337 | Geth private chains (default)

More chain IDs can be found at [chainlist.org](https://chainlist.org/). From my understanding the
EIP-155 went live in the Berlin hard fork and transactions before that does not have any chainId
variable. However sometimes old transactions before the fork still report a chainId number and then
it is often "0".

## Ethereum Receipts
The Keccak 256-bit hash of the root node of the trie structure populated with the receipts of each

#### Structure of a receipt
Table from `returns` of
[web3.eth.getTransactionReceipt(hash [, callback])](https://web3js.readthedocs.io/en/v1.2.9/web3-eth.html?highlight=receipt#gettransactionreceipt):

Value | Description
--- | ---
to | String, Address of the receiver. null when it’s a contract creation transaction
from | String, Address of the sender
contractAddress | String, The contract address created, if the transaction was a contract creation, otherwise null
transactionIndex | Number, Integer of the transactions index position in the block
gasUsed | Number, The amount of gas used by this specific transaction alone
logsBloom |
blockHash | 32 Bytes - String, Hash of the block where this transaction was in
transactionHash | 32 Bytes - String, Hash of the transaction
logs | Array, Array of log objects, which this transaction generated
blockNumber | Number, Block number where this transaction was in
confirmations |
cumulativeGasUsed | Number, The total amount of gas used when this transaction was executed in the block
effectiveGasPrice |
status | boolean, TRUE if the transaction was successful, FALSE if the EVM reverted the transaction
type | (Shown in ethers.js but not in web3.js)
byzantium | (Shown in ethers.js but not in web3.js)

#### Structure of logs

Value | Description
--- | ---
transactionIndex |
blockNumber |
transactionHash |
address | Address to contract emitting the event
topics | Array, of hashes of event function and arguments
data | Input arguments to the event function
logIndex |
blockHash |

#### Logs
Logs are part of the transaction receipt and show outcomes of transactions. Logs are results of the
Ethereum opcode being executed on the EVM. Thus, the logs does not need to be stored since they can
be recreated, however they are stored (in nodes) alongside the blockchain but not part of it. The logs can
though be verified by the blockchain since the hash of them are part of the block header.

#### Events
"Events" are events taking place in Ethereum smart contracts when transactions interact with
them. The log information from a block
has overview meta data about the block and the transactions in it. Events contain additional
logging information beyond that of the block information. Hashed Events are present in `topics` in
the block data and in the `data` field the transaction data is present.

The `emit` keyword in the Solidity language was introduced after the Dao hack in
[version 0.4.21](https://github.com/ethereum/solidity/releases/tag/v0.4.21)
to make events stand out with regards to regular function calls `emit EventName()` should be used
to call events. Here is [issue 2877](https://github.com/ethereum/solidity/issues/2877) were the
change was discussed.

```js
pragma solidity ^0.4.21;
contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );
function deposit(bytes32 _id) public payable {
        // Events are emitted using `emit`, followed by
        // the name of the event and the arguments
        // (if any) in parentheses. Any such invocation
        // (even deeply nested) can be detected from
        // the JavaScript API by filtering for `Deposit`.
        emit Deposit(msg.sender, _id, msg.value);
    }
}
```

#### Topics
Topic0 is the hashed signature of the event interface definition. The others topics are the
function parameters of the event function, these are typically addresses that are involved in the
event.

#### Data
Data in the log is input parameter values to the event function.

## Example transaction
At this example we will look at an arbitrage
[transaction](https://etherscan.io/tx/0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4)
that uses a flash loan to profit from differences in the exchange rage between wETH and DAI on
Uniswap and Sushiswap.

In order to analyze the transaction we will use Ethers.js and connect to an Infura node.

#### Fetch the transaction
```js
/*
This script queries an arbitrage transaction that uses Aave Protocol V2, Uniswap V2 and Sushiswap.

Test transaction:
    0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4
Etherscan:
    https://etherscan.io/tx/0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4
Resources:
    https://docs.ethers.io/v5/
*/

// Comment in console
console.log("Starting!");

// Import libraries
import { ethers } from "ethers";

// Set provider (Ethereum node)
const provider = new ethers.providers.JsonRpcProvider("urlOrConnectionInfo");

// Set transaction
const tx_hash = '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4';

// Define function to retrive tx
async function getTx() {
    const tx = await provider.getTransaction(tx_hash) //get tx from node
    const receipt = await provider.getTransactionReceipt(tx_hash) //get tx from node
    var fs = require("fs");
    var jsonTx = JSON.stringify(tx); //convert object to string
    var jsonReceipt = JSON.stringify(receipt); //convert object to string
    fs.writeFileSync ('./data/tx.json', jsonTx);
    fs.writeFileSync ('./data/txReceipt.json', jsonReceipt);
    console.log('Complete!')
};

// Export main function
export const run = () => {
  getTx().catch(err => console.error(err));
};

```

#### Raw data of the transaction
The raw data from the transaction looks as:
```js
{
  hash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
  type: 0,
  accessList: null,
  blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737',
  blockNumber: 11912416,
  transactionIndex: 0,
  confirmations: 1851433,
  from: '0x987069876A0073a080F9C1F5Bc3339Bcf6CEb0d3',
  gasPrice: { type: 'BigNumber', hex: '0x00' },
  gasLimit: { type: 'BigNumber', hex: '0x0b6cc6' },
  to: '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
  value: { type: 'BigNumber', hex: '0x00' },
  nonce: 2,
  data: '0x0c5c9fa8000000000000000000000000c02aaa39b223fe8d0a0e5c4f27ead9083c756cc200000000000000000000000000000000000000000000003635c9adc5dea00000000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000002e00000000000000000000000000000000000000000000000022e92f1cfbaaacd5d000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000002000000000000000000000000a478c2975ab1ea89e8196811f51a7b7ade33eb11000000000000000000000000c3d03e4f041fd4cd388c549ee2a29a9e5075882f00000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000a4022c0d9f0000000000000000000000000000000000000000000111fe488c4c806c90bc510000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c3d03e4f041fd4cd388c549ee2a29a9e5075882f000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a4022c0d9f0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000038aeea6492a28d1d130000000000000000000000004c74cb8ea548d81f03eb542c916dfd3187bc0dd90000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  r: '0x508bf2a09dc198806f599333432a24cf9f6ded1d0f31bea56dbaf8b97dbe2234',
  s: '0x59f4ed7f05b4bd64851a9a1c6a1a368d1d884989732a7477a6bb69be89fe72fe',
  v: 27,
  creates: null,
  chainId: 0
}
```
In the output, the leading `0x` in a string is an indicator that the string is a hexadecimal
representation. One byte is represented with 2 HEX characters thus a 32 byte string is represented with 64
HEX characters plus the leading 0x yielding a string of length 66.

"Data" stores a hexadecimal encoded hash representation of the target function and arguments for that
function. The first 4 bytes after `0x` (in this case `0c5c9fa8`) represents the hashed signature of the
function name being called and its input parameters (these are not always unique which can lead to
[problems](https://rattibha.com/thread/1425217046636371969?lang=en)).
The rest of the bytes are function arguments passed to the function. The
length of the input field can thus vary depending on the function called and its arguments. Every
subsequent 32 bytes (64 characters) after the function is a different input variable.

Note that the structure of transactions have changed slightly through out history. A recent
(2021-12-08) transaction have the following structure,

```js
{
      hash: '0x19cb27d42fd1f4711c51312173aae97af8a864fe53e685881a8be060503cc886',
      type: 2,
      accessList: [],
      blockHash: '0x329db63fd12c33c9b6804150281f1a84f550f4809b7e5c97e07e0953d631dc3d',
      blockNumber: 13765869,
      transactionIndex: 97,
      confirmations: 1,
      from: '0xcE5EB2F0Fa280Be625f0427A66652060658Ac583',
      gasPrice: [BigNumber],
      maxPriorityFeePerGas: [BigNumber],
      maxFeePerGas: [BigNumber],
      gasLimit: [BigNumber],
      to: '0x2A88696e0fFA76bAA1338F2C74497cC013495922',
      value: [BigNumber],
      nonce: 55,
      data: '0x0f6795f20000000000000000000000007d1afa7b718fb893db30a3abc0cfc608aacfebb0',
      r: '0x8ccfd1a1a32dfde7a97064df5e402fd0930d5838bc75adce82ec34f7dcbac55b',
      s: '0x174fe1c7fff0dd793e71f93032e678c2391537d7827c7cc5c45cdf14be00fdf1',
      v: 1,
      creates: null,
      chainId: 1,
      wait: [Function (anonymous)]
    }
```
Where `maxPriorityFeePerGas` and `maxFeePerGas` are new variables added in EIP-1559.

#### Decode/parse hexadecimals with ABI
To decode the input field we need to use the application binary interface (ABI) of the smart
contract. The ABI is a JSON object containing all the definitions of functions and events for a
given smart contract. With the ABI we can match the hashed signatures to readable definitions.

#### Raw data of the receipt
```js
{
  to: '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
  from: '0x987069876A0073a080F9C1F5Bc3339Bcf6CEb0d3',
  contractAddress: null,
  transactionIndex: 0,
  gasUsed: { type: 'BigNumber', hex: '0x056fbc' },
  logsBloom: '0x00200000400000000000001080010000000200000000000080200000000001000000000000001000000000020000000002000000084000040001000000200000000000000200800000000208000000a000000000004008000000000000000000102000000200800000000000000008000000000000080400000000100000000000000200008002200000004000000000000000000000000800000040000000000200800000000000040000000000000000000000000000000002000000000100001000420004000000000021004a0000000002000000001000002002000020000010200000000000000000040000000000000000080800000000000000000000',
  blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737',
  transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
  logs: [
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000000003635c9adc5dea00000',
      logIndex: 0,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000000003635c9adc5dea00000',
      logIndex: 1,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0x6B175474E89094C44Da98b954EedeAC495271d0F',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000000111fe488c4c806c90bc51',
      logIndex: 2,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xA478c2975Ab1Ea89e8196811F51A7B7Ade33eB11',
      topics: [Array],
      data: '0x000000000000000000000000000000000000000000302fb5984ce4f5106869160000000000000000000000000000000000000000000009b7884991b45ad4b263',
      logIndex: 3,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xA478c2975Ab1Ea89e8196811F51A7B7Ade33eB11',
      topics: [Array],
      data: '0x000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003635c9adc5dea000000000000000000000000000000000000000000000000111fe488c4c806c90bc510000000000000000000000000000000000000000000000000000000000000000',
      logIndex: 4,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x000000000000000000000000000000000000000000000038aeea6492a28d1d13',
      logIndex: 5,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC3D03e4F041Fd4cD388c549Ee2A29a9E5075882f',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000005f605fc2366901f6346a940000000000000000000000000000000000000000000013919028bf298a3851c1',
      logIndex: 6,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC3D03e4F041Fd4cD388c549Ee2A29a9E5075882f',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000000111fe488c4c806c90bc5100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000038aeea6492a28d1d13',
      logIndex: 7,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000000000022e92f1cfbaaacd5d',
      logIndex: 8,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000000003642471f01287a0000',
      logIndex: 9,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0x030bA81f1c18d280636F32af80b9AAd02Cf0854e',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000000000000003bc25883e3d13',
      logIndex: 10,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0x030bA81f1c18d280636F32af80b9AAd02Cf0854e',
      topics: [Array],
      data: '0x0000000000000000000000000000000000000000000000000003bc25883e3d1300000000000000000000000000000000000000000340b53c22be6576d65de1e6',
      logIndex: 11,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000001af9af0c7245e2e287b660000000000000000000000000000000000000000002951b3c299c43590c0b7fd0000000000000000000000000000000000000000000d33f6cd17cd16f566f99600000000000000000000000000000000000000000340b6193ac37b114f9c9ef40000000000000000000000000000000000000000033e5ee9b78c57a856754a73',
      logIndex: 12,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000000003642471f01287a0000',
      logIndex: 13,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    },
    {
      transactionIndex: 0,
      blockNumber: 11912416,
      transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
      address: '0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9',
      topics: [Array],
      data: '0x00000000000000000000000000000000000000000000003635c9adc5dea000000000000000000000000000000000000000000000000000000c7d713b49da00000000000000000000000000000000000000000000000000000000000000000000',
      logIndex: 14,
      blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
    }
  ],
  blockNumber: 11912416,
  confirmations: 1851433,
  cumulativeGasUsed: { type: 'BigNumber', hex: '0x056fbc' },
  effectiveGasPrice: { type: 'BigNumber', hex: '0x00' },
  status: 1,
  type: 0,
  byzantium: true
}
```

#### Events in the logs
There are 15 events in the logs related to the transaction. We can view them on
[Etherscan](https://etherscan.io/tx/0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4#eventlog)
to first get an overview,

Event no. | Contract | Event
--- | --- | ---
0 | Wrapped Ether | Transfer()
1 | Wrapped Ether | Transfer()
2 | Dai Stablecoin | Transfer()
3 | Uniswap V2: DAI 2 | Sync()
4 | Uniswap V2: DAI 2 | Swap()
5 | Wrapped Ether | Transfer()
6 | SushiSwap: DAI | Sync()
7 | SushiSwap: DAI | Swap()
8 | Wrapped Ether | Withdrawal()
9 | Wrapped Ether | Approval()
10 | Aave: aWETH Token V2 | Transfer()
11 | Aave: aWETH Token V2 | Mint()
12 | Aave: Lending Pool V2 | ReserveDataUpdated()
13 | Wrapped Ether | Transfer()
14 | Aave: Lending Pool V2 | FlashLoan()

#### Raw data of log with index 14
Looking at the content of the last log 14,
```js
{
    transactionIndex: 0,
    blockNumber: 11912416,
    transactionHash: '0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4',
    address: '0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9',
    topics: [
      '0x631042c832b07452973831137f2d73e395028b44b250dedc5abb0ee766e168ac',
      '0x0000000000000000000000004c74cb8ea548d81f03eb542c916dfd3187bc0dd9',
      '0x0000000000000000000000004c74cb8ea548d81f03eb542c916dfd3187bc0dd9',
      '0x000000000000000000000000c02aaa39b223fe8d0a0e5c4f27ead9083c756cc2'
    ],
    data: '0x00000000000000000000000000000000000000000000003635c9adc5dea000000000000000000000000000000000000000000000000000000c7d713b49da00000000000000000000000000000000000000000000000000000000000000000000',
    logIndex: 14,
    blockHash: '0x6d1d8405443725715b19856aa11ff8c2c71615a8e458564f590acc3a5b5ec737'
  }
```
Looking at "address" `0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9` is the contract address to the
Aave: Lending Pool V2.
Topic 0 it the FlashLoan() Event with hash
`0x631042c832b07452973831137f2d73e395028b44b250dedc5abb0ee766e168ac`. Topic 1 and 2 are the
"to address" from the transaction. Topic 4 is the contract address of the contract "Wrapped Ether".
From the ABI we can see that the FlashLoan function has the following input parameters,
```js
FlashLoan
(index_topic_1 address target, index_topic_2 address initiator, index_topic_3 address asset,
uint256 amount, uint256 premium, uint16 referralCoe)
```
where the topics are specified as addresses for "target", "initiator" and "asset".

The data field of log 14 contains the numerical input parameters of the event function,
```js
amount : 1000000000000000000000
premium : 900000000000000000
referralCode : 0
```
written in HEX. For example is 900000000000000000 equal to
[C7D713B49DA0000](https://www.rapidtables.com/convert/number/decimal-to-hex.html) in HEX.
Thus the first 64 characters (32 bytes) are
'00000000000000000000000000000000000000000000003635c9adc5dea00000' and thus equal to
1000000000000000000000 wei which is equal to 1000 ETH. The premium is 0.9 ETH which is equal to
the 0.09% fee that the Aave protocol takes.

#### Decode log 14 using the Aave ABI
To decode the event we use the ABI of the contract in the address field of the log. The signature
is a Keccak 256-bit hash of the event name and input argument types (argument names ignored). For
example an event `Hello(uint256 worldId)` the signature will be `keccak('Hello(uint256)').
```js
// Comment in console
console.log("Starting!");

// Import libraries
import { ethers } from "ethers";

// Import ABI for Aave Lending Pool V2
const LendingPoolV2Artifact = require('@aave/protocol-v2/artifacts/contracts/protocol/lendingpool/LendingPool.sol/LendingPool.json');

// Import tx.json and txReceipt.json
var fs = require('fs');
let txRaw = fs.readFileSync('./data/tx.json');
let tx = JSON.parse(txRaw);

let txReceiptRaw = fs.readFileSync('./data/txReceipt.json');
let txReceipt = JSON.parse(txReceiptRaw);

// Define log 14 (flash loan event)
const log14 = txReceipt.logs[14]

// Construct Interface of ABI
const iface = new ethers.utils.Interface(LendingPoolV2Artifact.abi)

// Decode log function
async function decodeLog(log) {
    const topics = log.topics //get topics from log
    const eventName = iface.getEvent(topics[0]).name //get name of event
    const data = log.data //get input parameters
    const decodedEvent = iface.decodeEventLog(eventName, data, topics)
    return {
        event: eventName,
        inputParams: decodedEvent
        };
};

// Define function to decode tx
async function getTx() {
    console.log(decodeLog(log14))
};

// Export main function
export const run = () => {
  getTx().catch(err => console.error(err));
};

```

Output:
```js
Starting!
Promise {
  {
    event: 'FlashLoan',
    inputParams: [
      '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
      '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
      '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      [BigNumber],
      [BigNumber],
      0,
      target: '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
      initiator: '0x4C74Cb8Ea548d81f03Eb542C916DfD3187bC0Dd9',
      asset: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2',
      amount: [BigNumber],
      premium: [BigNumber],
      referralCode: 0
    ]
  }
}
```
Here we can see that the same information is displayed: The flash loan event together with the
input parameters.
