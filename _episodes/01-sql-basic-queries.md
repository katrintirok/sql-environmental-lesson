---
title: "Basic Queries"
teaching: 30
exercises: 5
questions:
- "How do I write a basic query in SQL?"
objectives:
- "Write and build queries."
- "Filter data given various criteria."
- "Sort the results of a query."
keypoints:
- "It is useful to apply conventions when writing SQL queries to aid readability."
- "Use logical connectors such as AND or OR to create more complex queries."
- "Calculations using mathematical symbols can also be performed on SQL queries."
- "Adding comments in SQL helps keep complex queries understandable."
---

## Writing my first query

Let's start by using the **raingauge_data** table. Here we have rainfall data for every
raingauge within eThekwini, including the raingauge\_id that the data belongs to, when the data was uploaded
and the unix timestamp of the data.

Let’s write an SQL query that selects only the data column from the
raingauge_data table. SQL queries can be written in the box located under 
the "Execute SQL" tab. Click 'Run SQL' to execute the query in the box.

    SELECT data
    FROM raingauge_data;

We have capitalized the words SELECT and FROM because they are SQL keywords.
SQL is case insensitive, but it helps for readability, and is good style.

If we want more information, we can just add a new column to the list of fields,
right after SELECT:

    SELECT UT, data, raingauges_id
    FROM raingauge_data;

Or we can select all of the columns in a table using the wildcard *

    SELECT *
    FROM raingauge_data;

### Limiting results

Sometimes you don't want to see all the results you just want to get a sense of
of what's being returned. In that case you can use the LIMIT command. In particular
you would want to do this if you were working with large databases.

    SELECT *
    FROM raingauge_data
    LIMIT 10; 

### Unique values

If we want only the unique values so that we can quickly see what species have
been sampled we use `DISTINCT` 

    SELECT DISTINCT raingauges_id
    FROM raingauge_data;

If we select more than one column, then the distinct pairs of values are
returned

    SELECT DISTINCT data, raingauges_id
    FROM raingauge_data;

### Calculated values

We can also do calculations with the values in a query.
For example, if we wanted to look at the rainfall at each raingauge
on different dates, but we needed it in cm/5min instead of mm/5min we would use

    SELECT UT, data/10
    FROM raingauge_data;

When we run the query, the expression `data / 10` is evaluated for each
row and appended to that row, in a new column. If we used the `INTEGER` data type
for the data field then integer division would have been done, to obtain the
correct results in that case divide by `10.0`. Expressions can use any fields,
any arithmetic operators (`+`, `-`, `*`, and `/`) and a variety of built-in
functions. For example, we could round the values to make them easier to read.

    SELECT  UT, data, ROUND(data / 10, 2)
    FROM raingauge_data;

> ## Challenge
>
> - Write a query that returns the UT, update_ref, raingauges_id and data in m
{: .challenge}

## Filtering

Databases can also filter data – selecting only the data meeting certain
criteria.  For example, let’s say we only want data for the first raingauge
_1_.  We need to add a
`WHERE` clause to our query:

    SELECT *
    FROM raingauge_data
    WHERE raingauges_id=1;

We can do the same thing with text.
Here, we only want the data with a certain update_ref.

    SELECT * FROM raingauge_data
    WHERE update_ref = '1512227100-1';

<!--If we used the `TEXT` data type for the year the `WHERE` clause should
be `year >= '2000'`.--> We can use more sophisticated conditions by combining tests
with `AND` and `OR`.  For example, suppose we want the data on the first raingauge
after the Unix Timestamp 1512327300:

    SELECT *
    FROM raingauge_data
    WHERE (UT >= 1512327300) AND (raingauges_id = 1);

Note that the parentheses are not needed, but again, they help with
readability.  They also ensure that the computer combines `AND` and `OR`
in the way that we intend.

If we wanted to get data for any of the raingauges, which have
ID's 1, 5, and 10, we could combine the tests using OR:

    SELECT *
    FROM raingauge_data
    WHERE (raingauges_id = 1) OR (raingauges_id = 5) OR (raingauges_id = 12);

> ## Challenge
>
> - Produce a table listing the data for all raingauges with rainfall more than 20mm/5min, telling us the Unix Timestamp, raingauges id, and data
> (in cm). 
{: .challenge}

## Building more complex queries

Now, lets combine the above queries to get data for the _3_ raingauges above from
the Unix TimeStamp 1512327300 on.  This time, let’s use IN as one way to make the query easier
to understand.  It is equivalent to saying `WHERE (raingauges_id = 1) OR (raingauges_id
= 5) OR (raingauges_id = 10)`, but reads more neatly:

    SELECT *
    FROM raingauge_data
    WHERE (UT >= 1512327300) AND (raingauges_id IN (1, 5, 10));

We started with something simple, then added more clauses one by one, testing
their effects as we went along.  For complex queries, this is a good strategy,
to make sure you are getting what you want.  Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.

When the queries become more complex, it can be useful to add comments. In SQL,
comments are started by `--`, and end at the end of the line. For example, a
commented version of the above query can be written as:

    -- Get post UT: 1512327300 data on raingauges
    -- These are in the raingauge_data table, and we are interested in all columns
    SELECT * FROM raingauge_data
    -- Sampling Unix time is in the column `UT`, and we want to include 1512327300
    WHERE (UT >= 1512327300)
    -- Raingauges have the `raingauges_id` 1, 5, and 10
    AND (raingauges_id IN (1, 5, 10));

Although SQL queries often read like plain English, it is *always* useful to add
comments; this is especially true of more complex queries.

## Sorting

We can also sort the results of our queries by using `ORDER BY`.
For simplicity, let’s go to the **raingauges** table and order it by ward_id.

First, let's look at what's in the **raingauges** table. It's a table of the raingauges, their ID's, names and the ward they belong to. Having this in a separate table is nice, because we didn't need to include all
this information in our main **raingauge_data** table.

    SELECT *
    FROM raingauges;

Now let's order it by taxa.

    SELECT *
    FROM raingauges
    ORDER BY ward_id ASC;

The keyword `ASC` tells us to order it in Ascending order.
We could alternately use `DESC` to get descending order.

    SELECT *
    FROM raingauges
    ORDER BY ward_id DESC;

`ASC` is the default.

We can also sort on several fields at once.
To be alphabetical, we might want to order by genus then species.

    SELECT *
    FROM raingauges
    ORDER BY ward_id ASC, name ASC;

> ## Challenge
>
> - Write a query that returns UT, data in cm, raingauge\_id from
> the raingauge\_data table, sorted with the largest rainfall at the top.
{: .challenge}

## Order of execution

Another note for ordering. We don’t actually have to display a column to sort by
it.  For example, let’s say we want to order the birds by their species ID, but
we only want to see genus and species.

    SELECT name, ward_id
    FROM raingauges
    WHERE id >= 10
    ORDER BY ward_id ASC;

We can do this because sorting occurs earlier in the computational pipeline than
field selection.

The computer is basically doing this:

1. Filtering rows according to WHERE
2. Sorting results according to ORDER BY
3. Displaying requested columns or expressions.

Clauses are written in a fixed order: `SELECT`, `FROM`, `WHERE`, then `ORDER
BY`. It is possible to write a query as a single line, but for readability,
we recommend to put each clause on its own line.

> ## Challenge
>
> - Let's try to combine what we've learned so far in a single
> query.  Using the raingauge\_data table write a query to display the UT field,
> raingauges\_id, and data in cm (rounded to two decimal places), for
> data from UT:1512338400, ordered numerically by the raingauges\_id.
> - Write the query as a single line, then put each clause on its own line, and
> see how more legible the query becomes!
{: .challenge}

