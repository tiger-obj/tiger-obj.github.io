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

* Solution for both concurrency and failures: ***Transactions***
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

* **A**tomicity
  * crusshed happen during one transaction (before commit).
  * Each transaction is "all-or-nothing", never left half done.
  * Transaction Rollback (=Abort) undoes partial effects of transaction (however not all effects can be undone)
  * Can be system- or client-initialted
  * Never hold open a transaction and then wait for arbitratyamouts of time. (Lock won't be released)
* **C**onsistency
  * Each client, each transation:
    1. Can assume all contrains hold when transaction begins
    2. Must guarantee all constrains hold when transaction ends
  * Recall: Integrity constraint is a specification of which database states are legal. (e.g. BC Normal Form.)
  * Serializability $\Rightarrow$ constraints always hold.
* **I**solation
  * Serializability: Transactions may be interleaved, but execution must be **equivalent** to **some** sequential (serial) order of all transactions. Note that only serialization is guaranteed instead of global order.
  * I.e. there coulbe be interleaving between series of transactions from different client. However, each client see their series of transactions are executed in order.
  * Achieved by locking mechenisms.
* **D**urability
  * If system crashes ***after*** transaction commits, all effects of transaction remain in database.
  * Achieved by protocals based on Logging.

## Isolation Levels

Serializability allows understandable behavior and consistency but it does have some **overhead** involved in the locking protocols that are used and it does **reduce concurrency**.

### Weaker "Isolation Levels"

**Isolation Levels** are 1. Per transaction 2. "In the eye of the baholder", doesn't affect other clients. (Each transactions's reads must conform to its isolation level)

>**Dirty Reads**: "Dirty" data item: written by an uncommiteed transaction (by a different transaction). Since before the transaction being commited therer could be failures happening and the effects of transaction will be rolled back. A dirty read could read values that may never appear in the database.

With higher Cucurrenccy and lower overhead under the price of lower consistency guarantees, we have following different levels of isolation:

* Read Uncommitted   **weakest**
  * A transaction may perform dirty reads
  * <code type="sql">Set Transaction Isolation Level Read Uncommitted;
    </code>

* Read Committed
  * A transaction may not perform dirty reads, *but still does not guarantee global serializability.*
  * Transaction executing using "read committed" is not atomic, any other transactions are atomic.
  * <code type="sql">Set Transaction Isolation Level Read Committed;
    </code>
* Repeatable Read
  * A transaction may not perform dirty reads && An item read multiple times cannot change value. *But still does not guarantee global serializability.*
  * Note: Insertions can be made by another transaction, even between two entire readings of a table. Those newly inserted tuples(withou locks) are called "phantom" tuples.
  * I.e. we add lock for first read, which means we may have phantom typles between two reads of the same relation in a repeatable read transaction (because newly inserted tuples are not locked), but we won't have tuples disappear from mthe relation in between two reads of it (because we locked those tuples).
  * <code type="sql">Set Transaction Isolation Level Repeatable Read;
    </code>
* (serializability) **strongest** defined as before.

### Read Only transactions
* helps system optimize performance
* Independent of isolation level
* <code type="sql">Set Transaction Read Only;
    </code>

### Summary
| |dirty reads | nonrepeatable reads | phantoms|
|---|---|---|---|
|Read Uncommitted|Y | Y|Y
|Read Committed| N| Y|Y
|Repeatable Read|N |N |Y
|Serializable|N |N |N
>Y: allowed, N: not allowed
