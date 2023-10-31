---
title: "DWH 4: How to build the Data Analyst System on production using Databricks and AWS"
layout: post
date: 2023-10-21 21:00:00 +0700
image: /assets/images/blog/bigdata/2023-10-21/system_architecture.png
headerImage: false
tag:
- bigdata
category: blog
author: Long Nguyen
description: In previous posts, I introduced to you how to build the Data Analyst, BI and Data warehouse systems on premise. In this post, I will present step by step to deploy the systems on cloud (AWS) using Databricks, Trino and Superset.
---

In previous posts, I introduced how to build the Data Analyst, BI and Data warehouse systems on premise. These systems can be used for the development or testing phase, but stability and reliability in production are required. In this post, I will show to you a system design for production using Databricks, AWS and a few other technologies.

# Contents
1. [System architecture](#system-architecture)
2. [Create Data warehouse by using AWS Glue and S3](#create-storage)
3. [Run Spark job on Databricks](#run-spark-databricks)
4. [Install and config Trino on EC2](#add-node)
5. [Build tranform data pipeline by DBT](#dbt)
6. [Use Superset as the BI tool](#superset)

## System architecture <a name="system-architecture"></a>


<p align="center">
  <img src="/assets/images/blog/bigdata/2023-10-21/system_architecture.png" />
</p>

System is designed to include components sush as:
- *AWS S3*: Amazon Simple Storage Service (Amazon S3) is an object storage service offering industry-leading scalability, data availability, security, and performance, where our data are stored.  
- *AWS Glue*: is a serverless data integration service that makes it easier to discover, prepare, move, and integrate data from multiple sources for analytics, machine learning (ML), and application development. We will use it as a catalog storing metadata of the Data warehouse.
- *Databricks*: is a unified, open analytics platform for building, deploying, sharing, and maintaining enterprise-grade data, analytics, and AI solutions at scale. We will use it along with DBT to perform data transformation tasks on the data warehouse.
- *DBT*: is a SQL-first transformation workflow that lets teams quickly and collaboratively deploy analytics code following software engineering best practices like modularity, portability, CI/CD, and documentation. DBT will help us to build the data transform pipeline by using SQL and Python language.
- *Trino*: is a distributed SQL query engine designed to query large data sets distributed over one or more heterogeneous data sources. Trino produces the SQL service for BI tools or other applications can query data from Data warehouse by using SQL language.
- *Superset*: is an open-source modern data exploration and visualization platform. We will use Superset as a BI tools.

That's all we need to assemble this system, now it's time to get started.

## Create Data warehouse by using AWS Glue and S3 <a name="create-storage"></a>

First of all, we need a root account on AWS, if you don't have one, you can create new one [here](https://portal.aws.amazon.com/billing/signup#/start/email), we also need a credit card to pay some AWS fees.
1. Login to root account on AWS console, go to **Identity and Access Management (IAM)** service and create new one user with name *DataWarehouseAdmin* (you can choose any name you want but note that this will be your data warehouse administration account).
> Make sure you choose option: *Provide user access to the AWS Management Console*.
2. On *Set permissions* page let use *Attach policies directly* option, click **Create policy** to create a new policy for this user.
2. On *Create policy* page:
    1. Choose *Glue* service, check *All Glue actions* and *catalog*, *database*, *schema*, *table* resources.
    2. Click **Add more permissions**, choose *S3* service, check *All S3 actions* and *bucket*, *object* resources.
    3. Click **Next** to go to *Review and create* page
    4. In *Policy name* type *GlueAndS3Access* and click **Create policy**
3. Back to *Set permissions* page, refresh policies on *Permissions policies* section, search and check policy with name *GlueAndS3Access*.
4. Click **Next** and then **Create user** to complete user creation.
5. Login to *DataWarehouseAdmin* account, go to *S3* service and create a new bucket with name *lakehouse* (please choose your bucket name).
6. Create a new folder with name *datawarehouse* in your *lakehouse* bucket.



