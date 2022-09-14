---
layout: post
title: Ethereum gas fees
---

# Understanding Ex-Post Ethereum Gas Fees

## Resources
* [EIP-1559 Docs](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md)
* [Ethereum Docs](https://ethereum.org/en/developers/docs/gas/)

## Introduction
This blog post explains ex-post Ethereum gas fees transaction data, before and after the London Upgrade.

List of on-chain transaction variables used in this post,

Variable | Description
--- | ---
gas | gas provided (gas limit)
gasPrice | paid price per gas
maxFeePerGas | max gas for tx
maxPriorityFeePerGas | max tip to miner
gasCost | gasUsed * gasPrice
gasUsed | gas used
type | 0: legacy, 2: EIP-1559

These variables can be queried from a node.

## Legacy transactions
The transaction fees of legacy `TransactionType 0` txes are calculated as,
```
total fee = gasLimit * gasPrice
```
Example data from a legacy
[tx](https://etherscan.io/tx/0x0597736621572cbd87d2a1b3bddff10efd000586f2ac22311c69b32e61be606d):
```
"gas": 2562932,
"gasPrice": 31000000000,
"maxFeePerGas": 0,
"maxPriorityFeePerGas": 0,
"gasCost": 70459931000000000,
"gasUsed": 2272901,
"date": "2021-05-01 23:36:47 UTC",
"type": 0
```
The total gas fee for this transaction was,
```
gasCost = gasUsed * gasPrice = 70459931000000000
```
If look at the [block on Etherscan](https://etherscan.io/block/12351345) there is no `block base
fee per gas` for these legacy txes and thus the "new EIP-1559 formula" cannot be applied here.

## EIP-1559 transactions
The London Upgrade (2021-08-05) introduced `TransactionType 2` with new fee structure,
```
total fee = gasLimit * (baseFeePerGas + maxPriorityFeePerGas)
```
where `maxPriorityFeePerGas` is also called "tip".

Example data from an EIP-1559
[tx](https://etherscan.io/tx/0xa9d17f6f60e199774c24c2554e02fd0773f2ffb0b79218d853a1e64ae0d364cd):
```
"gas": 226041,
"gasPrice": 114574974786,
"maxFeePerGas": 122372813094,
"maxPriorityFeePerGas": 178,
"gasCost": 16660232508657474,
"gasUsed": 145409,
"date": "2022-01-24 12:44:59 UTC",
"type": 2
```
The total gas fee for this transaction was,
```
gasCost = gasUsed * gasPrice = 16660232508657474
```
However, we can also calculate the total fee according to the EIP-1559 formula. The tx was mined in
[block](https://etherscan.io/block/14068414) and the base fee for that block was `114574974608`.
```
total fee = gasUsed * (baseFeePerGas + tip) = 16660232508657474
```
where `tip = maxPriorityFeePerGas`.

Thus, we can see that `gasPrice = baseFeePerGas + maxPriorityFeePerGas` in this example. As we
will see later, it turns out that in EIP-1559 txes `gasPrice` is equal to the minimum of
`baseFeePerGas + maxPriorityFeePerGas` and `maxFeePerGas`.

## Legacy transactions after the London Upgrade
Let's investigate legacy transactions, `TransactionType 0`, that have been sent after the London
Upgrade. Looking at an example
[tx](https://etherscan.io/tx/0x63178f36b8299d722ebaee97a7d7c3028fed04c9184cd5a7fe968326010740d7),
```
"gas": 51051,
"gasPrice": 97900000000,
"maxFeePerGas": 0,
"maxPriorityFeePerGas": 0,
"gasCost": 4543539000000000,
"gasUsed": 46410,
"date": "2022-01-25 00:19:55 UTC",
"type": 0
```
One can see that the `maxFeePerGas` and `maxPriorityFeePerGas` is zero. The transaction fee is,
```
gasCost = gasUsed * gasPrice = 4543539000000000
```
The [block](https://etherscan.io/block/14071543)
base fee is `95659822298` and thus the EIP-1559 calculation would have yielded
`46410*(95659822298+0)` which is equal to `4439572352850180` which is lower than the actual paid
fee. Thus, this calculation cannot be used. Looking at the code below in the section
"The EIP-1559 code" we can see that actually the code sets
`gasPrice = maxFeePerGas = maxPriorityFeePerGas`.

## Same gasPrice, maxFeePerGas and maxPriorityFeePerGas
Let's look at another example
[tx](https://etherscan.io/tx/0x27c0fe1ffe0a4008ad31188670119289d4e1c5c807585d7215033f47583efa70),
```
"gas": 209977,
"gasPrice": 130549810158,
"maxFeePerGas": 130549810158,
"maxPriorityFeePerGas": 130549810158,
"gasCost": 17095889289620574,
"gasUsed": 130953,
"date": "2021-09-06 19:10:25 UTC",
"type": 2
```
Here we can see that `gasPrice`, `maxFeePerGas` and `maxPriorityFeePerGas` has the same value and
the tx is of `TransactionType 2`. The transaction fee is,
```
gasCost = gasUsed * gasPrice = 17095889289620574
```
The [block](https://etherscan.io/block/13173998) base fee is `121589027525` and if we use the
EIP-1559 calculation we get `130953*(121589027525+130549810158)` which is equal to
`33018337211101899` and thus larger than the fee actual paid in the transaction. This cannot hold
since `33018337211101899/gasUsed = 33018337211101899/130953 = 252138837683` is larger than
`maxFeePerGas` which is the maximum the transaction can pay.

In this situation the `maxFeePerGas` is equal to `130549810158` and is the maximal amount the tx
will pay per gas. Thus, `maxFeePerGas - baseFee`
will go to the miner. This means that the correct calculation is,
```
gasCost = gasUsed * (baseFee + maxFeePerGas - baseFeePerGas) = gasUsed * maxFeePerGas
```

This is an interesting example and points to the fact that,
```
# priority fee is capped because the base fee is filled first
priority_fee_per_gas = min(transaction.max_priority_fee_per_gas,
    transaction.max_fee_per_gas - block.base_fee_per_gas)
```
from the [EIP-1559.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md).

Here we can see that `gasPrice = maxFeePerGas = maxPriorityFeePerGas`.

## The EIP-1559 code
The gas fee for a transaction cannot exceed `maxFeePerGas`. The transaction will always pay the
`baseFeePerGas`, and pay the `maxPriorityFeePerGas`, as long as the combined amount of the two fees
doesn't exceed the transaction's `maxFeePerGas`. If it does the tx will pay the `maxFeePerGas` and
the miner will get `maxFeePerGas - baseFeePerGas`.

Here is the EIP-1559 code for calculating the gas fee for EIP-1559 transactions,
```
# execute transactions and do gas accounting
cumulative_transaction_gas_used = 0
for unnormalized_transaction in transactions:
	# Note: this validates transaction signature and chain ID which must happen before we normalize
	# below since normalized transactions don't include signature or chain ID
	signer_address = self.validate_and_recover_signer_address(unnormalized_transaction)
	transaction = self.normalize_transaction(unnormalized_transaction, signer_address)

	signer = self.account(signer_address)

	signer.balance -= transaction.amount
	assert signer.balance >= 0, 'invalid transaction: signer does not have enough ETH to cover attached value'
	# the signer must be able to afford the transaction
	assert signer.balance >= transaction.gas_limit * transaction.max_fee_per_gas

	# ensure that the user was willing to at least pay the base fee
	assert transaction.max_fee_per_gas >= block.base_fee_per_gas

	# Prevent impossibly large numbers
	assert transaction.max_fee_per_gas < 2**256
	# Prevent impossibly large numbers
	assert transaction.max_priority_fee_per_gas < 2**256
	# The total must be the larger of the two
	assert transaction.max_fee_per_gas >= transaction.max_priority_fee_per_gas

	# priority fee is capped because the base fee is filled first
	priority_fee_per_gas = min(transaction.max_priority_fee_per_gas, transaction.max_fee_per_gas - block.base_fee_per_gas)
	# signer pays both the priority fee and the base fee
	effective_gas_price = priority_fee_per_gas + block.base_fee_per_gas
	signer.balance -= transaction.gas_limit * effective_gas_price
	assert signer.balance >= 0, 'invalid transaction: signer does not have enough ETH to cover gas'
	gas_used = self.execute_transaction(transaction, effective_gas_price)
	gas_refund = transaction.gas_limit - gas_used
	cumulative_transaction_gas_used += gas_used
	# signer gets refunded for unused gas
	signer.balance += gas_refund * effective_gas_price
	# miner only receives the priority fee; note that the base fee is not given to anyone (it is burned)
	self.account(block.author).balance += gas_used * priority_fee_per_gas
```
Useful conditions in this part of the code to understand how transaction fees are calculated are,
```
# The total must be the larger of the two
assert transaction.max_fee_per_gas >= transaction.max_priority_fee_per_gas

# priority fee is capped because the base fee is filled first
priority_fee_per_gas = min(transaction.max_priority_fee_per_gas, transaction.max_fee_per_gas - block.base_fee_per_gas)

# signer pays both the priority fee and the base fee
effective_gas_price = priority_fee_per_gas + block.base_fee_per_gas
```
According to the conditions the effective price, i.e., `gasPrice`, is equal to the smallest of
`maxPriorityFeePerGas + baseFeePerGas` and `maxFeePerGas`.


Similarly, here is the EIP-1559 code for calulating the gas fee of legacy transactions,
```
def normalize_transaction(self, transaction: Transaction, signer_address: int) -> NormalizedTransaction:
	# legacy transactions
	if isinstance(transaction, TransactionLegacy):
		return NormalizedTransaction(
			signer_address = signer_address,
			signer_nonce = transaction.signer_nonce,
			gas_limit = transaction.gas_limit,
			max_priority_fee_per_gas = transaction.gas_price,
			max_fee_per_gas = transaction.gas_price,
			destination = transaction.destination,
			amount = transaction.amount,
			payload = transaction.payload,
			access_list = [],
			)
```

## Note that
- In none of the examples above is `maxFeePerGas = baseFeePerGas + maxPriorityFeePerGas` as you
sometimes read online.

- From the [EIP-1559.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md): "Previous
to this change, `GASPRICE` represented both the ETH paid by the signer per gas for a transaction as
well as the ETH received by the miner per gas. As of this change, `GASPRICE` now only represents the
amount of ETH paid by the signer per gas, and the amount a miner was paid for the transaction is no
longer accessible directly in the EVM."

- On Etherscan the transaction fee is defined as "Amount paid to the miner for processing the
transaction". However, this includes the base fee that was burned.
