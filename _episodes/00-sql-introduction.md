---
title: "Databases using SQL"
teaching: 60
exercises: 5
questions:
- "What is a relational database and why should I use it?"
- "What is SQL?"
objectives:
- "Describe why relational databases are useful."
- "Create and populate a database from a text file."
- "Define SQLite data types."
- "Select, group, add to, and analyze subsets of data."
- "Combine data across multiple tables."
keypoints:
- "SQL allows us to select and group subsets of data, do math and other calculations, and combine data."
- "A relational database is made up of tables which are related to each other by shared keys."
- "Different database management systems (DBMS) use slightly different vocabulary, but they are all based on the same ideas."
---

## Setup

_Note: this should have been done by participants before the start of the workshop._

We use [SQLite Manager](https://addons.mozilla.org/en-us/firefox/addon/sqlite-manager/)
and [rainfall data](https://drive.google.com/drive/folders/15HroRAa6L-asxbyEyUEVyQDCCG6GlYSV) from eThekwini throughout this lesson. 


# Motivation

To start, let's orient ourselves in our project workflow.  Previously, 
we used a spreadsheet and Python to go from messy, human created data 
to cleaned, computer-readable data.  Now we're going to use another advanced tool to analyze our data: SQL.  

## What is SQL?

SQL stands for Structured Query Language. SQL allows us to interact with relational databases through queries. 
These queries can allow you to perform a number of actions such as: insert, update and delete information in a database.


## Dataset Description

The data we will be using is the original eThekwini rainfall dataset. Our previous lessons have focussed on the *rainfall_combined.csv* that was a merge of all the eThekwini's datasets i.e. ward names, regions etc. Now let's download the
[full dataset](https://drive.google.com/drive/folders/15HroRAa6L-asxbyEyUEVyQDCCG6GlYSV) . (You should already have this).

## Questions

We'll need the following three files: 

* `raingauge_data.csv`
* `wards.csv`
* `regions.csv`
* `raingauges.csv`

> ## Challenge
>
> Open each of these csv files and explore them. 
> What information is contained in each file?  Specifically, if I had 
> the following research questions: 
> 
> * What is the peak rainfall intensity per raingauge per day?
> * What is the daily rainfall for each gauge?  
>* What is the peak rainfall intensity per ward per day?
> * What is the daily rainfall for each ward?
> 
> What would I need to answer these questions?  Which files of the data do I need? What 
> operations would I need to perform if I were doing these analyses by hand?  
{: .challenge}

## Goals

In order to answer the questions described above, we'll need to do the 
following basic data operations: 

* select subsets of the data (rows and columns)
* group subsets of data
* do math and other calculations
* combine data across spreadsheets

In addition, we don't want to do this manually!  Instead of searching 
for the right pieces of data ourselves, or clicking between spreadsheets, 
or manually sorting columns, we want to make the computer do the work.  

In particular, we want to use a tool where it's easy to repeat our analysis 
in case our data changes. We also want to do all this searching without 
actually modifying our source data.  

Putting our data into a relational database and using SQL will help us achieve these goals.  

> ## Definition: *Relational Database*
>
> A relational database stores data in *relations* made up of *records* with *fields*.
> The relations are usually represented as *tables*;
> each record is usually shown as a row, and the fields as columns.
> In most cases, each record will have a unique identifier, called a *key*,
> which is stored as one of its fields.
> Records may also contain keys that refer to records in other tables,
> which enables us to combine information from two or more sources.
{: .callout}

# Databases

## Why use relational databases

Using a relational database serves several purposes.

* It keeps your data separate from your analysis.
    * This means there's no risk of accidentally changing data when you analyze it.
    * If we get new data we can just rerun the query.
* It's fast, even for large amounts of data.
* It improves quality control of data entry (type constraints and use of forms in MS Access, Filemaker, Oracle Application Express etc.)
* The concepts of relational database querying are core to understanding how to do similar things using programming languages such as R or Python.
* It integrates well with web-based applications, providing the public with NB info.

## Database Management Systems

There are a number of different database management systems for working with
relational data. We're going to use SQLite today, however everything we
teach you will apply to the other database systems as well (e.g. MySQL,
PostgreSQL, MS Access, MS SQL Server, Oracle Database and Filemaker Pro). The 
only things that will differ are the details of exactly how to import and 
export data and the [details of data types](#datatypediffs).

## Relational databases

Let's look at a pre-existing database, the `rainfall_combined.sqlite`
file from the Portal Project dataset that we downloaded during
[Setup](''). Clicking on the "open file" icon, then
find that file and clicking on it will open the database.

You can see the tables in the database by looking at the left hand side of the
screen under Tables, where each table corresponds to one of the `csv` files 
we were exploring earlier.  To see the contents of any table, click on it, and
then click the “Browse and Search” tab in the right panel.  This will 
give us a view that we're used to - just a copy of the table.  Hopefully this 
helps to show that a database is, in some sense, just a collection of tables, 
where there's some value in the tables that allows them to be connected to each 
other (the "related" part of "relational database").  

The leftmost tab, "Structure", provides some metadata about each table.  It 
describes the columns, often called *fields*. (The rows of a database table 
are called *records*.)  If you scroll down in the Structure view, you'll 
see a list of fields, their labels, and their data *type*.  Each field contains 
one variety or type of data, often numbers or text.  You can see in the 
`raingauge_data` table that most fields contain numbers (integers) while the `wards` 
table is nearly all text.  

The "Execute SQL" tab is blank now - this is where we'll be typing our queries 
to retrieve information from the database tables.  

To summarize: 

* Relational databases store data in tables with fields (columns) and records
  (rows)
* Data in tables has types, and all values in a field have
  the same type ([list of data types](#datatypes))
* Queries let us look up data or make calculations based on columns

## Database Design

* Every row-column combination contains a single *atomic* value, i.e., not
   containing parts we might want to work with separately.
* One field per type of information
* No redundant information
    * Split into separate tables with one table per class of information
    * Needs an identifier in common between tables – shared column - to
       reconnect (known as a *foreign key*).

## Import

Before we get started with writing our own queries, we'll create our own 
database.  We'll be creating this database from the three `csv` files 
we downloaded earlier. Close the currently open database and then 
follow these instructions: 

1. Start a New Database 
    - **Database -> New Database**
    - Give a name **Ok -> Open**. Creates the database in the opened folder
2. Start the import **Database -> Import**
3. Select the `wards.csv` file to import
4. Give the table a name that matches the file name (`wards`), or use the default
5. If the first row has column headings, check the appropriate box
6. Make sure the delimiter and quotation options are appropriate for the CSV files.  Ensure 'Ignore trailing Separator/Delimiter' is left *unchecked*.
7. Press **OK**
8. When asked if you want to modify the table, click **OK**
9. Set the data types for each field using the suggestions in the table below (this includes fields from `regions`, `raingauge_data` and `raingauges` tables also):

| Field             | Data Type      | Motivation                                                                       | Table(s)          |
|-------------------|:---------------|----------------------------------------------------------------------------------|-------------------|
| ID               | INTEGER        | Having data as numeric allows for meaningful arithmetic and comparisons          | all           |
| TR             | DATETIME           | Field contains datetime data                                                 	| all           |
| UT   | INTEGER           | Field contains unix timestamp                                             | all           |
| data             | DOUBLE        | Field containing measured data          | raingauge_data           |
| *_id         | INTEGER        | Field contains numeric data	    						| all    |
| update_ref         | TEXT           | Field contains text data                                                 	| raingauge_data             |
| hours\_surrounding\_total              | DOUBLE           | Field contains numeric data                                                 	| raingauge\_data           |
| name        | TEXT           | Field contains text data								| raingauges  |
| location\_x,location\_y           | DOUBLE           | Field contains numeric data                                                 	| raingauges           |
| reference              | TEXT           | Field contains text data                                                 	| raingauges           |
| Region            | TEXT           | Field contains text data                                           | regions           |
| Ward              | TEXT        | Field contains text data                                 | wards           |


Finally, click **OK** one more time to confirm the operation.


> ## Challenge
>
> - Import the `raingauges`, `regions` and `raingauge_data` tables
{: .challenge}

You can also use this same approach to append new data to an existing table.

## Adding data to existing tables

1. "Browse and Search" tab -> Add
1. Enter data into a csv file and append


## <a name="datatypes"></a> Data types

| Data type                          | Description                                                                                              |
|------------------------------------|:---------------------------------------------------------------------------------------------------------|
| CHARACTER(n)                       | Character string. Fixed-length n                                                                         |
| VARCHAR(n) or CHARACTER VARYING(n) | Character string. Variable length. Maximum length n                                                      |
| BINARY(n)                          | Binary string. Fixed-length n                                                                            |
| BOOLEAN                            | Stores TRUE or FALSE values                                                                              |
| VARBINARY(n) or BINARY VARYING(n)  | Binary string. Variable length. Maximum length n                                                         |
| INTEGER(p)                         | Integer numerical (no decimal).                                                                          |
| SMALLINT                           | Integer numerical (no decimal).                                                                          |
| INTEGER                            | Integer numerical (no decimal).                                                                          |
| BIGINT                             | Integer numerical (no decimal).                                                                          |
| DECIMAL(p,s)                       | Exact numerical, precision p, scale s.                                                                   |
| NUMERIC(p,s)                       | Exact numerical, precision p, scale s. (Same as DECIMAL)                                                 |
| FLOAT(p)                           | Approximate numerical, mantissa precision p. A floating number in base 10 exponential notation.          |
| REAL                               | Approximate numerical                                                                                    |
| FLOAT                              | Approximate numerical                                                                                    |
| DOUBLE PRECISION                   | Approximate numerical                                                                                    |
| DATE                               | Stores year, month, and day values                                                                       |
| TIME                               | Stores hour, minute, and second values                                                                   |
| TIMESTAMP                          | Stores year, month, day, hour, minute, and second values                                                 |
| INTERVAL                           | Composed of a number of integer fields, representing a period of time, depending on the type of interval |
| ARRAY                              | A set-length and ordered collection of elements                                                          |
| MULTISET                           | A variable-length and unordered collection of elements                                                   |
| XML                                | Stores XML data                                                                                          |


## <a name="datatypediffs"></a> SQL Data Type Quick Reference

Different databases offer different choices for the data type definition.

The following table shows some of the common names of data types between the various database platforms:

| Data type                                               | Access                    | SQLServer            | Oracle             | MySQL          | PostgreSQL    |
|:--------------------------------------------------------|:--------------------------|:---------------------|:-------------------|:---------------|:--------------|
| boolean                                                 | Yes/No                    | Bit                  | Byte               | N/A            | Boolean       |
| integer                                                 | Number (integer)          | Int                  | Number             | Int / Integer  | Int / Integer |
| float                                                   | Number (single)           | Float / Real         | Number             | Float          | Numeric       |
| currency                                                | Currency                  | Money                | N/A                | N/A            | Money         |
| string (fixed)                                          | N/A                       | Char                 | Char               | Char           | Char          |
| string (variable)                                       | Text (<256) / Memo (65k+) | Varchar              | Varchar2 | Varchar        | Varchar       |
| binary object	OLE Object Memo	Binary (fixed up to 8K)   | Varbinary (<8K)           | Image (<2GB)	Long | Raw	Blob          | Text	Binary | Varbinary     |
