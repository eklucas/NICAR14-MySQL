NICAR14-MySQL
=============

Queries for Advanced SQL:

## look for records in INV that have to do with acceleration

``` sql
SELECT subject, count(*)
FROM inv
GROUP BY 1
ORDER BY 1;
```

*note that the word acceleration is often abreviated to "accel"*

## find the number of records that have the word part "accel"

``` sql
SELECT count(*)
FROM inv
WHERE subject like '%accel%';
```

*856 records*

## create a new_subject field to hold a value that we can quickly use. 

``` sql
ALTER TABLE inv ADD liz_sub varchar(255);
```

## populate the field with a word, here "accel", that is a marker for the records we're interested in. I'll include only models after the year 2000, which was approximately when throttle linkages became remote controlled -- thanks Robert Benincasa. 

``` sql
UPDATE inv SET liz_sub = 'ACCEL' WHERE subject like '%accel%' AND year > 2000;
```




