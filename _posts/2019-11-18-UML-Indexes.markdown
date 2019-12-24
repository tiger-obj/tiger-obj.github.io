---
layout:     post
title:      "Unified Modeling Language (UML) and Indexes"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-11-18 18:57:09
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - UML
    - Index
---

## Data Modeling

We've learned how to represent data for application for example use relational model or XML. In practice, database designer use higher-level model, which will then be translated into model of DBMS.

* Higher-Level Database Design Models
  * Entity-Relationship Model(E/R)
  * Unified Modeling Language(UML), a graphical language
  
### UML Data Modeling: 5 concepts

  1. Classes
     * Name, attributed, methods: For data modeling: add "primary key (pk)", drop methods.
  2. Associations
     * Relationships between objects of two classes. 
     * Multiplicity of Associations: Each object of Class $C_1$ is related to at least m and at most n objects of class $C_2$, denoted as m..n or m..\* (\>=m) or 0..n (\<=n) or 0..\* (no restrictions, abrev. as \*) or default (1..1 abrev. as 1).
     * Types of Relationships: One-to-One, Many-to-One, Many-to-Many and Complete(every object must participate in this relationship).
  3. Assoiation Classes
      * Relationships between objects of two classes, *with attributed on relationships*
      * UML can't describe the possibiliy of having more than one relationship or association between same student and the same college (in example of student applying to colleges).
      * Eliminating Association Classes: unecessary if 0..1 or 1..1 multiplicity.
      * Slef-Associations: associations between a class and itself.
  4. Subclasses
      * Subclasses (Specialization) inheritate (key) from superclass(Generalization).
      * Incompete (Partial) vs. Complete, where complete means every obj. in at least subclass.
      * Disjoint (Eclusive) vs. Overlapping, where disjoint means every obj. in at most one subclass.
  5. Composition & Aggregation
      * Composition: Objects of one class A belong to objects of another class B (1..1, filled diamend on the end of association attached to class B).
      * Aggregation: **some** objects of one class A belong to objects of another class B (0..1, empty diamend on the end of association attached to class B).

### Automatical transaltion

Designs in UML can be translated to relations automatically, provided every "regular" class has a key.

1. Class: Every class becomes a relation: pk -> primary key.
2. Association: Relation with key from each side.
   * Keys for association relations: depends on multiplicity. When we have 0..1 or 1..1 on one side of an association, then the key attribute from the other side **is a key** for the association.
   * When we have 0..1 or 1..1 on one side of an association, we actually could add key from this side to the class on the other end and eliminate the association. e.g.: $C1(K1,O1) \overset{1..1\;\;\;\;\;\;\;}{\longleftrightarrow} C2(K2,O2), A(K1,K2) \Rightarrow C1(K1,O1), C2(K2,O2,K1)$
3. Association Classes: Add attributes to relation for association.
   * since keys for association relations are new primary key, it indicates that it can't describe the possibiliy of having more than one relationship.
   * require a key for every "regular" class, where "regular" class is class on the end of association instead of association class.
   * Determining Keys for "Folding" still works.
   * Self-Associations: If one side has 1..1 then the other side is a key.
4. Subclasses: best translation may depend on properties.
   1. Subclass relations contain superclass key + specialized attributes: S(k,A), S1(k,B), S2(k,C)
   2. Subclass relations contain all attributes: S(k,A), S1(k,A,B), S2(k,A,C)
   3. One relation conatining all superclass + subclass attributes. S(k,A,B,C), makes it existence of null.
   * e.g. Heavily overlapping $\Rightarrow$ design 3, Disjoint and complete $\Rightarrow$ design 2.
   * Automatical translation requires keys for "regular" classes. Subclass inherites key from their superclass, and themselves are not considered as "regular" classes.
5. Composition \& Aggregation
   * Composition, i.e. 1..1 on filled diamend side: add key from this side to the class on the other end.
   * Aggregation, i.e. 0..1 on empty diamend side: add key from this side to the class on the other end and allow that attribute to be null.

## Indexes (Indices)

* Primary mechanism to get improved performance on a database
* Persistent data structure, stored in database

Utility:

* Index = difference between full table scans and inmmediate location of tuples.
* Underlying data structures
  * Balanced trees (B or B+ trees), fit for range query, Logarithmic
  * Hash table, only for equality query, Constant.
  
> Many DBMS's build indexes automatically on PRIMARY KEY (and somtimes UNIQUE) attributes. More about Query planning and optimization.

* Downsides of Indexes
  1. Extra space - marginal
  2. Index creation - medium
  3. Index maintenance - can even offset benifits

* Benefit of an index depends on:
  * Size of table (and possibly layout)
  * Data distributions
  * Query vs. update load

* SQL Syntax
  * Create Index IndexName on T(A)
  * Create Index IndexName on T(A1,A2,...,An)
  * Create Unique Index IndexName on T(A)
  * Drop Index IndexName
