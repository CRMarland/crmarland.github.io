---
layout: post
title: "Automating Table Creation from an External Stage in Snowflake"
date: 2021-06-28
image: images/280621-cover.jpg
---

Photo by <a href="https://unsplash.com/@christopher__burns?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Christopher Burns</a> on <a href="https://unsplash.com/s/photos/tech?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

Recently while I was readying to import some training data (I think [Tableau's Bookshop dataset](https://help.tableau.com/current/pro/desktop/en-us/bookshop_data.htm) they introduced alongside the relationship model is an incredibly good dataset) into Snowflake, I found myself hit by that sort of creative laziness. I really didn't want to write out a mountain of CREATE TABLE statements. Instead I decided an automated solution would be more fruitful. 

For those who have Alteryx, I've created an Alteryx App. For those who don't, I've created a short Python script. Both assume you're using an external S3 stage with CSVs inside, if that's not you then it shouldn't be too hard to adapt them if you know how to use Python or Alteryx.

Before I introduce them both - let's talk about what enables these automations. 

Snowflake allows users to [query data in stages](https://docs.snowflake.com/en/user-guide/querying-stage.html) (i.e. before they've been loaded into Snowflake). Let's look at an example from the link. 

```sql
select t.$1, t.$2 from @mystage1 (file_format => 'myformat', pattern=>'.*data.*[.]csv.gz') t;
```
<br/>
A few things are happening above:

1. Instead of SELECTing column headers, we're SELECTing $1 and $2 - these are positional arguments, because that metadata isn't loaded Snowflake doesn't know column 1 as 'author' it simply knows it as column 1. 


2. You'll notice that they're not just $1 and $2, they're t.$1 and t.$2 - what does this mean? Well, you can see right at the end we've aliased our @mystage1 as t. Therefore, t.$1 is short for @mystage1 (file_format => 'myformat', pattern=>'.*data.*[.]csv.gz').$1, a less clunky way of writing a query.


3. By SELECTing FROM the entire stage we're potentially creating a very time-consuming and misdirected query. For that reason we specify two things in the brackets after the FROM - the file format we expect the file to be in, allowing Snowflake to parse data from the file, and a pattern. The pattern uses RegEx to figure out which specific file it's going to read from within the stage. 

What you might have guessed by now is that in order to create a table from files in an external stage is going to require some understanding of the file schema being imported - you're going to need to know how many columns there are and what they're called (Snowflake is pretty good at getting data types right, so that's less of a concern). Potentially, writing out all these CREATE TABLE statements is going to take a while. Hence, why I decided to automate the process.


### Alteryx:
<iframe src="https://onedrive.live.com/embed?cid=031EFAC83F985321&resid=31EFAC83F985321%21502&authkey=AF3gvD7QrXO_rf4" width="98" height="120" frameborder="0" scrolling="no"></iframe>

### Python:
<script src="https://gist.github.com/CRMarland/0bb3d42e05384526e7c6582d083b1746.js"></script>

<br/>

Both of these essentially need you to download the staged files to a local folder (if they're big, I'd suggest taking cutting them down, you only really need the column headers) - what the app/script does is take the column header and position and use them to generate a SQL statement like the one below:

```sql
CREATE OR REPLACE TABLE DEMO.BOOKSHOP.AUTHOR AS
SELECT t.$1 AS AuthID,
       t.$2 AS First_Name,
       t.$3 AS Last_Name,
       t.$4 AS Birthday,
       t.$5 AS Country_of_Residence,
       t.$6 AS Hrs_Writing_per_Day
FROM @S3_DATATRASNFERTEST (file_format=> CSV_PIPE, pattern=>".*Author.csv") t;
```
<br/>

I hope this works for everyone, if not, please get in touch on Twitter or LinkedIn with any bugs and I'll try and fix. Happy Snowflaking!