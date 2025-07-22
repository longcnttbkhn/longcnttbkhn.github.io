---
title: AI powered copilot for crypto analyst
layout: post
date: 2024-11-25 10:00
image: /assets/images/posts/2024-11-25/intro.png
headerImage: false
published: false
tag:
- ethereum
- chatbot
category: english
author: Lake Nguyen
description: AI powered copilot for crypto analyst
---

Onchain data analysis is an activity that plays an extremely important role in the decision making of any crypto investor or trader. There are many data analysis products and tools on the market today to serve a variety of needs. These tools allow onchain data to be displayed in charts and dashboards so that users can easily read and research them. They can be divided into the following two main groups:

__Specialized analysis tools__: These are analysis tools designed for one or more specific purposes. Typical products in this group include:

- [DefiLlama](https://defillama.com/){:target="_blank"}: Providing comprehensive dashboards for DeFi
- [Nansen](https://www.nansen.ai/){:target="_blank"}:Smart Money Cash Flow Analysis Tool\\
- [RWA](https://www.rwa.xyz/){:target="_blank"}: Analytics Dashboard for Real World Assets\\
- [Chainplay](https://chainplay.gg/){:target="_blank"}: NFT analysis tool for GameFi\\
- [Arkham](https://intel.arkm.com/){:target="_blank"}: Behavioral analysis tool of onchain objects and entities.

The advantages of products in this group are simplicity and ease of use. However, the disadvantage is that each tool can only meet one or a few specific needs, so users need to use many tools at the same time and also need to understand many different products to be able to use them effectively.

__General analysis tools__: These are data analytics products that allow users to build their own custom analytics based on the platform's data using programming or query tools like SQL. Some notable products in this group include:

- [Dune Analytics](https://dune.com/home){:target="_blank"}: SQL Data Analytics Platform
- [Footprint Analytics](https://www.footprint.network){:target="_blank"}: Web3 Data Analytics Platform

The advantage of products in this group is to provide a general solution for users, which can solve any problem or need. However, the disadvantage is that it is difficult to use, requiring users to have programming skills, query data with SQL...

We are researching and developing a new product that can balance the pros and cons of both of these product groups. By combining a general data platform with a specially trained AI Chatbot, we have created:

- [Sonar](https://sonar.chainslake.io/){:target="_blank"} - an AI powered copilot for crypto analysis.


## Contents

1. [Overview](#overview)
2. [Simple questions](#simple_questions)
3. [Complex questions](#complex_questions)

## Overview <a name="overview"></a>

*Sonar* is a crypto Chatbot, used to answer questions about blockchain data analysis, with some form of data provided as shown below:

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/intro.png" alt="Sonar - Who are you?" width="550"></p>

*Sonar* can answer a number of questions from simple ones such as looking up and summarizing information on a data table to complex queries that require combining multiple tables. Here are some examples from simple to complex for your reference:

## Simple questions <a name="simple_questions"></a>

**1. Questions about price**

:man: *How much is btc?* \\
:man: *What is the price of eth?*\\
:man: *Price of eth in btc*\\
:man: *Top 10 coins with the highest percentage increase yesterday*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_1.png" alt="Query price" width="550"></p>

**2. Questions about trading volume**

:man: *What is the btc trading volume today?*\\
:man: *Top 10 coins with the largest total trading volume today*\\
:man: *Max price and total volume of bnb by hour in the last 48 hours*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_2.png" alt="Query volume" width="550"></p>

**3. Questions about balances**

:man: *link balance of wallet 0x8652fb672253607c0061677bdcafb77a324de081*\\
:man: *top 10 biggest holders of link*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_3.png" alt="Query volume" width="550"></p>   

**4. Questions about token transfer**

:man: *Top 10 wallets that received the most btc in the last 48 hours*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_4.png" alt="Query volume" width="550"></p>

**5. Questions about Centralized Exchange (CEX)**

:man: *List of 10 wallets of okx exchange*\\
:man: *List of 10 deposit wallets of binance exchange*\\
:man: *Top 10 exchanges with the most wallets*\\
:man: *Top 10 exchanges with the most deposit wallets*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_6.png" alt="Query volume" width="550"></p>

**6. Questions about Decentralized Exchange (DEX)**

:man: *Top 10 biggest btc trades on dex today*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_7.png" alt="Query volume" width="550"></p>

## Complex questions <a name="complex_questions"></a>

Complex questions are questions that require combining multiple pieces of data together. Because it has not been trained, *Sonar* will not be able to answer or will answer incorrectly. At this point, you will need to add definitions and hints to help *Sonar* understand what needs to be done. For example:

:man: *Top 10 wallets that bought the most btc on dex today.*

In the data that *Sonar* knows, there is no data that shows information about the wallet that purchased the token on the DEX, so for *Sonar* to answer this question, we will need to add the following definition and instructions:

:man: *Definition: Wallets that buy tokens on DEX are wallets that receive tokens in token purchases on DEX. Request: Get the top 10 wallets that bought the most btc on DEX today. Hint: Get the btc purchases on DEX today and join it with the BTC transfer table today according to transaction hash. Then group by the wallet that receives btc, then calculate the total btc received by each wallet and return the list of receiving wallets in descending order of btc*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_8.png" alt="Query volume" width="550"></p>

>You can continue asking about other tokens' DEX wallets in this conversation to take advantage of the established context.

Some other complex questions that I have trained *Sonar* to answer directly without further guidance:\\
:man: *Top 10 wallets selling the most btc on dex today*\\
:man: *Top 10 wallets with the most BTC deposits on binance today*\\
:man: *Top 10 wallets withdraw the most eth from coinbase today.*

<p style="
    text-align: center;
"><img src="/assets/images/posts/2024-11-25/query_9.png" alt="Query volume" width="550"></p>