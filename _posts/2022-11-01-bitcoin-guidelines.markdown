---
title: "Bitcoin guidelines"
layout: post
date: 2022-11-01 23:14:28 +0700
image: /assets/images/blog/blockchain/2022-11-01/bitcoin-md.png
headerImage: false
tag:
- blockchain
category: english
author: Lake Nguyen
description: Bitcoin is the first and most important cryptocurrency in the Crypto world. In this article, I will introduce the most basic knowledge about Bitcoin and some initial instructions when using it.
---

## Contents
1. [Overview](#introduction)
2. [Install and test with Bitcoin](#experience)
3. [Some things to note when using Bitcoin](#note)
4. [Conclusion](#conclusion)

## Overview <a name="introduction"></a>

Bitcoin was first introduced in the paper of the same name by anonymous author Satoshi Nakamoto, you can see it in full [here][bitcoin-paper]. I will summarize the main contents as follows:

*Bitcoin operates on a peer-to-peer mechanism*. In the traditional monetary system, to make a transaction, we always need a third party such as a bank, the state or an organization that both parties trust to act as an intermediary to record the transaction history and update the balance for both parties. Meanwhile, Bitcoin can still make transactions without any other third party, and the two parties do not need to trust each other, this is called a peer-to-peer mechanism.

*Decentralized ledger*. All transaction history is stored in a ledger, in a centralized system only a third party is kept and has the right to add transactions to the ledger. In Bitcoin, each participant keeps a complete copy of the ledger, this ensures that no one can change the information in the ledger without the consensus of all parties.

*Consensus mechanism*. To add a transaction to a decentralized ledger, the transaction must be valid, that is, it cannot spend an amount larger than the balance, if this happens, other parties will refuse to record the transaction in their ledger and the transaction will fail. The problem here is that to know if a transaction is valid, we need to know all the transactions that happened before it, that is, the transactions must be in order. In a centralized system, we can ensure the order of transactions by using a queue, but in a decentralized system, where there are many parties who want to add transactions to the ledger, we need a consensus mechanism.

*Proof-of-Work*. Bitcoin uses a proof-of-work consensus mechanism, according to which all parties will solve a difficult problem, the input data of the problem is only available when the last transaction has been added to the ledger, this ensures that no one can start finding the solution to the problem before others. The party that finds the solution first will have the right to add transactions to the ledger.

*Blockchain:* In reality, in the Bitcoin network, there will be parties with stronger computing power than others, so they have a big advantage in finding the solution to the problem. Instead of adding transactions to the ledger themselves, others can send transactions to powerful computing nodes, which will aggregate the transactions into a block and add it to the ledger, which is now a chain of blocks (blockchain).

*Miners:* The nodes that add new blocks to the blockchain are called miners. Each time a new block is added, miners will receive a reward of a certain amount of bitcoins, they will also receive transaction fees from each transaction sent.

*Authentication mechanism:* To send and receive bitcoins, users need a pair of <address, private key>. To receive bitcoins, just give the address to the sender, but if you want to send bitcoins to someone else, users need to create a transaction with the recipient's address, the amount of bitcoins to be transferred, and a digital signature created using the private key to sign the transaction. When miners receive the transaction, they will check the transaction and the digital signature to authenticate the information before the transaction is added to the blockchain.

## Install and test with Bitcoin <a name="experience"></a>

That's the theory, now let's start practicing! There are many ways to use Bitcoin, but here I will use [Bitcoin Core][bitcoin-core], this is the software created by the father of Bitcoin, so it can be considered as the official one. Bitcoin core has all the functions to serve the Bitcoin network as well as each user, so it is very suitable for research and testing with Bitcoin.

You download and install it like other normal software, note to choose the correct operating system you are using. By default, when launching Bitcoin core, it will connect to other data nodes in the network and download the entire transaction history, this process will take about 150GB of hard drive and many days of waiting. Because it only serves the purpose of learning and research, I will change to test mode by adding the configuration `chain=regtest` to the `bitcoin.conf` file in the directory:

* On Windows: `%APPDATA%\Bitcoin\`
* On Macos: `$HOME/Library/Application Support/Bitcoin/`
* On Linux: `$HOME/.bitcoin/`

### Create wallet

After configuring, launch Bitcoin core and the interface will look like this:

![Create wallet](/assets/images/blog/blockchain/2022-11-01/create-wallet.png)

Create wallet
![Overview](/assets/images/blog/blockchain/2022-11-01/overview.png)

> The wallet is where address information and corresponding secret keys are stored. The wallet performs the task of creating, signing and authenticating transactions, synthesizing the balances of all addresses in the wallet, thereby making it easier for users to send and receive.

You open the *console* interface by going to Window and selecting Console:

![Console](/assets/images/blog/blockchain/2022-11-01/console.png)

On the console we can use some of the following commonly used commands:

* help: Display a list of all commands
* getbalance: Display the current wallet balance
* getnewaddress: Create a new address in the wallet. You can create many addresses in the wallet, usually each time you receive money, a new address will be created.
* listaddressgroupings: Display a list of addresses with money in the wallet

### Mine block

Now we will mine the block to get the reward. In `regtest` mode from block 101, each mined block will receive 50 BTC reward, on the console we use the following command:

```sh
generatetoaddress 101 bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an
```

> Note: `bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an` is the address I created with the command `getnewaddress`, you can replace it with the address you created.

After mining, there will be 50 BTC in the wallet (of course this is just a test BTC)

![Console](/assets/images/blog/blockchain/2022-11-01/balance.png)

> On the console, use the commands `getbalance` and `listaddressgroupings` to check the balance and address in the wallet.

### Send and receive money

First, you need to create a new wallet by going to File, selecting Create wallet... Next, go to the Receive tab to create a new address for receiving money:

![Receive](/assets/images/blog/blockchain/2022-11-01/receive.png)

Select Copy address then switch to the original account, select the Sent tab then enter the address in the Pay To box, the amount in Amount, select Transaction Fee as Custom then click `send` to send.

![Send](/assets/images/blog/blockchain/2022-11-01/send.png)

After Sending, you will see that the transaction has been created but not confirmed. Now we need to mine a new block containing this transaction, go to the console and type the following command:

```sh
generateblock bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an '["3204b38460db25d788577ef4b5051cb7d4284ff0bf56bcca084888fce6aa0adf"]'
```

> In which: `bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an` is the address to receive the reward when mining the block, `3204b38460db25d788577ef4b5051cb7d4284ff0bf56bcca084888fce6aa0adf` is the Transaction ID of the transaction.

After mining the block, we check again on the receiving wallet and see that the money has been transferred:

![Send success](/assets/images/blog/blockchain/2022-11-01/send_success.png)

## Some things to note when using Bitcoin <a name="note"></a>

### Regularly back up your wallet

When using Bitcoin, backing up your wallet is **extremely important**. If for some reason (damaged, lost hard drive) you lose your wallet data, you will **not** be able to transfer money out of your wallet, which means losing all the money in your wallet. You should back up your wallet after each transaction. The backup file should be saved on a trusted device, you can use a hardware wallet [Ledger][ledger].

To back up your wallet on Bitcoin core, go to File and select Backup Wallet...

### Do not reuse addresses

Because all Bitcoin transactions are public on the blockchain, to avoid being tracked, you should create a new address for each time you receive money (When sending money, the wallet will automatically create a new address and send the remaining amount to it).

### Bitcoin Mining

Bitcoin mining is the process by which miners compete to add new blocks to the blockchain. Miners try to find a number called a `nonce` that satisfies the problem by trying many different numbers until they find the `nonce`. The miner with the stronger computing power or the luckiest one who finds the `nonce` first gets to add the new block to the blockchain and receives a reward, you can see more about how Bitcoin works in the demo [this][demo-blockchain]. Currently, Bitcoin mining is done using specialized software and hardware such as ASIC machines, which allow for more efficient bitcoin mining.

The bitcoin algorithm automatically adjusts to increase the difficulty of the problem as the computing power of miners increases, ensuring that every 10 minutes a new block is added to the blockchain. The reward for each new block added by miners is halved every 210,000 blocks (about 4 years). Initially the reward was 50 BTC per block, now it is 12.5 BTC, it is expected that by 2140 all 21 million BTC will be mined.

### Bitcoin Wallet

Most Bitcoin users only need to use the functions of sending and receiving money, so they do not necessarily need to download the entire Bitcoin transaction history, instead they just need to use the wallet to store address information and secret keys. Currently, there are many types of wallets that can be used with Bitcoin, which can be divided into 2 types as follows:
* Hot wallet: is a type of wallet with an Internet connection including 2 types:
* Online wallet: address, secret key is managed by the wallet service provider, users need to register an account to use, for example wallets of exchanges: Binance, Coinbase...
* Offline wallet: is software installed on a computer or phone, address information and secret keys are also stored on the user's device, for example: Trust Wallet...

The advantage of a hot wallet is that it is simple, easy to use, low cost, but the disadvantage is that it is not very secure, if you use an Online wallet, there is also the risk of not being able to withdraw money if the provider locks your account.

* Cold wallet: This is a type of wallet that is not connected to the Internet, it can be a USB that stores information or users can even print out the wallet information on paper and store it in a safe place. The advantage of a cold wallet is good security because there is no Internet connection, but the disadvantage is that it is more difficult to use.

### Bitcoin Explorer

Bitcoin Explorer is a website that allows you to search for transaction information, check the balance of any address on the bitcoin network. Usually people will use a wallet in combination with Bitcoin Explorer instead of having to store the entire bitcoin history on the computer. For example: [btc.com](https://explorer.btc.com/btc), [blockchair](https://blockchair.com/bitcoin), [Blockcypher](https://live.blockcypher.com/)

## Conclusion <a name="conclusion"></a>

Through this article, I have introduced the most basic knowledge about Bitcoin and some initial instructions with it, hoping to help you more or less in the process of using it. See you again in the next articles.

[bitcoin-paper]: https://bitcoin.org/bitcoin.pdf
[bitcoin-core]: https://bitcoin.org/en/download
[ledger]: https://www.ledger.com/
[trust-wallet]: https://trustwallet.com/vi/bitcoin-wallet/
[demo-blockchain]: https://andersbrownworth.com/blockchain/
