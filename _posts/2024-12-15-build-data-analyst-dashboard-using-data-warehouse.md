---
title: "Build data analyst dashboard using Data Warehouse"
layout: post
date: 2024-12-15 10:00:00 +0700
image: /assets/images/blog/bigdata/2024-12-15/demo-dashboard.png
headerImage: false
published: false
tag:
- bigdata
- blockchain
category: blog
author: lake
description: This article will introduce how to use the Metabase tool to build analytical dashboards based on data from the Data Warehouse to serve BI activities
---

Continuing the series of articles on the Data Warehouse topic, today we will come to a quite interesting content which is building a Data Analysis Dashboard on the Data warehouse. Building a Data Analysis Dashboard is one of the most basic functions in BI (Business Intelligence) activities, helping to visualize data in the form of charts, thereby helping users to easily understand and see insights from the data. \\
You can use [Chainslake](https://metabase.chainslake.io){:target="_blank"}, a blockchain data warehouse that allows users to query blockchain data, build and share blockchain data analysis dashboards completely free of charge.

## Contents
1. [Overview of Metabase](#introduction)
2. [Install and configure Metabase to connect to Data warehouse](#install-and-config)
3. [Data tables in data warehouse](#tables)
4. [Query data, build and share dashboard](#query-build-share)
5. [Conclusion](#conclusion)

## Overview of Metabase <a name="introduction"></a>

[Metabase](https://www.metabase.com/) is a BI tool that allows querying data using SQL language on many different databases and SQL engines through [plugins](https://www.metabase.com/docs/latest/databases/connecting), presenting query results into tables, charts, data on analytical dashboards, you can see a demo dashboard of mine [here](https://metabase.chainslake.io/public/dashboard/ac9dbee4-af29-4ba8-b494-eae69f4ee835){:target="_blank"}.

Metabase has both an Opensource version with basic functions that can be installed on existing infrastructure and an Enterprise version with advanced functions that can be used directly on Metabase's cloud or self-hosted via a license key. You can see details [here](https://www.metabase.com/pricing/){:target="_blank"}.

## Install and configure Metabase to connect to Data warehouse <a name="install-and-config"></a>

I will install the Opensource version of Metabase via docker and connect to our DWH cluster via Trino (you can review my article on how to install Trino [here](/cai-dat-trino-truy-van-du-lieu-trong-data-warehouse))

First, I will create a database for Metabase in postgres on the [DWH](/how-to-build-data-warehouse-on-hadoop-cluster-part-1/#install_postgresql) cluster


```sql
postgres=# CREATE DATABASE metabase;
```

Install metabase via Docker

```sh
$ wget https://github.com/starburstdata/metabase-driver/releases/download/5.0.0/starburst-5.0.0.metabase-driver.jar
$ docker run -d -p 3000:3000 --name metabase \
  --network hadoop --add-host=node01:172.20.0.2 \
  -e "MB_DB_TYPE=postgres" \
  -e "MB_DB_DBNAME=metabase" \
  -e "MB_DB_PORT=5432" \
  -e "MB_DB_USER=postgres" \
  -e "MB_DB_PASS=password" \
  -e "MB_DB_HOST=node01" \
  -v starburst-5.0.0.metabase-driver.jar:/plugins/starburst-5.0.0.metabase-driver.jar \
   --name metabase metabase/metabase
```

> Note `172.20.0.2` is the container ip of node01 on my machine, you replace it with your machine ip.

Access `http://localhost:3000` you will see the Metabase startup interface, after creating an admin account you will be taken to the main working interface of Metabase.

![Metabase intro](/assets/images/blog/bigdata/2024-12-15/metabase-intro.png)

To connect to Trino, go to `Admin settings`, select the `Databases` tab, select `Add database` and set it up as shown below:

![Connect to Trino](/assets/images/blog/bigdata/2024-12-15/connect-trino.png)

After configuring, `Exit admin` and check if there is data in `Browser/Databases`. You can see Chainslake's data tables [here](https://metabase.chainslake.io/browse/databases/3-chainslake){:target="_blank"}

![Chainslake data](/assets/images/blog/bigdata/2024-12-15/chainslake-data.png)

## Data tables in the data warehouse <a name="tables"></a>

In this section, I will briefly describe the data tables in the data warehouse of [Chainslake](https://metabase.chainslake.io/browse/databases/3-chainslake){:target="_blank"} to have a basis for writing queries and building dashboards in the next section.

Chainslake's data warehouse includes many folders, each folder includes many tables with the same topic, specifically I will introduce some of the following tables and folders:

- __ethereum__: Folder containing Ethereum raw data (You can learn more about the data types of Ethereum blockchain in my previous article [here](/he-thong-phan-tich-du-lieu-blockchain-phan-2/))
    - __transactions__: Contains all transaction data on Ethereum (from the beginning to the present)
    - __logs__: Contains all log data (events) emitted from contracts when they execute transactions.
    - __traces__: Contains all Internal Transactions data (The smallest transaction units on Ethereum).
    - __address_type__: Contains all addresses that have ever appeared on Ethereum, including 2 types: `wallet` and `contract`
- __ethereum_decoded__: Directory containing decode data of some protocols on Ethereum. Currently, Chainslake has decode data of some popular protocols as follows:
    - __erc20_evt_transfer__: ERC20 token transfer data (all tokens from the past to present)
    - __erc721_evt_transfer__: ERC721 NFT transfer data
    - __erc1155_evt_transferbatch__, __erc1155_evt_transfer_single__: NFT 1155 transfer data
    - __uniswap_v2_evt_swap__, __uniswap_v3_evt_swap__: Token swap data on liquidity pools.
- __ethereum_contract__: Contains all contract information of some popular protocols
    - __erc20_tokens__: Information about ERC20 tokens.
    - __erc721_tokens__: Information about ERC721 NFTs.
    - __erc1155_tokens__: Information about ERC1155 NFTs.
    - __uniswap_v2_info__, __uniswap_v3_info__: Information about Swap pool contracts (Uniswap and other protocols with the same standard)
- __ethereum_dex__: Contains trading data of ERC20 tokens on DEX decentralized exchanges.
    - __token_trades__: Contains trading data of ERC20 tokens on DEX decentralized exchanges with WETH.
- __ethereum_prices__: Contains data on token prices according to transactions taken from DEX data.
    - __erc20_usd_day__, __erc20_usd_hour__, __erc20_usd_minute__: ERC20 token prices in usd by day, hour, minute.
- __ethereum_balances__: Directory containing information on balances, balance fluctuations of addresses on Ethereum
    - __erc20_native__: Summary table of final balances and daily balance fluctuations of all addresses of all tokens including Native tokens on Ethereum.
- __binance_cex__: Table containing transaction price data of coins on Binance exchange
    - __coin_token_address__: List of coins listed on Binance exchange that have corresponding tokens on Ethereum
    - __trade_minute__: Price data of trading pairs by minute on Binance.

## Query data, build and share dashboards <a name="query-build-share"></a>

Writing queries and building dashboards on Chainslake is quite simple if you already have SQL skills. I have created a dashboard in the collection [Demo](https://metabase.chainslake.io/collection/61-demo){:target="_blank"} so you can understand and easily start creating your own analytical dashboards.

![Demo collection](/assets/images/blog/bigdata/2024-12-15/demo-collection.png)

![Demo dashboard](/assets/images/blog/bigdata/2024-12-15/demo-dashboard.png)

Some notes when writing queries:
- The data tables all have an extremely large number of records (because they contain all historical data), so all tables are partitioned by `block_date`, you should use this column to filter or join between tables whenever possible to help your queries run faster and more efficiently.

- Using query optimization techniques, reducing the amount of data that needs to be scanned when executing queries always needs to be carefully considered to ensure that the query can be executed within the allowed time, and this also helps you improve your ability to write and optimize queries.
- Query results will be automatically cached for 1 day, you can change in the configuration of each query.

- Finally, don't forget to share your dashboards with everyone so that Chainslake can have more users.

## Conclusion <a name="conclusion"></a>

Above are my shares on how to build a data analysis dashboard on a data warehouse, I hope to receive support from all of you. See you again in the next articles.