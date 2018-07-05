---
title: "SQL Aggregation and aliases"
teaching: 50
exercises: 10
questions:
- "How can I summarize my data by aggregating, filtering, or ordering query results?"
- "How can I make sure column names from my queries make sense and aren't too long?"
objectives:
- "Apply aggregation to group records in SQL."
- "Filter and order results of a query based on aggregate functions."
- "Employ aliases to assign new names to items in a query."
- "Save a query to make a new table."
- "Apply filters to find missing values in SQL."
keypoints:
- "Use the `GROUP BY` keyword to aggregate data."
- "Functions like `MIN`, `MAX`, `AVERAGE`, `SUM`, `COUNT`, etc. operate on aggregated data."
- "Aliases can help shorten long queries. To write clear and readible queries, use the `AS` keyword when creating aliases."
- "Use the `HAVING` keyword to filter on aggregate properties."
- "Use a `VIEW` to access the result of a query as though it was a new table."
---

## COUNT and GROUP BY

Aggregation allows us to combine results by grouping records based on value, also it is useful for
calculating combined values in groups.

Let’s go to the raingauge_data table and find out how many data entries there are.
Using the wildcard * simply counts the number of records (rows):

    SELECT COUNT(*)
    FROM raingauge_data;

We can also find out the total rainfall:

    SELECT COUNT(*), SUM(data)
    FROM raingauge_data;

We can output this value in metre (dividing the value to 1000.0), then rounding to 3 decimal places:
(Notice the divisor has numbers after the decimal point, which forces the answer to have a decimal fraction)

    SELECT ROUND(SUM(data)/1000.00, 3)
    FROM raingauge_data;

There are many other aggregate functions included in SQL, for example:
`MAX`, `MIN`, and `AVG`.

> ## Challenge
>
> Write a query that returns: total rainfall, average rainfall, and the min and max rainfall
> for all raingauges over the duration of the observation period.
> Can you modify it so that it outputs these values only for rainfall between 1 mm/5min and 5 mm/5min?
{: .challenge}

Now, let's see how many data entries there are for each raingauge. We do this
using a `GROUP BY` clause

    SELECT raingauges_id, COUNT(*)
    FROM raingauge_data
    GROUP BY raingauges_id;

`GROUP BY` tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give `GROUP BY` a comma separated list.

> ## Challenge
>
> Write queries that return:
>
> 1. How many data entries there are for each UT
>    
> 2. Average rainfall of each UT
>
> Can you modify the above queries combining them into one?
{: .challenge}

## Ordering Aggregated Results

We can order the results of our aggregation by a specific column, including
the aggregated column.  Let’s count the number of data entries for each
raingauge, ordered by the count:

    SELECT raingauges_id, COUNT(*)
    FROM raingauge_data
    GROUP BY raingauges_id
    ORDER BY COUNT(raingauges_id);

## Aliases

As queries get more complex names can get long and unwieldy. To help make things
clearer we can use aliases to assign new names to things in the query.

We can use aliases in column names or table names using `AS`:

    SELECT MAX(data) AS peak_rainfall
    FROM raingauge_data;

The `AS` isn't technically required, so you could do

    SELECT MAX(data) pk
    FROM raingauge_data;

but using `AS` is much clearer so it is good style to include it.

## The `HAVING` keyword

In the previous episode, we have seen the keyword `WHERE`, allowing to
filter the results according to some criteria. SQL offers a mechanism to
filter the results based on **aggregate functions**, through the `HAVING` keyword.

For example, we can request to only return information
about raingauges with more than 200 entries:

    SELECT raingauges_id, COUNT(raingauges_id)
    FROM raingauge_data
    GROUP BY raingauges_id
    HAVING COUNT(raingauges_id) > 200;

The `HAVING` keyword works exactly like the `WHERE` keyword, but uses
aggregate functions instead of database fields to filter.

If you use `AS` in your query to rename a column, `HAVING` you can use this
information to make the query more readable. For example, in the above
query, we can call the `COUNT(raingauges_id)` by another name, like
`occurrences`. This can be written this way:

    SELECT raingauges_id, COUNT(raingauegs_id) AS occurrences
    FROM raingauge_data
    GROUP BY raingauges_id
    HAVING occurrences > 200;

Note that in both queries, `HAVING` comes *after* `GROUP BY`. One way to
think about this is: the data are retrieved (`SELECT`), which can be filtered
(`WHERE`), then joined in groups (`GROUP BY`); finally, we can filter again based on some
of these groups (`HAVING`).

> ## Challenge
>
> Write a query that returns, from the `raingauges` table, the number of
> `raingauges` in each `ward`, only for the `ward` with more than 2 `gauges`.
>
>> ## Did you get it right?
>> ```
>> SELECT name, COUNT(*) AS number
>> FROM raingauges GROUP BY ward_id HAVING number>=2
>> ```
> {: .solution}
{: .challenge}

## Saving Queries for Future Use

It is not uncommon to repeat the same operation more than once, for example
for monitoring or reporting purposes. SQL comes with a very powerful mechanism
to do this by creating views. Views are a form of query that is saved in the database,
and can be used to look at, filter, and even update information. One way to
think of views is as a table, that can read, aggregate, and filter information
from several places before showing it to you.

Creating a view from a query requires to add `CREATE VIEW viewname AS`
before the query itself. For example, imagine that my project only covers
the data gathered for the first 10 gauges.  That
query would look like:

    SELECT *
    FROM raingauge_data
    WHERE raingauges_id <= 10;

But we don't want to have to type that every time we want to ask a
question about that particular subset of data. Hence, we can benefit from a view:

    CREATE VIEW gauges_10 AS
    SELECT *
    FROM raingauge_data
    WHERE raingauges_id <= 10;

You can also add a view using *Create View* in the *View* menu and see the
results in the *Views* tab, the same way as creating a table from the menu.

Using a view we will be able to access these results with a much shorter notation:

    SELECT *
    FROM gauges_10
    WHERE raingauges_id == 5;