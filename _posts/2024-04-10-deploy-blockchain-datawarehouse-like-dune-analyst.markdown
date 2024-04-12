---
title: "Deploy Blockchain Data Warehouse like Dune Analyst on AWS just from 400$ per month"
layout: post
date: 2024-04-10 09:00:00 +0700
image: /assets/images/blog/bigdata/2024-04-10/blockchain-dwh.png
headerImage: false
tag:
- bigdata
- blockchain
category: blog
author: Long Nguyen
description: From only 400$/month for infrastructure costs on AWS, I got a blockchain data warehouse like Dune Analyst. How I did it?
---

Before we get started, you need to note that this is just my suggestion based on the minimum needs for the system to work, it takes more things and of course more money to make it work on production, however it would be a good starting point. Please contact me on Telegram @longnh1 if you have any question.

## Content

1. [Questions & Answers](#q&a)
2. [Kiến trúc hệ thống](#system_architecture)
3. [Cài đặt và cấu hình](#install_and_config)
4. [Thử nghiệm](#test)
5. [Kết luận](#conclusion)


## Questions & Answers <a name="q&a"></a>

First question: *Why "like Dune Analyst"?*

If you don't know *Dune Analyst* you can read about it in [here](https://docs.dune.com/home), this is a great product if you are locking for a tool similar to Google but on Web3. There are so many analyst dashboards on blockchain protocols built by community on Dune. If you know SQL, you can write data queries on Dune's available tables to build your own dashboards. *Please learn carefully about Dune and how to use it before continuing to read this article*.

The reason I wanted to build a data warehouse like Dune is because Dune is an open data warehouse, which allows everyone to see how they have built the tables in the data warehouse, you can see in [Spellbook](https://github.com/duneanalytics/spellbook) project of Dune on Github. I can take advantage of what Dune has already built instead of doing everything from scratch.

Second question: *Why do I want to have my own data warehouse instead of using it directly on Dune?*

First of all, I can do everything with my data warehouse without any limit, I can add functions that I need but don't have on Dune, my applications can easily Accessing data in the warehouse, I can use the data for many different purposes such as building AI models, providing it to third parties... I can optimize to handle my own problems efficiently.

I can work with any blockchain EVM without being dependent on support from Dune. For example, new small chains do not yet have data on Dune.

I can join blockchain data with my private data without worrying about security issues on Dune.

