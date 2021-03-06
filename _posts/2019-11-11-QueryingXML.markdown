---
layout:     post
title:      "Querying XML"
subtitle:   "Notes of Introduction to Databases Stanford"
date:       2019-11-11 14:04:32
author:     "tiger-obj"
header-img: "img/database_stanford/databases.png"
catalog: true
mathjax: true
tags:
    - Database
    - XML
    - XPath
    - XQuery
---

Querying XML is not as matrue as Querying Relational (e.g. SQL)

|Xpath | XSLT | XQuery|
|-----|-----|------|
|path expressions + conditions | XPath + transformations, output formatting | XPath + full-featured Query Language|

## XPath: Path expressions + Conditions


**Think of XML as a tree, navigating in the tree using xpath**

### Construction of XPath

* Basic Constructs
  
|Construct| function|
|-----|-----|
|/|root element, separator|
|name of element|"*" matches anything|
|@A| A: attribute name|
|//|match any descendant, including the element we currently are|
|[c]| c is expression of condition, or a number indicates index|

* Built-in functions 
  
  |----|-----|
  contains(S1,S2) | returns true if first string contains second string.
  name() | returns tag of current element in the path.

* Navigation axes
  
  |----|-----|
  |parent:: | returns parents, search up in the tree|
  |following-sibling:: | siblings after current element|
  |descendant:: | same as //|
  |self:: | current element|

* More Details:
  * XPath queries operate on and return sequence of elements
  * XPath result could be expressed as XML but not always

### Examples

  * **(A\|B)** in path indicates A or B.
  * **"//*"** returns XML it self.
  * some attributes do not have structual result so we use **data(@attribute)** to serillize the result
  * **Book[@Price < 90]** returns books whose price is less than 90.
  * **Book[Remark]** returns book that *has* a remark.
  * intersection of independent multiple full path conditions is **NOT** equivalent to  multiple conditions in a single full path expression.

  * ***doc("BookstoreQ.xml")//Book[contains(Remark,"great")]/Title*** returns Titles of books with a remark containing "great".
  * ***doc("BookstoreQ.xml")//Magazine[Title=doc("BookstoreQ.xml")//Book/Title]*** returns all magazines where there's a book with the same tile.
  * ***doc("BookstoreQ.xml")/Booksore//\*[name(parent::\*)!="Bookstore" and name(parent::\*) != "Book"]*** returns all elemennts whose parent is not "Bookstore" or "Book".
  * ***doc("BookstoreQ.xml")/Bookstore/(Book\|Magazine)[Title = following-sibling::\*/Title or Title = preceding-sibling::\*/Title]*** returns all books and magazines with non-unique titles (both side of each equality).
  * ***XPath revolves around implicit existential quantification.*** means "A=B" in XPath is equivalent to $\exists elements \in A \space that \in B$. If we want a uniform equal relation: ***doc("BookstoreQ.xml")//Book[count(Authors/Author[contains(First_name,"J") = count(Authors/Author/First_Name)])]*** returns Books where every author's first name includes "J".
  * ***doc("BookstoreQ.xml")//Book[Authors/Author/Last_Name="Ullman" and count(Authors/Author[Last_Name = "Widom"])=0]/Title*** returns titles of books where "Ullman" is an author and "Widom" is not an author.

## XQuery

* XQuery is an expression language (compositional), which means result of XQuery can be input of another XQuery.
* Each expression operates on and returns sequence of elements.
* XPath is one type of expression
  
### FLWOR (For, Let, Where, Order by, Return) expression

```xquery
For $var in expr    (:iterator variables where expr is a set of N elements:)
Let $var:= expr     (:assignment:)
Where condition     (:filter:)
Order By expr       (:sorts result:)
Return expr         (:The only necessary expression:)
```

* For and Let can be repeated and interleaved
* Note: let expression is an assignment where needs a ":" before "=".
* XQuery result can be mixed into XML as well: ```<<Result>{query}</Result>```


### Examples
Again using bookstore xml

1. Returns titles and author first names of books whose title contains one of the author's first names, in return closure we construct a xml:

    ```xquery
    for $b in doc("BookstoreQ.xml")/Bookstore/Book
    where some $fn in $b/Authors/Author/First_Name
            satisfies contains($b/Title,$fn)
    return  <Book>
                {$b/Title}
                {$b/Authors/Author/First_Name} 
                //all first names of those books
                { for $fn in $b/Authors/Author/First_Name 
                    where contains($b/Title,$fn) return $fn} 
                //include only those first names showing up in titles
            </Book>
    ```

2. Find the books whose price is below average

    ```xquery
    let $a := avg(doc("BookstoreQ.xml")/Bookstore/Book/@Price)
    for $b in doc("BookstoreQ.xml")/Bookstore/Book
    where $b/@Price < $a
    order by xs:int($b/@Price)
    (:xs:int is a built-in funcion converts string to int:)
    return  <Book>
                { $b/Title }
                <Price>{ $b/data(@Price)} </Price>
            </Book>
    ```

3. get distinct last names

    ```xquery
    for $n in distinct-values(doc("BookstoreQ.xml")//Last_Name)
    return <Last_Name> {$n} </Last_Name>
    ```

    * Note: distinct-alues() only return seriliezed values, tags need to be added manuelly
    * Note: <>**{}**<>, curly brackets are necessary to make xquery to evaluate the query inbetween.
4. use every to explictely enforce uniforme equal, returns Books where every author's first name includes "J".

    ```xquery
    for $b in doc("BookstoreQ.xml")/Bookstore/Book
    where every $fn in $b/Authors/Author/First_Name
            satisfies contains($fn,"J")
    return $b
    ```

5. Self-Join, Note that results are pairs where one Book "exists" last name equls some last name to the other.

    ```xquery
    for $b1 in doc("BookstoreQ.xml")/Bookstore/Book
    for $b2 in doc("BookstoreQ.xml")/Bookstore/Book
    where $b1/Authors/Author/Last_Name = $b2/Authors/Author/Last_Name 
            and $b1/Title < $b2/Title
    return
        <BookPair>
            <Title1> {data{$b1/Title}} </Title1>
            <Title2> {data{$b2/Title}} </Title2>
        </BookPair>
    ```

6. Invert Data: Authors with the books they've written. Assume Last Names are unique.

    ```xquery
    <InvertedBookstore>
        {for $ln in distinct-values(doc("BookstoreQ.xml")//Author/Last_Name)}
        {for $fn in distinct-values(doc("BookstoreQ.xml")//Author/First_Name)}
        return 
            <Author>
                <First_Name> {$fn} </First_Name>
                <Last_Name> {$ln} </Last_Name>
                {for $b in doc("BookstoreQ.xml")/Bookstore/
                            Book[Authors/Author/Last_Name=$ln]
                return  <Book>
                            {$b/@ISBN}{$b/Price}{$b/Edition}
                            {$b/Title}{$b/Remark}
                        </Book>
                }
            </Author>
    </InvertedBookstore>
    ```

    * Note: Although not every book has Edition or Remark, it's not a problem.

## XSLT

>XSL = Extensible Stylesheet Language <br>
>XSLT = Extensible Stylesheet Language (with) Transformations

### Rule-Based Transformations

* Match template and replace
* Recursively match templates
* Extract values
* Iteration (for-each)
* Conditionals (if)
* Strange default/whitespace behavior, if there exists any elements that doesn't match any template we give. We need to use <xsl:template match="text()"/> to discard redudant text information. Namely we project(replace) text to spaces.
* Implicit template priority. If there are multiple templates match, xsl processor chooses the "concreter" (the one with more specific decription) one. If there are multiple templates with same "descriptive power" match, processor chooses the second (later showing up) one.

### Briefly summary from Example

Since lecture video is reletively fast while xslt is pretty complicated, we are going to study typical structure of XSLT using concrete query examples in post-deadline quiz.

1. XSLT to return a list of department titles.
  
    ```xml
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        
        <xsl:template match="Course[@Enrollment &gt; 500]">
            <xsl:copy-of select="."/>
        </xsl:template>
        <xsl:template match="text()"/>

        <xsl:template match="text()"/>

    </xsl:stylesheet>
    ```

    Note:
    1. First two and last lines are template of one xslt query. As we could see, xslt is exactly a file in xml format.
    2. Multiple templates may exists in the body of stylesheet. They work like filter and excute "projection". copy-of or value-of will project those value with or without tag into results. If template has an **empty body** means that we want contains matching that template to be filtered out.
    3. match="**a XPath expression**". Difference here comparing to canonical XPath expression are it is written as a string in quotes. Several escapes need to be applied instead, for example:

    
    |XPath|Match expr|example|
    |---|---|---|
    |>|```&gt;```|\<xsl:template match="Course[@Enrollment \&gt; 500]">|
    |<|```&lt;```|\<xsl:template match="Course[@Enrollment \&lt; 500]">|
    |"|```&quot;``` or '|\<xsl:template match="country[contains(@name,```&quot;```stan```&quot;```)]"> <br>or<br>\<xsl:template match="country[contains(@name,'stan')]"> |


2. Example of generic template, which matches all possible elements. It is necessary for (succeeding) ***recursively*** match templates:
   
   ```xml
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="*|@*|text()">
        <xsl:copy>
        <xsl:apply-templates select="*|@*|text()" />
        </xsl:copy>
    </xsl:template>
    <xsl:template match="country[@area &gt; 40000]"/>
    <xsl:template match="country[count(city)=0]"/>
    </xsl:stylesheet>
    ```
    Note:
    1. "\*\|@\*\|text()" represents all children nodes, attributes and text-node children.
    2. Using this methode we can copy whole document and add other templates as exeption of copying. Generic template at the beginning is necessary to keep the structure of querying result. By calling generic template, we enforce each succeeding template to be applied on each type of elments (nodes, attributes and so on) of the whole document.

3. XSLT an also be used for transforming xml to html, e.g. intepreting query result in browser, more details can be found in lecture video. In the following is an example of using XSLT for \"transforming\" result into new structure (\<Stan>\</Stan> in this case).

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="country[contains(@name,&quot;stan&quot;)]">       
        <Stan><xsl:value-of select="./@name" /></Stan>
    </xsl:template>
    <xsl:template match="text()"/>
</xsl:stylesheet>
```