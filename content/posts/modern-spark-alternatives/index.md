---
title: "Spark in 2025 - Should you go with it?"
draft: false
date: 2024-12-19T21:00:00.000Z
description: "Spark is getting old."
categories:
  - Data Engineering
tags:
  - Data Engineering
  - Spark
  - Polars
  - DuckDB
---

Apache Spark is one of the most used processing engines today in data
engineering. Much of that is thanks to Databricks, that have built their entire
product around Apache Spark. I've been using Apache Spark (mainly through
Databricks) for some years now and I like it very much, mainly because of the
flexibility that I can use Python and the entire universe of Python libraries
together with Apache Spark to cover the entire ETL/ELT process from start to
finish. Loading data from external systems, ad-hoc data analyses, or data
tranformations with dbt can all be handled with Apache Spark without having to
go through the pain of integrating separate compute engines that handle their
own litte piece of the pipeline. 
 
However, one thing that I have had suprisingly little use of in Apache Spark is
the fact that it is a distributed computing framework. The main idea and selling
point behind Apache Spark is that it can scale horizontally almost indefinitely,
by adding more and more compute nodes to the underlying cluster. This feature is
what enables Spark to process very large datasets by distributing workloads
across these nodes to avoid running out of memory for very large datasets. This
sounds great when saying it out loud, but it is actually quite far from the
truth. In fact, distributing your workload to many nodes may actually harm your
performance in many cases.

## Do everyone really need distributed computing?

When I started using Apache Spark I knew quite little about distributed systems
and their benefits and drawbacks, and while I am still far from an expert on the
subject (nor is it my objective to become one), I have over the years
encountered a few eye-opening insights related to how Apache Spark works under
the hood that heavily shifted my understanding of what it is really good for.
When I first started using spark, I thought that increasing the amount of nodes
for a cluster would always be beneficial for the type of work I was doing with
    Spark, which was standard data engineering and analytics work, meaning
    loading data, writing data, cleaning data and modeling data. Spark is after
    all an analytics engine, and horizontal scaling was then and is still a hot
    industry buzzword that is marketed as premium price/performance compared to
    vertical scaling. However, horizontal scaling and distributed computing is
    not a silver bullet for scalability and performance, and can even harm
    performance depending on your use case.

It was actually an
[article](https://docs.databricks.com/en/compute/cluster-config-best-practices.html)
on cluster configuration in the Databricks documentation that made me want to
look into this a bit deeper. In the article they list different use cases and
explain how to optimize your cluster to get the best performance out of your
cluster. The thing that cought my eye is that nowhere does Databricks suggest to
_increase_ the amount of nodes for a cluster, but rather they suggest to either
decrease the number of nodes or just use a _single node_ cluster if you are
doing _Data Analysis_, _training a machine learning model_ or running _complex
batch ETL_. The only use case listed that it does not say benefit from
decreasing number of workers or using a single node cluster is what they call
_Basic Batch ETL_, which means loading and writing data without performing more
complex operations such as joins or aggregations. This sounded weird to me at
first, because horizontal scaling is supposed to bring superior
price/performance.

One reason why horizontal scaling in Spark might be bad for performance, is
because of something called "data shuffling". Data shuffling occurs when a
cluster has partitioned datasets loaded into memory on the different nodes and
the user perform an operation that needs to be performed over the entire
datasets, for example a join operation where rows residing on different nodes
need to be combined into a single row, or calculating a sum over a column. This
results in that the cluster needs to send data between the nodes over the
network, and this is slow. If we were using a single node cluster, we can do the
entire join or aggregation without having to send data anywhere, since
everything is loaded into memory on our single node. I found this realization
quite eye-opening, as the tech industry and especially the Data domain, have
been so extremely pro distributed computing since the start of the "Big Data"
hype in the early 2010s.

## Alternatives to Apache Spark

Up until a few years ago, the general understanding of scalability in relation
to data platforms, was that if you want to make sure your data platform can
scale, you either need to use a MPP Database (Redshift, Snowflake) or a
distributed compute framework like Apache Spark. The alternatives were simply
too slow or inflexible. Regular relational databases were/are not flexible
enough and the most popular dataframe library, Pandas, would not scale. I
personally believe that if Pandas would have been performant enough, I think it
would totally dominate the data space today because of how easy it is for
anybody to just install it on their computer and learn it. Many people started
their data career with Pandas, myself included.

### Polars and DuckDB outclasses Apache Spark in benchmarks

After the release of Apache Arrow in 2016, more performant tools that do not use
distributed computing started emerging. On the Python-side, Polars emerged as a
dataframe library that is way more performant than Pandas. Around the same time,
DuckDB emerged on the SQL side. And both of them absolutely weep the floor with
Apache Spark when run on a single machine. According to (these
benchmarks)[https://duckdblabs.github.io/db-benchmark/] from September this year
(2024), both Polars and DuckDB are several times faster than Apache Spark for
both group by- and join-operations for all three used dataset sizes (500mb, 5gb
and 50gb). These benchmarks are run on a single machine with a 128 core
processor and 250gb RAM. For the 50Gb dataset, Polars and DuckDB were the only
ones to even complete the join-queries, as the rest of the tools either reported
Out-of-memory errors or other kinds of timeout errors.

There are a bunch of other benchmarks as well out there showing a similar
pattern, for example (this one)[https://docs.coiled.io/blog/tpch.html] or (this
one)[https://medium.com/walmartglobaltech/duckdb-vs-the-titans-spark-elasticsearch-mongodb-a-comparative-study-in-performance-and-cost-5366b27d5aaa]
out there as well that also shows quite poor performance for Apache Spark. The
latter also has some basic cost-calculations where Spark is 50 times as
expensive as running DuckDB.

Pretty impressive results. However, Spark does indeed shine in other areas such
a API-maturity and flexibility. In Spark you can choose between several
programming languages and the Spark SQL dialect is also very mature. In
comparison, DuckDB does not have any dataframe API at all and is entirely SQL
focused while Polars SQL support is not very mature. Both DuckDB and Polars have
both also
only recently reached version 1, so they are much less mature.

## So, should we continue using Spark or migrate to a new tool?

Well, if you are looking to improve your price/performance, migrating some of
your pipelines to Polars or DuckDB is not a bad idea, since you will be able to
process the same data with less cores and less memory. But there is of course a
migration cost in terms of development time and other factors that you need to
take into account when making the decision. Questions you can ask yourself are: Do you need both SQL and Dataframe
API? How many tools can your team support? Does your team have the skills and
motivations to work with these newer tools?

