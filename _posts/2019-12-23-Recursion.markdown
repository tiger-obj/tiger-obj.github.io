---
layout:     post
title:      "Recursion in SQL"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-12-23 08:30:39
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - Recursion in SQL
---

## Recursion in SQL

### Basic recursive WITH statement

SQL is not a "Turing Complete" language, instead

* Simple, convenient, declarative
* Expressive enough for most database queries
* But basic SQL can't express unbounded computations
  * e.g. find all of someone's ancestors from relation called ParentOf(parent,child)
  * e.g. find total salary cost of project 'X' from relations Employee(ID,salary), Manager(mID,eID),Project(name,mgrID).
  * Find cheapest way to fly from 'A' to 'B' from relation Flight(orig,dest,airline,cost).

* SQL *WITH* Statement

```sql
With R1(A1,A2,...An) As (query-1),
     R2 As (query-2),
     ...
     Rn As (query-n)
<query involving R1,...,Rn (and other tables)>
```

* SQL *WITH Recursive* Statement

```sql
With Recursive
     R As (base query (not R)
            Union
            recursive query)
<query involving R (and other tables)>
```
where union will eliminate duplicates in its result, so that recursion could finally terminate (query involving in last statement reach "fix-point").

### Examples

* Find Anscestors for Mary from ParentOf(parent,child):

```sql
with recursive
    Ancestor(a,d) as (select parent as a, child as d from ParenOf
                      union
                      select Ancestor.a,ParentOf.child as d
                      from Ancestor, ParentOf
                      where Ancestor.d=ParentOf.parent)
select a from Ancestor where d = 'Mary'
```

* Find total salary cost of project 'X' from relations Employee(ID,salary), Manager(mID,eID),Project(name,mgrID)

```sql
with recursive
    Superior as (select * from Manager
                 union
                 select S.mID, M.eID
                 from Superior S, Manager M
                 where S.eID=M.mID)
select sum(salary)
from Employee
where ID in
    (select mgrID from Project where name='X'
    union select eID from Project, Superior
    where Project.name = 'X' AND Project.mgrID = Superior.mID)
```

Or more efficiently:

```sql
with recursive 
Xemps(ID) as (select mgrID as ID from Project where name='X'
              union
              select eID as ID
              from Manager M, Xemps X
              where M.mID = X.ID)
select sum(salary)
from Employee
where ID in (select ID from Xemps)
```
Note here *union* gaurantees that ID in Xemps will always be distinct.

* Find cheapest way to fly from 'A' to 'B' from relation Flight(orig,dest,airline,cost) without considering how many transfers to be taken.

```sql
with recursive
    Route(orig, dest, total, length) as
        (select orig, dest, cost as total, 1 from Flight
         union
         select R.orig, F.dest, cost+total as total, R.length+1 as length
         from Route R, Flight F
         where R.length<10 and R.dest = F.orig)
select * from Route
where orig='A' and dest='B'
```

> Note: add length to limit number of routs possibly containing loops. However, limit number is hard to estimate.

or alternatively:

```sql
with recursive
    FromA(dest, total) as
        (select dest, cost as total from Flight where orig='A'
         union
         select F.dest, cost+total as total
         from FromA FA, Flight F
         where FA.dest = F.orig)
select * from FromA where dest='B'
```

### Nonlinear Recursion

> Linear recursion refers to those recursions with only one reference to R.

Nonlinear vs. Linear

* \+ Query looks clearner
* \+  Converges faster
* \- Harder to implement
* SQL standard only requires linear

### Mutual Recursion

Example: Hubs and Authorities Algo for relation Link(src, dest), initial hubs and authorities need to be pre-designated explicitly

* Hub is defined as node which *points* to $\ge$ 3 Authority
* Authority is defined as node which is *pointed* by $\ge$ 3 Hub

```sql
with recursive
    Hub(node) as (select node from HubStart
                  union
                  select src as node from Link L
                  where src not in (select node from Auth)
                  and dest in (select node from Auth
                  group by src having count(*) >=3)),
    Autho(node) as (select node from AuthStart
                    union
                    select dest as node from Link L
                    where dest not in (select node from Hub)
                    and src in (select node from Hub
                    group by dest having count(*) >=3)
    select * from Hub;
```

> Note: this kind of mutual recursion was currently not supported by PostgreSQL system
> Note: negative dependency on other relation causes non-deterministic behaviors.

### Recursion with Aggregation
For relation P(x)
```sql
with recursive 
    R(x) as (select x from P
             union
             select sum(x) from R)
select * from R
```
There is no good definition for waht R should contain based on this recursion. So that recursion with agggregation is **disallowed**.

### Summary

Recursion extends expressiveness of SQL

* Basic functionality: linear recursion
* Extended functionality: nonlinear recursion, nutual recursion
* Disallowed: recursive subqueries(negative dependency), aggregation