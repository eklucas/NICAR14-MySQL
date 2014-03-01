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
*471 records*

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

## How often do manufacturers initiate recalls versus the two federal oversight offices?
## demonstrates simple math in SELECT
``` sql
SELECT 
	Influenced_By, 
	COUNT(*) AS Count_Recalls,
	(COUNT(*) / (select COUNT(*) FROM recall)) * 100 AS Pct_of_Total
FROM recall
GROUP BY INFLUENCED_BY
ORDER BY COUNT(*) DESC;
```

## here is a query that returns the percentage change in the number of recalls and the sum or potentially affected units from one year to the next

## this actually reveals an interesting fact: the yearly number of recalls had declined for four straight year and then suddenly jumped 33 percent in 2013, one of the biggest jumps in 20 years

## this query demonstrates: 1) doing a sub-queries, 2) doing simple math in the SELECT clause, 3) doing an enterprise 'non-equi' join
``` sql
SELECT 
	current_year.oYear, 
	current_year.Recall_Count, 
	last_year.Recall_Count, 
	((current_year.Recall_Count - last_year.Recall_Count) / last_year.Recall_Count) * 100 AS Count_Recall_Pct_Change,
	current_year.Sum_Pot_Aff,
	last_year.Sum_Pot_Aff, 
	((current_year.Sum_Pot_Aff - last_year.Sum_Pot_Aff) / last_year.Sum_Pot_Aff) * 100 AS Sum_Pot_Aff_Pct_Change
FROM (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS current_year
LEFT JOIN (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS last_year
ON current_year.oYear = last_year.oYear + 1
ORDER BY current_year.oYear DESC;
```
## drill into these changes a little more by breaking down the recall counts by recall type

``` sql
SELECT 
	current_year.oYear, 
	current_year.RCLTYPECD,
	current_year.Recall_Count, 
	last_year.Recall_Count, 
	((current_year.Recall_Count - last_year.Recall_Count) / last_year.Recall_Count) * 100 AS Count_Recall_Pct_Change,
	last_year.Sum_Pot_Aff, 
	((current_year.Sum_Pot_Aff - last_year.Sum_Pot_Aff) / last_year.Sum_Pot_Aff) * 100 AS Sum_Pot_Aff_Pct_Change
FROM (
	SELECT YEAR(nicar_odate) AS oYear, RCLTYPECD, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate), RCLTYPECD
) AS current_year
LEFT JOIN (
	SELECT YEAR(nicar_odate) AS oYear, RCLTYPECD, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate), RCLTYPECD
) AS last_year
ON current_year.oYear = last_year.oYear + 1
AND current_year.RCLTYPECD = last_year.RCLTYPECD
ORDER BY current_year.oYear DESC, RCLTYPECD;
```

## For each manufacturer what percent of the time do they initiate their own recalls? 
## And let's also narrow this to only recalls where the consequence of the recall was really bad (e.g., involved a potential fire or wreck or explosion)
## And manufacturers that have had more than 50 recalls
## long way to do this
``` sql
SELECT 
	all_recalls.MFGTXT,
	all_recalls.Count_Recalls,
	MFR_recalls.Count_Recalls,
	(MFR_recalls.Count_Recalls / all_recalls.Count_Recalls) * 100 as Pct_MFR_Recalls
FROM (
	SELECT MFGTXT, COUNT(*) as Count_Recalls
	FROM recall
	WHERE CONSEQUENCE_DEFECT LIKE '%Fire%'
	OR CONSEQUENCE_DEFECT LIKE '%Burn%'
	OR CONSEQUENCE_DEFECT LIKE '%Crash%'
	OR CONSEQUENCE_DEFECT LIKE '%Roll%'
	OR CONSEQUENCE_DEFECT LIKE '%Explode%'
	OR CONSEQUENCE_DEFECT LIKE '%Explosion%'
	OR CONSEQUENCE_DEFECT LIKE '%Injury%'
	OR CONSEQUENCE_DEFECT LIKE '%Wreck%'
	GROUP BY MFGTXT
) as all_recalls
LEFT JOIN (
	SELECT MFGTXT, COUNT(*) as Count_Recalls
	FROM recall
	WHERE (CONSEQUENCE_DEFECT LIKE '%Fire%'
	OR CONSEQUENCE_DEFECT LIKE '%Burn%'
	OR CONSEQUENCE_DEFECT LIKE '%Crash%'
	OR CONSEQUENCE_DEFECT LIKE '%Roll%'
	OR CONSEQUENCE_DEFECT LIKE '%Explode%'
	OR CONSEQUENCE_DEFECT LIKE '%Explosion%'
	OR CONSEQUENCE_DEFECT LIKE '%Injury%'
	OR CONSEQUENCE_DEFECT LIKE '%Wreck%')
	AND Influenced_By = 'MFR'
	GROUP BY MFGTXT
) AS MFR_recalls
ON all_recalls.MFGTXT = MFR_recalls.MFGTXT
HAVING all_recalls.Count_Recalls > 50
ORDER BY Pct_MFR_Recalls;
```
## short way
``` sql
SELECT 
	MFGTXT,
	Count(*) AS Count_Bad_Recalls,
	SUM(CASE WHEN Influenced_By = 'MFR' THEN 1 ELSE 0 END) AS Count_Recalls_By_Mfr,
	(SUM(CASE WHEN Influenced_By = 'MFR' THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS Pct_of_Recalls_By_Mfr
FROM recall
WHERE CONSEQUENCE_DEFECT LIKE '%Fire%'
OR CONSEQUENCE_DEFECT LIKE '%Burn%'
OR CONSEQUENCE_DEFECT LIKE '%Crash%'
OR CONSEQUENCE_DEFECT LIKE '%Roll%'
OR CONSEQUENCE_DEFECT LIKE '%Explode%'
OR CONSEQUENCE_DEFECT LIKE '%Explosion%'
OR CONSEQUENCE_DEFECT LIKE '%Injury%'
OR CONSEQUENCE_DEFECT LIKE '%Wreck%'
GROUP BY MFGTXT
HAVING COUNT(*) > 50
ORDER BY Pct_of_Recalls_By_Mfr;
```


