---
layout:     post
title:      "On-Line Analytical Processing (OLAP) and NoSQL Systems"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-12-24 09:50:11
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - OLAP
    - NoSQL
---

## On-Line Analytical Processing (OLAP)

Two broad types of database activity

* OLTP-Online Transaction Processing
  * Short transactions
  * Simple queries
  * Touch small portions of data
  * Frequent updates
  
* OLAP-Online Analytical Processing
  * Long transactions
  * Complex queries
  * Touch large portions of the data
  * Infrequent updates

* Data warehousing:
  * Bring data from operational (OLTP) sources into a single "warehouse" for (OLAP) analysis

* Decision support system (DSS)
  * Infrastructure for data analysis
  * E.g., data warehouse tuned for OLAP

### "Star Schema" - fact table references dimension tables

* Fact table: Updated frequently, often append-only, very large. Fact tables may be updated frequently, but an OLAP query session may work on a single snapshot so no updates are involved. 
  * Contains forein keys from other dimension tables, called dimension attributes.
  * Remaining attributes are refered to as dependent attributes.
* Demension tables: Updated infrequently, not as large

### OLAP queries

Typical procedure (although not explicitly) of OLAP queries:

* Join $\rightarrow$ Filter $\rightarrow$ Group $\rightarrow$ Aggregate

Performance

* Inherently very slow:
  * special indexes, query processing techniques
  * Extensive use of materialized views

Data Cube (a.k.a multidimensional OLAP)

* Dimension data forms axes of "cube"
* Fact(dependent) data in cells
* Aggregated data on sides(one dimension data aggregated), edges(two dimensional data aggregated), corner(all dimensional data aggregated).
* If dimension attributes not key, must aggretate
* Date (or even time) can be used to create key and work as a dimensional attribute

Drill-down and Roll-up

* Drill-down: Examining summary data, break out by dimension attribute (add more detals)
* Roll-up: summarize by dimension attribute (make more abstract summary)

SQL Constructs (only mySQL supports *With Rollup* at that time, written after group by)

* *With Cube*: Add to result: faces, edges, and corner of cube using NULL values
* *With Rollup*: For hierarchical dimensions, portion of With Cube. It won't produce all the combination for aggregation but depends on the order of attributes written in the group by clousure $\rightarrow$ fit for hierarchical data.

## NoSQL

### Motivation

* "SQl" = Traditional relational DBMS
* Recognition over past decade or so:
  * Not every data management/analysis problem is best solved exclusively using a traditional relaitional DBMS
* "NoSQL" = Not Only SQL(using traditional relational DBMS)

Traditional DBMS:

* Convenient
  * Simple data model
  * declarative query language
  * transaction guarantees
* Multi-user, Safe, Persistent, Reliable, Massive, Efficient

NoSQL Systems: Alternative to traditional relational DBMS

* \+ Flexible schema
* \+ Quicker/cheaper to set up
* \+ Massive scalability
* \+ Relaxed consistency $\rightarrow$ higher performance & availability 
* \- No declarative query language $\rightarrow$ more programming
* \- Relaxed consistency $\rightarrow$ fewer guarantees

### NoSQL: Overview

Several examples

* MapReduce framework $\sim$ OLAP
* Key-value stores $\sim$ OLTP
* Document stores
* Graph database systems

#### MapReduce Framework

Originally from Google, open source Hadoop

* No data model, data stored in files
  * GFS (Google File System)
  * HDFS (Hadoop Distributed File System)
* User provides specific functions
  * Map(), Reduce()
  * reader(), writer(), combine()
* System provides data processing "glue", fault-tolerance, scalability

Map and Reduce Functions

* Map: Divide problem into subproblems
  * map(item) $\rightarrow$ 0 or more {key,value} pairs
* Reduve: Do work on subproblems, conbine results
  * reduce(key, list-of-values) $\rightarrow$ 0 or more records

Since schemas and declarative queries are missed, more tools are introduced over MapReduce Framework:

* Hive - schemas, SQL-like query language
* Pig - more impereative but with relational operators
* *Both compile to "workflow" of Haddop (MapReduce) jobs* 

Dryad allows user to specify workflow

* Also DryadLINQ language

#### Key-Value Stores

Extremely simle infterface

* Data model: (key,value) pairs
* Operations: Insert(key,value), Fetch(key), Update(key), Delete(key)
* Some allow (non-uniform) columns within value
* Some allow Fetch on range of keys

Implementation: effciency, scalability, fault-tolerance

* Records distributed to nodes based on key
* Replication
* Single-record transactions, "eventual consisitency"

#### Document Stores

Like Key-Value Stores except value is document

* Data model: (key, document) pairs
* Document: JSON, XMOL, other semistructured formats
* Basic operations: Insert(key, document), Fetch(key), Update(key), Delete(key)
* Also Fetch based on document contents (system format secific)

#### Graph Database Systems

* Data model: nodes and edegs
* Nodes may have properties (including ID)
* Edges may have labels or roles (allowed to be non-uniform)
* Interfaces and query languages vary
* Single-step versus "path expressions" versus full recursion
* RDF "triple stores" can map to graph databases