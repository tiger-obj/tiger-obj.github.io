---
layout:     post
title:      "Transactions"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-11-25 10:41:49
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - Transactions
---

## Transaction: Motivation and Definition

* Concurrent database access:

  * Attributed/Tuple/Table-level or Mylti-statement inconsistency
  * Concurrency Goal: Execute *sequence of SQL statements* so they appear to be running in *solation*
  * Enable concurrency for Multiprocessor, Myltithreads and Asynchronous I/O and so on.

* Resilience to system failures:
  * System-Failure Goal: Guarantee all-or-nothing execution, regardleee of failures

* Solution for both concurrency and failures: Transactions
  * **A transaction is a sequence of one or more SQL opreations treated as a unit**
  * Transactions appear to run in isolation
  * If the system fails, each transaction's changes are reflected either entirely or not at all.

* SQL Standard
  * Transaction begins automatically on first SQL statement
  * On "commit" transaction ends and new one begins
  * Current transaction ends on session termination
  * "Autocommit" turns each statement into transaction
  
## Transactions Properties

### ACID Properties

* Atomicity
  * crusshed happen during one transaction (before commit).
  * Each transaction is "all-or-nothing", never left half done.
  * Transaction Rollback (=Abort) undoes partial effects of transaction (however not all effects can be undone)
  * Can be system- or client-initialted
  * Never hold open a transaction and then wait for arbitratyamouts of time. (Lock won't be released)
* Consistency
  * Each client, each transation:
    1. Can assume all contrains hold when transaction begins
    2. Must guarantee all constrains hold when transaction ends
  * Recall: Integrity constraint is a specification of which database states are legal. (e.g. BC Normal Form.)
  * Serializability $\Rightarrow$ constraints always hold.
* Isolation
  * Serializability: Transactions may be interleaved, but execution must be **equivalent** to **some** sequential (serial) order of all transactions. Note that only serialization is guaranteed instead of global order.
  * I.e. there coulbe be interleaving between series of transactions from different client. However, each client see their series of transactions are executed in order.
  * Achieved by locking mechenisms.
* Durability
  * If system crashes ***after*** transaction commits, all effects of transaction remain in database.
  * Achieved by protocals based on Logging.

### Isolation Levels

