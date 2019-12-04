---
layout:     post
title:      "Constraints and Triggers"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-12-02 17:12:46
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - Constraints and Triggers
---

## Constraints and Triggers
  
* For relational databases
* SQL standard; different systems vary considerably
  
### (Integrity) Constraints (static)
  
  * constrain alowable database states
  * Impose restrictions on allowable data, beyond those imposed by structure and types
  * e.g: $0<GPA \le 4.0$, $enrollment < 50000$, decision $\in$ \{'Y','N',null\}, 
  major='cs' $\Rightarrow$ decision=null, sizeHS<200 $\Rightarrow$ not admitted by enrollment > 30000

#### Motivation: 
    
  1. Data-entry errors (insert) 
  2. Correctness criteria (update) 
  3. Enforce consistency 
  4. Tell system about data - store, query processiong

#### Different types of constraints:

 * Non-null (insert and update must conform to not null GPA): 
   * create table Student(sID int, sName text, GPA real ***not null***, sizeHS int)
 * Key (insert and update must conform to unique primary key):
   * only one element can be primary key, others can be still decalared as ***unique***
     * create table Student(sID int ***primary key***, sName text, GPA real, sizeHS int)
   * pair of keys as primary key
     * create table College(cName text, state text, enrollment int, ***primary key (cName, state)***)
   * multipule pairs of keys
     * create table Apply(sID int, cName text, decision text, ***unique(sID, cName), unique(sID, major)***)
   * most sql systems allow repeated null values  

 * Referential integrity (foreign key) ``references``
   * Integrity of references = No "dangling pointers"
   * Directional: Referential integrity from R.A to S.B means Each value in column A of table R must appear in colum B of table S. ($R.A \subseteq S.B$)
   * A is called the "foreign key", Referential integrity is also called the Foreign key constraints.
   * B is usually required to be the primary key for table S or at least unique (for efficient implementation)
   * Multi-attribute foreign keys are allowed.
   * Potentiall Violations occur: 
     * Insert into R
     * Delete from S (``on delete set null/cascade``)
       * Special actions: Restrict(default generate error), Set Null, Cascade(delete entrys in R that has the reference in S)
     * Update R.A
     * Update S.B (``on update set null/cascade``)
       * Special actions: Restrict(default generate error), Set Null, Cascade(propagate update entrys in R that has the reference in S)
   * End of table definition: 
     * ``create table T(A int, B int, C int, primary key (A,B), foreign key (B,C) references T(A,B) on delete cascase)``

 * Attribute-based and tuple-based
   * ***Attribute:*** create table Student(sID int, sName text, GPA real ***check(GAP $\le$ 4.0 and GPA >0.0)***, sizeHS int ***check(sizeHS<5000)***)
   * ***Tuple:*** create table Apply(sID int, cName text, decision text, ***check(decision = 'N' or cName <> 'Stanford' or major <> 'CS'***) 
     * No CS student admits to Stanford
   * create table T(A int check(A not in (select A from T)));
     * Trying to enforce key constraint on T.A, *can't be executed*
   * create table T(A int check((select count(distinct A) from T)=(select count(*) from T))); 
     * Enforce A is a key for T
 * General assertion (not implemented in any DBMS)
 ``
 create assertion Key
 check((select count(distinct A) from T) = (select count(*) from T))
 ``
#### Declaring and enforcing constraints
  * Declaration
    * With original schema - checked after bulk loading
    * Or latar - checked on current Database
  * Enforcement 
    * Check after every "dangerous" modification
    * Deferred constraint checking, check after every "dangerous" *transaction*

### Triggers (dynamic)

* monitor database changes, check conditions and initiate actions
* "Event-Condition-Action Rules": When event occurs, check condition; if true, do action.
* E.g. enrollment>35000 $\Rightarrow$ reject all applicants, insert app with GPA>3.95 $\Rightarrow$ accept automatically, update sizeHS to be>7000 $\Rightarrow$ change to "wrong" and raise an arrow.
#### Motivation:
  * Move monitoring logic from application into DBMS
  * To enforce constraints, due to expressiveness and constraint "repair" logic
  * Implementations vary significantly
#### Triggers in SQL (SQL standard)
> contents between () are explantation:

```sql
Create Trigger name
Before | After | Instead Of events
    (insert (new data) or delete (old data) on T, update[of c1,c2,...,cn] on T (new and old data))
[ referencing-variables] 
    (old row as var|new row as var|old table as var|new table as var,
    var can be later referred in the trigger condition and action)
[ For Each Row ] 
    (once for each modified tuple, if absent only table can be referred)
When ( condition )
    (condition like SQL where, like general assertion)
action
    (SQL statement)
```
* Row-level vs. Statement-level
  * New/Old Row and New/Old Table
  * Before, Instead of
* Multiple triggers activated at same time, which goes first?
* Trigger actions activating other triggrs(chaining)
  * Also self-triggering, cycles, nested invocations
* Conditions in When vs. as part of action

> Implementations vary significantly

|Postgres|SQLite|MySQL|
|---|---|---|---|
|Expressiveness/behavior=full standard<br>Cumbersome & awkward syntax|Row-level only, immediate activation <br>$\Rrightarrow$ no old/new table|Row-level-only, immediate activation <br>$\Rrightarrow$ no old/new table <br>Only one trigger per event type <br>Limited trigger chaining|

#### Example

```sql
Create Trigger IncreaseInserts
After Insert On T
Referencing New Row As NR, New Table As NT
For Each Row
When (Select Avg(V) From T) < (Select Avg(V) From NT)
Update T set V=V+10 where K=NR.K
```

* No statement-level equivalent
* ***Nondeterministic final state***
  
---
SQLite: row-level triggers, ***immediate*** activation
  * *For Each Row* implicit if not specified
  * No *Old Table* or *New Table*
  * No *Referencing* clause
  * *Old* and *New* predefined for *Old Row* and *New Row*
  * Trigger action: SQL statements in *Begin-End* block

---
Similar to referential integrity constraint:
```sql
create trigger R2
after delete on Student
for each row
begin
    delete from Apply where sID=Old.sID
end;
```
```sql
create trigger R3
after update of cName on College
for each row
begin
    update Apply
    set cName=New.cName
    where cName=Old.cName;
end;
```
```sql
create trigger R4
before insert on College
for each row
when exists (select * from College where cName=New.cName)
begin
    select raise(ignore);
end;
```
```sql
create trigger R26
after insert on Apply
for each row
when (select count(*) from Apply where cName=New.cName)>10
begin
    update College set cName=cName || '-Done'
    where cName = New.cName;
end;
```
```sql
create trigger TooMany
after update of enrollment on College
for each row
when(Old.enrollment <=16000 and New.enrollment > 16000)
begin 
    delete from Apply
        where cNamae = New.cName and major = 'EE';
    update Apply
        set decision = 'U'
        where cName = New.cName
        and decision = 'Y';
end;
```
Note: last trigger can't be implemented by constraints

* Self-triggering, cycles by using ``pragma recursive_triggers = on`` in SQLite system, while a termination condition may be necessary.
* Confilicts: The priority of triggers that activated at the same time, depends on their order of declaration.
* Nested trigger invocations: in one perticular trigger the next action is activated only after trigger chain of previous action returns.
