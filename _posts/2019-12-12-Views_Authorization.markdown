---
layout:     post
title:      "Views and Autorization"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-12-12 17:36:17
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - Views
    - Authorization
---

## Views

Three-level vision of database:

Physical-Conceptual-Logical. As from logical-level vision, view represents a further abstraction of conceptual-level vision, i.e. relations.

### Motivation

* Hide some data from some users
* Make some queries easier / more natural
* Modularity of database access
* *Real applications tend to use lots and lots of views.*

### Defining and using views

* View V=ViewQuery(R1,R2,...,Rn)
* Schema of V is schema of query result
* Query Q involving V, conceptually
    ```
    V := ViewQuery(R1,R2,...,Rn)
    Evaluate Q
    ```
* In reality, Q rewritten to use R1,...,Rn instead of V
* Note: Ri could itself be a view

SQL Syntax
```sql
Create View Vname As
<Query>
Create View Vname(A1,A2,...,An) As
<Query>
Drop Vname
(error in Postgre by dropping,
and error in SQLite and MySQL by invoking Vname later)
```

> Note: views are not stored physically.

In order to be updatable according to the SQL standard, a view must:

* Have only one table T in its top-level FROM clause
* Not use SELECT DISTINCT in its top-level FROM clause
* Include all attributes from T that do not permit NULLs
* Not refer to T in subqueries
* Not use GROUP BY or aggregation

### View Modifications

Querying views

* Once V defined, can reference V like any table
* Queries involving V rewritten to use base tables

Modifying views

* For some users, for those views are the entire "view" of the database, we need make views-modification possible for them.

* Solution: Modifications to **V** rewritten to modify **base tables**. The rewritten (translation) can usually be done without problem, however there are often many possible modification, and the system doesn't know which is intended by the user.
  * Rewriting process specified explicitly by view creator
    * Enabled by *Instead of* Triggers
    * \+ Can handle all modifications
    * \- But no guarantee of correctness (or meaningfull)
  * <u>Restrict views + modifications</u> so that translation to base table modifications is meaningful and unambiguous
    * Due to SQL standard, very rigorous
    * \+ No user intervention
    * \- However, restrictions are significant

### View Modifications using *INSTEAD OF* triggers

* e.g. we would like to delete some students from CSaccept which is a view of those accepted cs students (SQLite)

```sql
create trigger CSaccept Delete
instead of delete on CSaccspt
for each row
begin
    delete from Apply
    where sID=Old.sID
    and cName=Old.cName
    and mahjor='CS' and decision ='Y';
end;
```
here old reprensents the deleted tuple in view.

> Note: Nothing would prevent triggers doing "wrong" thing. A equivalent trigger is very important and the responsibility of programmer. Sometimes designer won't allow anything happen at all for ambiguous modificaiton or modifications that don't make much sence (modifying aggregation results or queries contains self interaction).

### Automatic View Modifications

Only supported by MySQL instead of SQLite or PostgreSQL

Restrictions in SQL Standard for "updatable views":

1. SELECT (no DISTINCT) on single table T
2. Attributes not in view can be NULL or have default value
3. Subqueries must not refer to T
4. No GROUP BY or aggregation

* Example: use *with check option* to make insertion to use existed information

```sql
create view CSaccept2 as 
select sID, cName
from Apply
where major = "CS" and decision = "Y"
with check option
```

won't allow the following insertion to be executed (since major and decision are NULL by default and that inserted tuple won't appear in the view)

```sql
insert into CSaccept2 values (444,'Berkeley');
```

> Note: *with check option* will affect the efficiency of database

### Materialized Views

Comparing to virtual views, materialized view could improve query performance.

Queries over materialized Views

* View V=ViewQuery(R1,R2,...,Rn)
* Create table V with schema of query result
* Execute ViewQuery and put results in V
* Queries refer to V as if it's a table
  
Downsides:

* V could be very large
* Modifications to R1, R2, ..., Rn $\Rightarrow$ recopute or modify V
* Modifications to base data invalidate view

Modifications on materialized views

* Good news: just update the stored table
* Bad news: base tables must stay in synch
  * Same issues as with virtual views
* Modifications to V must also modify base tables

(Efficiency) benefitss of a materialized view depend on: (Qeury-Update tradeoff)

* Size of data
* Complexity of view
* Number of queries using view
* Number of modificaitons affecting view
  * Also "incremental mantenance" versus full recomputation.

Database systems sometimes do automatic query rewriting to use materialized views

## Database Authorization

### Motivation

* Make sure users see only the data they're supposed to see
* Gard the database against modifications by malicious users

> SQL Injection and other system security issues will not covered here

### Examples

Users have privileges and can only operate on data for which they are authorized, for example:

```sql
Update Apply
Set dec='Y'
where sID In (Select sID From Student Where GPA>3.9)
```

Privileges needed:

* Apply: update(dec), select(sID)
* Stduent: select(sID,GPA)

In order to provide privileges on specific attributes of some relations, we need to get use of views. Must have updatable views to provide modification privileges through views.

### Obtaining Privileges

* Relation creator is *owner*
* Owner has all privileges and *may grant privileges*

```sql
Grant privs On R To users
[With Grant Option]
```
where privs is like "select(sID), Delete", users could be public, Option could grant same or lesser to others.

### Revoking Privileges

```sql
Revoke privs On R From users
[Cascade | Restrict]
```

* Cascade: revoke all privileges granted by its descendents. Also revoke privileges granted fro mprivileges being revoked(transitively), unless also granted from another source.
* Restrict: Disallow if Cascade would revoke any other privileges. For transitive cases, need to revoke privileges manualy bottom-up through that graph. As default if not explicitly specified.