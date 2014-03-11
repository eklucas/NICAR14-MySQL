NICAR14-MySQL
=============

For this class, we will be working with a subset of the National Highway Transportation Safety Adminstration (NHTSA) [Vehicle Recalls and Complaints database](http://ire.org/nicar/database-library/databases/nhtsa-vehicle-recalls-and-complaints/ "Vehicle Recalls and Complaints database"). You can download this data as an executable MySQL file from this public Dropbox folder [public Dropbox folder](https://www.dropbox.com/sh/eu3bniitjuwnlo2/5Qij3EceFy "public Dropbox folder").

Also, be aware that some of the queries we'll be writing will NOT work in Microsoft Access. Which is why we'll be working in MySQL.

## ADDing and UPDATEing columns

Most of the queries you've been running up to this point have been SELECT statements, the kind of SQL query that returns values from the columns currently in your database. While SELECT statements are incredibly useful, they are barely the tip of the iceberg in terms of what SQL can do. 

Source data is almost always 'dirty', as we often hear. Or it may just be in a format that's not suitable for answering the questions we have. In cases like this, you might want to improve the source data by adding additional columns and populating (updating) them with whatever values, in whatever format, you desire.

### Why not just fix the columns and values instead of adding new ones?

Yeah, you could do that. But remember: SQL doesn't come with an 'undo' button. Once you change your data, you can't simply hit Crtl + Z and go back to the previous version. And if just erronously change your only copy of the source data, you're totally hosed.

Also, best practice suggests you keep track of everything you are doing and be transparent about that process. So if you're showing your work to an editor or colleague, it helps to be able to say 'Here is what this column looked like before, and here is what it looks like now.'

Convinced? Good. Now let's get to it. 

Say we are wanting to mine NHTSA data for investigation records with an eye toward the problems that prompted these investigations. After consulting our Readme and Record Layout docs, we learn that there's one table of investigation records -- inv -- and one column on this table -- subject -- that describes what the investigation was intended to look into.

So here's a good first question: What are all of the subjects of these investigations and how frequently do each of them come up?

Though you've probably heard this before, it's worth repeating: Before you write any query, take a minute to think about what you want the results to look like. In this case, we want a list of all of the investigation subjects and, next to each subject, a count of the number of records with that subject. And since we are probably most interested in the most frequent subjects, we want those to appear at the top of the list.

This sh*t practically writes itself.

``` sql
SELECT subject, count(*)
FROM inv
GROUP BY 1
ORDER BY 2;
```

A quick note: You might be wondering about that '1' in the GROUP BY clause and that '2' in the ORDER BY clause. What you're seeing is just a shorthand for referencing the columns specified in the SELECT clause. We are grouping by subject (the first column) and ordering the records by the count (the second column) in descending order. These are called column aliasis, and we'll cover these in more depth a little further down.

If you check out the results, you'll notice is that the word 'ACCELERATION' shows up several times in the subjects, especially in those with high record counts. You'll also see that sometimes this word is abbreviated as 'ACCEL'.

Can you feel your inner reporter's innate sense of curiosity kicking in? Here's our next question: How many investigations had something to do with accelerations?

How do we SELECT records WHERE the a column a contains a value LIKE something you are interested in? 

Come on, we're practically giving you this one:

``` sql
SELECT count(*)
FROM inv
WHERE subject like '%accel%';
```

Easy, right?

Now that we have identified all of the investigations that have something to do with acceleration, we want to start treating these records as a distinct group. But we don't have any column with a common value that allows us to group them all together. So...let's add one! Here's the syntax for that

``` sql
ALTER TABLE inv ADD liz_sub varchar(255);
```

The ALTER TABLE command tells SQL we want to change a table -- 'inv' in this case -- and add a column we are calling 'liz_sub' and we are setting the data type to be varchar (essentially a string) with a maximum number of 255 characters (note: this is usually the default number of characters for varchar fields).

Now that we've made that column, let's put some data into it! In this case, we want to use the same value for all of these investigation records that include 'accel' in the subject field. And we'll include another set of criteria: Only models after the year 2000, which was approximately when throttle linkages became remote controlled (thanks, [Robert Benincasa](http://www.npr.org/templates/story/story.php?storyId=124276771 "Robert Benincasa")).

Here is what that query looks like:

``` sql
UPDATE inv 
SET liz_sub = 'ACCEL' 
WHERE subject like '%accel%' AND year > 2000;
```

Two new things we've introduced:
1. The UPDATE clause, which is where we specify the table ('inv') that contains the column we are updating
2. The SET clause, which is where we specify the column ('liz_sub') that we want to update.

Also, notice how the WHERE clause looks exactly like what we might see in a SELECT query. Before you run any UPDATE statement, best practice suggests you run the same query in the form of a SELECT statement so that you can see both the number and the exact records that are going to be affected, so you can double-check your filtering conditions. In this case, that would be: 

``` sql
SELECT *
FROM inv 
WHERE subject like '%accel%' AND year > 2000;
```

## Advanced SQL functions

Check out the 'odate' column which, according the Readme and Layout docs, contains the date the given investigation was opened. Super-useful information, right? Except the data type of this field is currently varchar, rather than date. So, for example, we couldn't compare this date each investigation opened to the date each investigation closed (the 'cdate' field) in order to get the duration of each investigation. Bummer.

But wait a sec...we already know how to add a column with whatever data type we want (date, in this case). So we just need to figure out how to convert varchar (aka, string) data into date data.

Would you believe me if I told you that MySQL has this capability already built into it? Would you believe me if I told you that it has lots of really awesome capabilities for manipulating and enhancing your data?

We're talking, of course, of functions. A function is pre-defined set of instructions that takes one or more inputs, called arguments, and returns your desired output value. Think of functions as your own personal little helpers: an army of well-trained, highly specialized intern elves who will do exactly what you want, but only if you ask them in exactly the manner they expect.

We've actually used a few SQL functions already. 'count()' is one of several aggregate functions built into SQL. Another one you've probably seen before is 'sum()', which is usually used to add up all of the numeric values in a column.

If any of this sounds familiar from your time working in spreadsheet applications, like Microsoft Excel, that's no coincidence. The functions you've used to manipulate and enhance data in Excel are very similar to those available in SQL. In fact, 'function' is term that's general to math and computer science, and if you go on to learn how to program in a language like Python or Javascript, you'll spend a lot of time writing and using functions.

And, yes, you can define you're own custom SQL functions to do your bidding. These are called (rather unimaginatively) 'user-defined functions', and they are way outside the scope of this course.

It's also important to note that the number and name of functions available will differ from one vendor's version of SQL to the next. [Here's the list of functions](http://dev.mysql.com/doc/refman/5.6/en/func-op-summary-ref.html "Here are the functions") available in MySQL version 5.6.

Alright, back to the issue at hand: We need to somehow transform the string data in the 'odate' field into dates. Luckily we've got MySQL's [str_to_date()](http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_str-to-date "str_to_date()") at our disposal. We just have to 'call' it with the right arguments. Again, think of a function as your specialized helper: You call the str_to_date() function, pass it the arguments it's expecting between the '()', and it will respond with the values you want.

There are two arguments for str_to_date(), and remember that the order in which they listed does matter. 

The first is 'str', which is the string you want to convert into a date. In this case, that's just the 'odate' column.

The other one is 'format', which is the current format of the value you want to convert into a date. This could be something like '20140228', '02/28/14', 'February 28, 2014'... etc. There are, of course, a lot of ways to write the same date, but the str_to_date() function won't know what format your data is currently in unless you tell it. 

Let's try each of these, starting with the first:

``` sql
SELECT STR_TO_DATE('20140228','%Y%m%d');
```

What you're seeing in the format (the second) argument are the [date format specifiers](http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_date-format "date format specifiers"), which is how we tell str_to_date() which of the string's characters represent the year ('%Y') versus the month (%m) versus the day(%d). And as you can see in the documentation, the case matters. '%Y' is a four-digit representation of a year, while '%y' is a two-digit one. 

Let's try the next one:

``` sql
SELECT STR_TO_DATE('02/28/14','%m/%d/%y');
```

Notice how this time we had to change more than just the order and case of the specifiers; we also had to account for the the delimiters (the '/') which exist in our current string's date format.

See if you can figure out how to do this last one ('February 28, 2014') on your own. Note that you will probably need to consult the [list of date format specifiers](http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_date-format "list of date format specifiers").

Give up? Okay, here's how:

``` sql
SELECT STR_TO_DATE('February 28, 2014','%M %d,%Y');
```

We just spent a lot of time figuring out how to use one function, which should emphasize the importance of consulting your database manager's documentation, all readily available to you at multiple stops somewhere on the information super-highway (am I the only one still using this phrase?)

Moving on... Take a look at the 'odate' field and figure the date format and how the specifers will need to be arranged. Then let's write more SQL!

But wait: Remember your best practices! We need to add a new column before we try to convert this data. Here's a pro-tip for all your troubles: We don't actually have to run the ALTER TABLE and UPDATE statements separately. We can run them at the same time as along as we separate the two statements with a ';'. Like this:

``` sql
ALTER TABLE inv ADD liz_odate date;

UPDATE inv SET liz_odate = STR_TO_DATE(odate, '%Y%m%d');
```

## Subqueries

Now let's take our queries to whole other level with subqueries. These are exactly what you're probably imagining: They're just queries that are contained within another SQL statement. And like a regular query, a subquery can return:
* A single value (one row and one column)
* A single column of many values
* Many rows and many columns, which as we will see is sort of like the equvalent a temporary table

What the subquery returns will determine how it can be incorporated into the containing query: in the SELECT clause, the FROM clause, the WHERE clause...wherever.

Before we give it a try, let's pose another question that could reveal some journalistic value: How many recalls resulted from investigations that spanned longer than a year?

The challenge for answering this question is that investigation and recall records are in two different tables -- 'inv' and 'recall' -- but we want to somehow filter the recall records based on criteria stored on the investigation records.

The two tables do share a common column: 'campno', which is the unique id of the recall campaign, so one way to do this is to join these two tables together. Let's give that try and see what happens:

``` sql
SELECT *
FROM recall
JOIN inv
ON recall.CAMPNO = inv.CAMPNO
WHERE datediff(inv.nicar_cdate, inv.nicar_odate) > 365;
```

We just got a lot more records back than we were expecting, several of which look like duplicates. Why is that? Well, it's because 'campno' is not a primary key on either of these tables. Actually there's a one-to-many relationship both between recall campaigns and investigations and between recall campaigns and recalls.

Before sort through that mess, there are a couple of other things worth noting about the last query.

First, we're using MySQL's built-in [datediff()](http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_datediff "datediff()") function, which takes two dates and returns the number of days between the two dates.

Second, we've switched syntax styles a bit. Since this query contains references to multiple tables in our database, rather than using column aliases, we're including specific table and column names, separated by '.' in the ON and WHERE clause. When you start to write more complicated queries, as we will be doing for the remainder of the course, you may find that this sort of explicitness makes the SQL more readable.

So how do we get exactly the recall records we want? We could write a join query that accounts for these one-to-many relationships, using a LEFT JOIN and/or DISTINCT clause among other things, but it isn't going to be very intuitive. So let's take a step back and think carefully about the results we are trying to get. What we want are recall records that have a 'campno' that's the same as the campno of any investigation that lasted over a year.

Well, we know how to get the 'campno' values of investigations that lasted over a year, so let's start by writing that SELECT statement:

``` sql
SELECT campno
FROM inv
WHERE datediff(inv.nicar_cdate, inv.nicar_odate) > 365;
```

If we run this query, what we get is one column with many rows. These are the 'campno' values of investigations that lasted more than a year, which we can use to filter our recall records. Now how do we make this into a subquery? Easy. Just put parentheses around it, and then add it the appropriate clause of a containing query. Again, in this case we want to SELECT records FROM recall WHERE the 'campno' value is within the set of 'campno' values returned by the subquery. Here is what the whole thing looks like together.

``` sql
SELECT *
FROM recall
WHERE campno IN (
	SELECT campno
	FROM inv
	WHERE datediff(inv.nicar_cdate, inv.nicar_odate) > 365
);
```

You might not have seen the IN operator before. That's just a way to tell SQL to look for a value that's within another set of values, which could either be returned by a subquery or listed out explicitly like this 'WHERE letter IN ('A', 'B', 'C'...)'.

Also, note that the indentation isn't necessary for the query to execute correctly. It's just good syntactical style, making the SQL more readable.

This process of writing the subqueries, checking the results and then incorporating them into containing queries is a pretty good pattern to follow as you start writing more complicated queries. Pro-tip: If you are trying to unravel a dense, complicated query written by someone else (maybe something you did yourself months ago), you can run just the subquery by highlighting it in your query window, and then hitting run.

## SELECTing FROM and JOINing to subqueries

One thing that subqueries are really useful for doing is combining aggregates result sets. For example, say we wanted to investigate the manufacturers that come up frequently in our NHTSA data delve. One basic question we might come up with is: How many complaints, investigations and recalls has each manufacturer had?

The problem is that these records are in three different tables: complaint, inv and recall. For each table, we need to GROUP BY whatever field contains the manufacturer (consult your Readme and Record Layout docs to get the exact column name) and COUNT the records for each manufacturer. We could run each query individually and copy and paste the results into Excel or some other spreadsheet. But...that's so...MANUALLY. Come on! You gotta make the computer do all the work!

Let's think about how we might do it in one query. The results we want are a list of manufacturers and three counts next to each manufacturer: the number of complaints, the number of investigations and the number of recalls.

We know how to get those counts in three separate queries, right? So let's start by writing those SELECT statements (don't forget those semi-colons):

``` sql
SELECT MFR_NAME, COUNT(*)
FROM comp
GROUP BY MFR_NAME;

SELECT MFR_NAME, COUNT(*)
FROM inv
GROUP BY MFR_NAME;

SELECT MFGTXT, COUNT(*)
from recall
GROUP BY MFGTXT;
```

All we need to do now is JOIN these results sets together, the same as if they were actual tables in our database. Here is what that looks like:

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

Remember back in some of our earlier queries when we were using column aliases (the '1' and '2' shorthand)? In the above query, we are actually specifying what we want our column aliases to be for each table's count(*) column. That's what the 'AS Count_Complaints', 'Count_Investigations' and 'AS Count_Recalls' stuff is doing. We could use any alias we want ('AS Num_Comps', 'AS foo_bar'...whatever). But again, it's good to be explicit so that someone else (maybe a future you) can read and understand your SQL.

And actually, these column aliases aren't necessary, but they just give our results a clear column header. 

However, the table aliases -- the 'AS Complaints', 'AS Investigations' and 'AS Recalls', which appear after the closing parenthesis of each subquery -- are necessary because we are joining to a results set, not an acutal database table with a defined name.

## Doing math in the SELECT clause

For the SELECT clause, we already knew how to reference specific columns and call functions, like count() and sum(). Another thing we can do is write our own formulas -- again, much like we can in Excel -- that can incorporate columns in our database, functions and also subqueries. This allows us to do useful calculations, like percent of whole and percent change, and perform operations on string values for improving the quality of our data.

Let's take a closer look as the NHTSA's Readme and Record Layout docs. You'll notice that on the 'recall' table there's a column named 'INFLUENCED_BY', which indicates which organization instigated the recall, either the manufacturer ('MFR'), the Office of Vehicle Safety Compliance ('OVSC') or the Office of Defect Investigation ('ODI').

So here's a new question: How often do manufacturers initiate recalls versus the two federal oversight offices? And what we want here is the percentage of the whole because, in cases like these, raw counts usually don't have as much impact, especially as the number of records you are dealing with grows.

So what do we want our results to look like? We want a list of all the possible values of 'INFLUENCED_BY', so we know we are going to SELECT and GROUP BY 'INFLUENCED_BY'. And next to each value we want the percentage of records that have value.

Remember the formula for percentage: The part over the whole all multipled by 100. Well, we already know that if we include 'count(*)' when we are SELECTing and GROUPing BY 'INFLUENCED_BY', our results will include the count of records that have each value. So we just need to divide 'count(*)' by the total number of records in the recall table and multiply all of that by 100.

There are two ways to do this: You could run a separate query to get the count of all the records in the recall table, and then hard-code that number into your formula. Or you could incorporate that query as a subquery in the SELECT clause's formula, like this:

``` sql
SELECT 
	Influenced_By, 
	COUNT(*) AS Count_Recalls,
	(COUNT(*) / (select COUNT(*) FROM recall)) * 100 AS Pct_of_Total
FROM recall
GROUP BY INFLUENCED_BY
ORDER BY COUNT(*) DESC;
```

So based on these results, about 70 percent of the time the manufacturer is instigating the recalls. Makes it seem like manufacturers are, on a whole, a pretty responsible group. We'll come back to this later.

## Calculating year-over-year percent change:
	
Now were are going to bring things together, combining the concepts of math in the SELECT clause and JOINing subquery results. Fair warning: There's a bit of curveball ahead, so stay sharp.

Here's a new question we have for investigating the NHTSA database: What is the year-to-year percentage change in the number of recalls and the number of potentially affected units?

What do want our results to look like? We want a list of the years and, for each year, we want a count of recalls, a sum of potentially affected units, and the percentage change in each of these numbers from the previous year.

In order to get a list of years, we'll use MySQL's built-in ['year()'](http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_year "year()") function which accepts a date argument and returns the year part of the given date. Now let's start by getting all of the years and, for each, the count of recalls and the sum of potentially affected units:

``` sql
SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
FROM recall
GROUP BY YEAR(nicar_odate)
```

Remember the formula for percent change: The difference between the new number and the old number, divided by the old number, all multiplied by 100, or ((N - O)/O) * 100.

So for each year in our results, in addition to the current count of recalls and sum of affected units, we need to somehow incorporate the previous year's count of recalls and sum of affected units. So how do we get the current year's numbers and the previous year's number on the same row of our results?

We actually already know how to get the list of previous years and their numbers because these are one-and-the-same with the list of current years and their numbers, which are returned by the previous query. So what we can do is run that same subquery to get these results twice, and then it's just a matter of how we line up the two sets of results, that is, how we JOIN them.

``` sql
SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
FROM recall
GROUP BY YEAR(nicar_odate) as current_year;

SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
FROM recall
GROUP BY YEAR(nicar_odate) as previous_year;
```

What column of values do these two results set have in common? It's year, of course, so that's the field we are going to join on. Let's go ahead and set that up, along with the columns and formulas in the SELECT clause of our containing query:

``` sql
SELECT 
	current_year.oYear, 
	current_year.Recall_Count, 
	previous_year.Recall_Count, 
	((current_year.Recall_Count - previous_year.Recall_Count) / previous_year.Recall_Count) * 100 AS Count_Recall_Pct_Change,
	current_year.Sum_Pot_Aff,
	previous_year.Sum_Pot_Aff, 
	((current_year.Sum_Pot_Aff - previous_year.Sum_Pot_Aff) / previous_year.Sum_Pot_Aff) * 100 AS Sum_Pot_Aff_Pct_Change
FROM (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS current_year
JOIN (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS previous_year
ON current_year.oYear = previous_year.oYear
ORDER BY current_year.oYear DESC;
```

If you run this query, the results you will get are the year, the count of recalls repeated twice, a 0 percent change in recalls, the sum of affected units repeated twice and a 0 percent change in affected units. This is because the first row has 2014's numbers next to 2014's numbers, and the next row has 2013's numbers next to 2013's numbers and so forth.

But we want 2013's numbers next to 2014's numbers, 2012's next to 2013's and so forth. So how can we trick SQL into putting 2014 and 2013 on the same row, 2013 and 2012 on the same row and so forth?

Think of it this way: What's the difference between 2014 and 2013? Obviously, the answer is 1 year, same as the difference between 2013 and 2012 and the same as any year and the year before it. So if we are going to trick SQL into thinking that 2014 and 2013 are actually the same, we need either add 1 to 2013 or subtract 1 from 2014. You'll do that in your JOIN clause like this:

``` sql
SELECT 
	current_year.oYear, 
	current_year.Recall_Count, 
	previous_year.Recall_Count, 
	((current_year.Recall_Count - previous_year.Recall_Count) / previous_year.Recall_Count) * 100 AS Count_Recall_Pct_Change,
	current_year.Sum_Pot_Aff,
	previous_year.Sum_Pot_Aff, 
	((current_year.Sum_Pot_Aff - previous_year.Sum_Pot_Aff) / previous_year.Sum_Pot_Aff) * 100 AS Sum_Pot_Aff_Pct_Change
FROM (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS current_year
JOIN (
	SELECT YEAR(nicar_odate) AS oYear, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate)
) AS previous_year
ON current_year.oYear = previous_year.oYear + 1
ORDER BY current_year.oYear DESC;
```

You might call it a hack. We prefer to call it an enterprise join.

If you wanted to drill into these changes a little further, you could break the results down by recall type ('RCLTYPECD') by adding this column to the two subqueries, the SELECT clause, the JOIN clause and the ORDER BY clause:

``` sql
SELECT 
	current_year.oYear, 
	current_year.RCLTYPECD,
	current_year.Recall_Count, 
	previous_year.Recall_Count, 
	((current_year.Recall_Count - previous_year.Recall_Count) / previous_year.Recall_Count) * 100 AS Count_Recall_Pct_Change,
	previous_year.Sum_Pot_Aff, 
	((current_year.Sum_Pot_Aff - previous_year.Sum_Pot_Aff) / previous_year.Sum_Pot_Aff) * 100 AS Sum_Pot_Aff_Pct_Change
FROM (
	SELECT YEAR(nicar_odate) AS oYear, RCLTYPECD, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate), RCLTYPECD
) AS current_year
JOIN (
	SELECT YEAR(nicar_odate) AS oYear, RCLTYPECD, COUNT(*) AS Recall_Count, SUM(PotAff) AS Sum_Pot_Aff
	FROM recall
	GROUP BY YEAR(nicar_odate), RCLTYPECD
) AS previous_year
ON current_year.oYear = previous_year.oYear + 1
AND current_year.RCLTYPECD = previous_year.RCLTYPECD
ORDER BY current_year.oYear DESC, RCLTYPECD;
```

## CASE expressions

For this last exercise, we are going to go back to one of the results of a previous query. Remember when we found out the manfacturers, as a whole, seem to be doing a really great job of initiating recalls for their own defective products? While that might be generally true of all the manufacturers, surely there are some manufacturers who aren't that forthcoming about their mistakes. Let's try to figure out who are the worst actors in this group.

Here's the specific question we want to answer: For each manufacturer what percent of the time do they initiate their own recalls? And let's also narrow this to only recalls where the consequences of the defect was really bad (e.g., involved a potential fire or wreck or explosion). Let's also narrow it to manufacturers that have had a significant number of recalls, say 50.

So what do we want the results to look like? It should be a list of manufacturers, a count of recalls for that manufacturer where the defect's consequences were really bad and the manufacturer initiated the recall, a count of all recalls for that manufacturer with really bad defects and the percent.

As is usually the case in SQL, there's more than one way to write this query. Like the previous query, we could use two subqueries -- one that returns counts of recalls for really bad defects where the manufacturer initiated the recall and another nearly identical query that includes counts for all recalls for really bad defects, regardless of the initiator. But because of the very narrowly defined filtering conditions, this is going to be a really long query with a lot of repeated SQL.

So here's a new approach: Use a [CASE expression](https://dev.mysql.com/doc/refman/5.0/en/case.html "CASE expression") which is allows you to incorporate conditional logic (if this, then that) into your SQL queries.

In order to get familiar with the syntax of CASE expressions, let's start by writing a much more simple SELECT statement. Let's just SELECT all recall records and, if the initiator or the recall was the manufacturer, then let's ask SQL to give us 1, otherwise we want a 0:

``` sql
SELECT
	MFGTXT,
	Influenced_By,
	CASE WHEN Influenced_By = 'MFR' THEN 1 ELSE 0 END
FROM recall;
```

As you can see, you open the expression with the term 'CASE' then you define your first condition using 'WHEN', in this case the condition is when the Influenced_By column is 'MFR'. Then you tell SQL what you want to happen for these conditions using 'THEN', which this case is give us a 1. Note that you can specify multiple conditions if that suits your needs. You can also tell SQL what you want to happen for any other condition you don't define by using 'ELSE', which in our case is to return 0. And you always close your CASE expression with 'END'.

Cool, so for every recall record, we've got a 1 or a 0 depending on whether or not the manufacturer intiated that recall. Now let's figure out all of the complicated filtering which will let us narrow to just the recall records for really bad defects. Actually, there's no scientific definition here. Rather, we just have to spend sometime thinking about what makes a defect really bad. One reasonable approach is to look in the 'CONSEQUENCE_DEFECT' column for words that really stand out, like 'Fire' or 'Crash' or 'Explosion'. Let's write a SELECT query that gets us just these results:

``` sql
SELECT 
	MFGTXT,
	Influenced_By,
	CASE WHEN Influenced_By = 'MFR' THEN 1 ELSE 0 END
FROM recall
WHERE CONSEQUENCE_DEFECT LIKE '%Fire%'
OR CONSEQUENCE_DEFECT LIKE '%Burn%'
OR CONSEQUENCE_DEFECT LIKE '%Crash%'
OR CONSEQUENCE_DEFECT LIKE '%Roll%'
OR CONSEQUENCE_DEFECT LIKE '%Explode%'
OR CONSEQUENCE_DEFECT LIKE '%Explosion%'
OR CONSEQUENCE_DEFECT LIKE '%Injury%'
OR CONSEQUENCE_DEFECT LIKE '%Wreck%';
```

We're getting close, but remmeber we want a list of manufacturers and their counts and percentages. So that's just a GROUP BY query, right? But also, we said we want to focus on only those manufacturers with a significant number of recalls (again, no scientific definition here, we're just going with 50). But this filter can't be part of the WHERE clause since it's based on the aggregate results rather than any of the column values of the table's individual rows. 

Can you remember which clause allows us to filter down to just aggregate results that have certain values?

``` sql
SELECT 
	MFGTXT,
	Count(*) AS Count_Bad_Recalls,
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
HAVING COUNT(*) > 50;
```

That's right. It's the HAVING clause, which can be used in conjunction with the GROUP BY clause.

Almost there. Now we just need to SUM the results of the CASE expression and specify the formula for percent of whole (again, part divided by whole all multiplied by 100). And let's order by the percentage, so that the worst actors appear at the top:

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

Can you see that? The first dozen or so worst actors are all manufacturers of R.V.s, campers and trailers. Maybe there's a story there.