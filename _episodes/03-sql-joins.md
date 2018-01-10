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

For example, what if we wanted information on when individuals of each
species were captured, but instead of their species ID we wanted their
actual species names.

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
> - Write a query that returns the UT, data, ward name, and gauge name
> >
> > SELECT raingauge\_data.UT,raingauge\_data.data, raingauges.name, wards.Ward
    FROM raingauge\_data    
    JOIN raingauges, wards
    ON raingauge\_data.raingauges\_id = raingauges.id AND raingauges.ward\_id=wards.ID;
>  {:.answer}
> 
{: .challenge}

### Different join types

We can count the number of records returned by our original join query.

    SELECT COUNT(*)
    FROM raingauge_data
    JOIN raingauges
    USING (id);

Notice that this number is smaller than the number of records present in the
survey data.

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
>> SELECT wards.Ward, COUNT(raingauges.name)
    FROM raingauges
    JOIN wards
    ON raingauges.ward_id = wards.id
    GROUP BY wards.Ward 
    ORDER BY wards.Ward;
> {:.answer}
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

<!--
We can represent unknown sexes with `'U'` instead of `NULL`:

    SELECT species_id, sex, IFNULL(sex, 'U')
    FROM surveys;

The lone "sex" column is only included in the query above to illustrate where
`IFNULL` has changed values; this isn't a usage requirement.

> ## Challenge:
>
> - Write a query that returns 30 instead of `NULL` for values in the
> `hindfoot_length` column.
{: .challenge}

> ## Challenge:
>
> - Write a query that calculates the average hind-foot length of each species,
> assuming that unknown lengths are 30 (as above).
{: .challenge}
-->
`IFNULL` can be particularly useful in `JOIN`. 
<!--When joining the `species` and
`surveys` tables earlier, some results were excluded because the `species_id`
was `NULL` in the surveys table. We can use `IFNULL` to include them again, re-writing the `NULL` to
a valid joining value:

    SELECT surveys.year, surveys.month, surveys.day, species.genus, species.species
    FROM surveys
    JOIN species
    ON IFNULL(surveys.species_id,'AB') = species.species_id;

> ## Challenge:
>
> - Write a query that returns the number of genus of the animals caught in each
> plot, using `IFNULL` to assume that unknown species are all of the genus
> "Rodent".
{: .challenge}
-->
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

<!--
> ## Challenge:
>
> Write a query that returns genus names, sorted from longest genus name down
> to shortest.
{: .challenge}

As we saw before, aliases make things clearer, and are especially useful when joining tables.

    SELECT surv.year AS yr, surv.month AS mo, surv.day AS day, sp.genus AS gen, sp.species AS sp
    FROM surveys AS surv
    JOIN species AS sp
    ON surv.species_id = sp.species_id;
    
To practice we have some optional challenges for you.

> ## Challenge (optional):
>
> SQL queries help us *ask* specific *questions* which we want to answer about our data. The real skill with SQL is to know how to translate our scientific questions into a sensible SQL query (and subsequently visualize and interpret our results).
>
> Have a look at the following questions; these questions are written in plain English. Can you translate them to *SQL queries* and give a suitable answer?  
> 
> 1. How many plots from each type are there?  
> 
> 2. How many specimens are of each sex are there for each year?  
> 
> 3. How many specimens of each species were captured in each type of plot?  
> 
> 4. What is the average weight of each taxa?  
> 
> 5. What is the percentage of each species in each taxa?  
> 
> 6. What are the minimum, maximum and average weight for each species of Rodent?  
>
> 7. What is the average hindfoot length for male and female rodent of each species? Is there a Male / Female difference?  
> 
> 8. What is the average weight of each rodent species over the course of the years? Is there any noticeable trend for any of the species?  
>
> > ## Proposed solutions:
> >
> > 1. Solution: `SELECT plot_type, count(*) AS num_plots  FROM plots  GROUP BY plot_type  ORDER BY num_plots DESC`
> >
> > 2. Solution: `SELECT year, sex, count(*) AS num_animal  FROM surveys  WHERE sex IS NOT null  GROUP BY sex, year`
> >
> > 3. Solution: `SELECT species_id, plot_type, count(*) FROM surveys JOIN plots ON surveys.plot_id=plots.plot_id WHERE species_id IS NOT null GROUP BY species_id, plot_type`
> >
> > 4. Solution: `SELECT taxa, AVG(weight) FROM surveys JOIN species ON species.species_id=surveys.species_id GROUP BY taxa`
> >
> > 5. Solution: `SELECT taxa, 100.0*count(*)/(SELECT count(*) FROM surveys) FROM surveys JOIN species ON surveys.species_id=species.species_id GROUP BY taxa`
> >
> > 6. Solution: `SELECT surveys.species_id, MIN(weight) as min_weight, MAX(weight) as max_weight, AVG(weight) as mean_weight FROM surveys JOIN species ON surveys.species_id=species.species_id WHERE taxa = 'Rodent' GROUP BY surveys.species_id`
> >
> > 7. Solution: `SELECT surveys.species_id, sex, AVG(hindfoot_length) as mean_foot_length  FROM surveys JOIN species ON surveys.species_id=species.species_id WHERE taxa = 'Rodent' AND sex IS NOT NULL GROUP BY surveys.species_id, sex`
> >
> > 8. Solution: `SELECT surveys.species_id, year, AVG(weight) as mean_weight FROM surveys JOIN species ON surveys.species_id=species.species_id WHERE taxa = 'Rodent' GROUP BY surveys.species_id, year`
> {: .solution}
{: .challenge}
-->