---
layout: post
title:  "Using Dynamic SQL Queries to Change Column Headers to Rows"
date:   2021-02-05
image:  images/050221.jpg
tags:   [SQL]
---

<span>Photo by <a href="https://unsplash.com/@jeremythomasphoto?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Jeremy Thomas</a> on <a href="https://unsplash.com/s/photos/stars?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Although designed for Tableau Prep, I find Carl Allchin, Jenny Martin, Jonathan Allenby and Tom Prowse’s Preppin’ Data challenges really good for flexing SQL muscles. I decided I wanted to experiment a little with dynamic queries and [**2020 Week 53’s challenge**][pd-w53] provided nice ground to play around with.  

The essence of the challenge is that you have 3 inputs: old star sign date ranges, new star sign date ranges and a scaffold. Sound easy enough, except for the fact that the old date ranges come in a messy format, unlike the new ones they don’t have column headers!

![]({{site.baseurl}}/images/050221-1.PNG)

While it might be a slightly convoluted use case, it was a nice opportunity to play around. In this scenario, simply renaming the column headers would lose us 3 rows of data. Sure, we could just use a bit of copying and pasting, but what would be the fun in that?

```sql
DECLARE @Command1 varchar(1000)
SET @Command1 = 'SELECT [' + (SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 1) + '] AS Sign, [' +
(SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 2) + '] AS Date FROM OldStarSigns;'
DECLARE @T1 table (Sign varchar(20), Date varchar(20))
INSERT @T1 EXEC (@Command1)

DECLARE @Command2 varchar(1000)
SET @Command2 = 'SELECT [' + (SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 3) + '] AS Sign, [' +
(SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 4) + '] AS Date FROM OldStarSigns;'
DECLARE @T2 table (Sign varchar(20), Date varchar(20))
INSERT @T2 EXEC (@Command2)

DECLARE @Command3 varchar(1000)
SET @Command3 = 'SELECT [' + (SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 5) + '] AS Sign, [' +
(SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 6) + '] AS Date FROM OldStarSigns;'
DECLARE @T3 table (Sign varchar(20), Date varchar(20))
INSERT @T3 EXEC (@Command3);
```

Above is the query I ended up writing – convoluted, I know, but I’ll break it down in step by step.

```sql
DECLARE @Command1 varchar(1000)
```


I’d think of this as a statement of intention, I’m basically saying I’m creating a local variable, it’s going to be a string, and I expect I won’t need more than 1,000 characters.

```sql
SET @Command1 = 'SELECT [' + (SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 1) + '] AS Sign, [' +
(SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'OldStarSigns' AND ORDINAL_POSITION = 2) + '] AS Date FROM OldStarSigns;'
```


Here, I am creating the string itself, building a query that I want to run where I don’t know what the values might be. 

I start with SET @Command1 = because this is going to tell MS SQL Server I want to now add a string into the local variable I’ve just declared.

Then I start construction of the string.

SELECT [‘ (…) ‘] AS Sign, [‘ (…) ‘] AS Date FROM OldStarSigns;’ – this is a classic query, the interesting bit is what does inside the two (…)s.

```sql
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA WHERE TABLE_NAME = ‘OldStarSigns’ AND ORDINAL POSITION = 1
```


Here I am querying SQL Server’s Information Schema asking it to give me the name of the first column.

```sql
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA WHERE TABLE_NAME = ‘OldStarSigns’ AND ORDINAL POSITION = 2
```


And here I’m asking for it to give me the name of the second column.

The end product of this is a string that reads “SELECT [Aquarius] AS Sign, [1/20–2/19] AS Date FROM OldStarSigns;”

```sql
DECLARE @T1 table (Sign varchar(20), Date varchar(20))
INSERT @T1 EXEC (@Command1)
```


Once written I then create a table variable and insert into that table an execution of the query I built above. 

![]({{site.baseurl}}/images/050221-QueryResult.PNG)


This is repeated another two times, and the results are then combined in a CTE

```sql
WITH DateSigns2 AS (
				SELECT * FROM @T1
				UNION
				SELECT * FROM @T2
				UNION
				SELECT * FROM @T3
				),
```


Producing the following: 

![]({{site.baseurl}}/images/050221-DateSigns2.PNG)

After that, I summoned all the dates

```sql
StarDates1 AS (
				SELECT COLUMN_NAME AS Dates, ORDINAL_POSITION - 1 AS Pos
				FROM INFORMATION_SCHEMA.COLUMNS
				WHERE TABLE_NAME = 'OldStarSigns' AND COLUMN_NAME LIKE '%[^a-zA-Z]%'),
```


Then summoned all the star signs

```sql
StarSigns1 AS (
				SELECT COLUMN_NAME AS Sign, ORDINAL_POSITION AS Pos
				FROM INFORMATION_SCHEMA.COLUMNS
				WHERE TABLE_NAME = 'OldStarSigns' AND COLUMN_NAME NOT LIKE '%[^a-zA-Z]%'),
```


And after that, joined them together (notice how I used the ORIGINAL_POSITION -1 for even numbers so that they would correspond to the odd numbers in a join).

```sql
DateSigns1 AS (
				SELECT Sign, Dates
				FROM StarDates1 a
				INNER JOIN StarSigns1 b ON a.Pos = b.Pos),
```


The end result ended up looking like this

![]({{site.baseurl}}/images/050221-QueryResult.PNG)

Which could then be unioned with my DateSigns2 table (this was all the star signs that weren’t column headers)

```sql
DateSigns AS (
				SELECT *
				FROM DateSigns1
				UNION
				SELECT*
				FROM DateSigns2
				),
```


Producing an output like this:

![]({{site.baseurl}}/images/050221-DateSigns.PNG)

While this was by no means the most efficient way of solving this problem, but that wasn’t the intention of the exercise, sometimes it’s useful to go about things the hard way in order to learn a little along the way. Reflecting on this methodology, the one bit of room for improvement would be the fact that I’ve hardcoded in the column positions – if I wanted to *really* dynamic, I need a way of creating a query that could handle any number of columns.


P.S. Another trick I learnt in the solving of this challenge is that if you want to add a new row ID that mirrors the row index on the left hand side, you can do so by adding the following:

```sql
ROW_NUMBER() OVER (ORDER BY (SELECT 100)) AS Row_ID
```



[pd-w53]:https://preppindata.blogspot.com/2020/12/2020-week-53.html
