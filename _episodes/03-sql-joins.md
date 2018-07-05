---
title: "Joins"
teaching: 15
exercises: 10
questions:
- "How do I bring data together from separate tables?"
objectives:
- "Employ joins to combine data from two tables."
- "Apply functions to manipulate individual values."
- "Employ aliases to assign new names to tables and columns in a query."
keypoints:
- "Use the `JOIN` command to combine data from two tables---the `ON` or `USING` keywords specify which columns link the tables."
- "Regular `JOIN` returns only matching rows. Other join commands provide different behavior, e.g., `LEFT JOIN` retains all rows of the table on the left side of the command."
- "`IFNULL` allows you to specify a value to use in place of `NULL`, which can help in joins"
- "`NULLIF` can be used to replace certain values with `NULL` in results"
- "Many other functions like `IFNULL` and `NULLIF` can operate on individual values."
---

## Joins

To combine data from two tables we use the SQL `JOIN` command, which comes after
the `FROM` command.

The `JOIN` command on its own will result in a cross product, where each row in
the first table is paired with each row in the second table. Usually this is not
what is desired when combining two tables with data that is related in some way.

For that, we need to tell the computer which columns provide the link between the two
tables using the word `ON`.  What we want is to join the data with the same
raingauge id.

    SELECT *
    FROM raingauge_data
    JOIN raingauges
    ON raingauge_data.raingauges_id = raingauges.id;

`ON` is like `WHERE`, it filters things out according to a test condition.  We use
the `table.colname` format to tell the manager what column in which table we are
referring to.

The output of the `JOIN` command will have columns from the first table plus the
columns from the second table. For the above command, the output will be a table
that has the following column names:

| ID | TR | UT | data | raingauges\_id | update\_ref | invalid | hours\_surrounding\_total | id | name | location_x | location_y | region\_id | reference | ward\_id |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|...|||||||||||||||
| 3| '17/12/04 10:51' | 1512227700 | 0.2 | 1 | '1512227700' | 0| 1.8 | 1| 'BLUFF RES NO.3' | 31.005075 | -29.933965 |1 | 'bluff3' | 66|
|...|||||||||||||||


Alternatively, we can use the word `USING`, as a short-hand. `USING` only 
works on columns which share the same name. 

We often won't want all of the fields from both tables, so anywhere we would
have used a field name in a non-join query, we can use `table.colname`.

For example, what if we wanted information on rainfall per raingauge, but instead of the raingauge IDs we wanted the
actual raingauge names.

    SELECT raingauge_data.UT,raingauge_data.data, raingauges.name
    FROM raingauge_data
    JOIN raingauges
    ON raingauge_data.raingauges_id = raingauges.id;

| UT | data | name | 
|---|---|---|
| ... |||
| 1512210300 | 0.2 | 'BALLITO' |
|...|||

Many databases, including SQLite, also support a join through the `WHERE` clause of a query.  
For example, you may see the query above written without an explicit JOIN.

	SELECT raingauge_data.UT,raingauge_data.data, raingauges.name
    FROM raingauge_data
    ON raingauge_data.raingauges_id = raingauges.id;

For the remainder of this lesson, we'll stick with the explicit use of the `JOIN` keyword for 
joining tables in SQL.  

> ## Challenge:
>
> - Write a query that returns the UT, data, ward name, and raingauge name
> 
>>
>> `SELECT raingauge_data.UT,raingauge_data.data, raingauges.name, wards.Ward
    FROM raingauge_data    
    JOIN raingauges, wards
    ON raingauge_data.raingauges_id = raingauges.id AND raingauges.ward_id=wards.ID;`
>>
>  {: .solution}
{: .challenge}

### Different join types

We can count the number of records returned by our original join query.

    SELECT COUNT(*)
    FROM raingauge_data
    JOIN raingauges
    USING (id);

Notice that this number is smaller than the number of records present in the raingauge data.

    SELECT COUNT(*) FROM raingauge_data;

This is because, by default, SQL only returns records where the joining value
is present in the joined columns of both tables (i.e. it takes the _intersection_
of the two join columns). This joining behaviour is known as an `INNER JOIN`.
In fact the `JOIN` command is simply shorthand for `INNER JOIN` and the two
terms can be used interchangably as they will produce the same result.

We can also tell the computer that we wish to keep all the records in the first
table by using the command `LEFT OUTER JOIN`, or `LEFT JOIN` for short.

> ## Challenge:
>
> - Re-write the original query to keep all the entries present in the `raingauge_data`
> table. How many records are returned by this query?
{: .challenge}

<!--
> ## Challenge:
> - Count the number of records in the `surveys` table that have a `NULL` value
> in the `species_id` column.
{: .challenge}

Remember: In SQL a `NULL` value in one table can never be joined to a `NULL` value in a
second table because `NULL` is not equal to anything, not even itself. 
-->
### Combining joins with sorting and aggregation

Joins can be combined with sorting, filtering, and aggregation. So, if we
wanted average rainfall of the gauges in each ward, we
could do something like

    SELECT wards.Ward,raingauges.name, AVG(raingauge_data.data)
    FROM raingauge_data
    JOIN raingauges,wards
    ON raingauge_data.raingauges_id = raingauges.id AND raingauges.ward_id = wards.id
    GROUP BY wards.Ward DESC;

> ## Challenge:
>
> - Write a query that returns the number of raingauges in each ward in alphabetical order.
>
>> `SELECT wards.Ward, COUNT(raingauges.name)
    FROM raingauges
    JOIN wards
    ON raingauges.ward_id = wards.id
    GROUP BY wards.Ward 
    ORDER BY wards.Ward;`
> {:.solution}
{: .challenge}

<!--
> ## Challenge:
>
> - Write a query that finds the average weight of each rodent species (i.e., only include species with Rodent in the taxa field).
{: .challenge}
-->
## Functions `IFNULL` and `NULLIF` and more

SQL includes numerous functions for manipulating data. You've already seen some
of these being used for aggregation (`SUM` and `COUNT`) but there are functions
that operate on individual values as well. Probably the most important of these
are `IFNULL` and `NULLIF`. `IFNULL` allows us to specify a value to use in
place of `NULL`.

The inverse of `IFNULL` is `NULLIF`. This returns `NULL` if the first argument
is equal to the second argument. If the two are not equal, the first argument
is returned. This is useful for "nulling out" specific values.

We can "null out" raingauge 7:

    SELECT raingauges_id, data, NULLIF(raingauges_id, 7)
    FROM raingauge_data
    ORDER BY raingauges_id;

Some more functions which are common to SQL databases are listed in the table
below:

| Function                     | Description                                                                                     |
|------------------------------|-------------------------------------------------------------------------------------------------|
| `ABS(n)`                     | Returns the absolute (positive) value of the numeric expression *n*                             |
| `LENGTH(s)`                  | Returns the length of the string expression *s*                                                 |
| `LOWER(s)`                   | Returns the string expression *s* converted to lowercase                                        |
| `NULLIF(x, y)`               | Returns NULL if *x* is equal to *y*, otherwise returns *x*                                      |
| `ROUND(n)` or `ROUND(n, x)`  | Returns the numeric expression *n* rounded to *x* digits after the decimal point (0 by default) |
| `TRIM(s)`                    | Returns the string expression *s* without leading and trailing whitespace characters            |
| `UPPER(s)`                   | Returns the string expression *s* converted to uppercase                                        |

Finally, some useful functions which are particular to SQLite are listed in the
table below:

| Function                            | Description                                                                                                                                                                    |
|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `IFNULL(x, y)`                      | Returns *x* if it is non-NULL, otherwise returns *y*                                                                                                                           |
| `RANDOM()`                          | Returns a random integer between -9223372036854775808 and +9223372036854775807.                                                                                                |
| `REPLACE(s, f, r)`                  | Returns the string expression *s* in which every occurrence of *f* has been replaced with *r*                                                                                  |
| `SUBSTR(s, x, y)` or `SUBSTR(s, x)` | Returns the portion of the string expression *s* starting at the character position *x* (leftmost position is 1), *y* characters long (or to the end of *s* if *y* is omitted) |