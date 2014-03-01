NICAR14-MySQL
=============

Data:

https://www.dropbox.com/sh/eu3bniitjuwnlo2/5Qij3EceFy


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
*471 records*

## another example is adding columns for cleaning purposes.

``` sql
ALTER TABLE inv ADD liz_odate date;
;
UPDATE inv SET liz_odate = str_to_date(odate, '%Y%m%d')
```

## How many recalls resulted from investigations that spanned longer than a year?
## demonstrates why a joining recalls and inv on CAMPNO is not ideal in this situation because of how the records are multiplied
``` sql
SELECT *
FROM recall
JOIN inv
ON recall.CAMPNO = inv.CAMPNO
WHERE DATEDIFF(inv.nicar_cdate, inv.nicar_odate) > 365;
```

## so use a sub-query instead
## demonstrates sub-query in the where clause

``` sql
SELECT *
FROM recall
WHERE CAMPNO IN (
	SELECT CAMPNO
	FROM inv
	WHERE DATEDIFF(nicar_cdate, nicar_odate) > 365
);
```

## How many complaints, investigations and recalls has each manufacturer had?
## demonstrates using sub-queries to join aggregate results

``` sql
SELECT 
	Complaints.MFR_NAME, 
	Complaints.Count_Complaints, 
	Investigations.Count_Investigations, 
	Recalls.Count_Recalls
FROM (
	SELECT MFR_NAME, COUNT(*) AS Count_Complaints
	FROM comp
	GROUP BY MFR_NAME
) AS Complaints
JOIN (
	SELECT MFR_NAME, COUNT(*) AS Count_Investigations
	FROM inv
	GROUP BY MFR_NAME
) AS Investigations
ON Complaints.MFR_NAME = Investigations.MFR_NAME
JOIN (
	SELECT MFGTXT, COUNT(*) AS Count_Recalls
	from recall
	GROUP BY MFGTXT
) AS Recalls
ON Complaints.MFR_NAME = Recalls.MFGTXT;
```

## How often do manufacturers initiate recalls versus the 
