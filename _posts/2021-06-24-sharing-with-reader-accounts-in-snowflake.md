---
layout: post
title: "Sharing with Reader Accounts in Snowflake"
date: 2021-06-24
image: images/240621readers.jpg
---

Image by <a href="https://pixabay.com/users/stocksnap-894430/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=2594786">StockSnap</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=2594786">Pixabay</a>

Recently, I was challenged to give a presentation on three really cool features of Snowflake. It didn't take me long to know what the team would be most interested in, I chose:

1. Snowpipes
2. Time Travel
3. Data Sharing with Reader Accounts

But no presentation is any good without a demo, and I quickly realised that I had never actually set up a share, or worked with reader accounts. To the documentation I went. Only, I came to the opinion that the documentation is a little convoluted on the how-to with this one. Never mind, after a bit of studying and chatting with my manager, we got to the bottom of how to create a share along with reader accounts to read that share. Yet, for the benefit of others, I'd like to produce a blog with a bit more of a step-by-step account of how to share data with reader accounts in Snowflake.

## STEP 1: CREATING STUFF TO SHARE

OK, I suppose this is obvious, but you'll need to create something you can actually share. As part of this demo, I wanted to create a nasty-looking, convoluted query that would send shivers down the spine of anyone looking at it. For the sake of my reputation, I must say I would never actually write something like this I intent to use in production - quintuple-embedded sub-queries are nasty, and I much prefer using CTEs, but this was more about theatre than good coding.

```sql
USE SCHEMA DEMO.BOOKSHOP;

CREATE OR REPLACE SECURE VIEW SALES_PER_AWARD AS

    SELECT *, AWARDS_WON/AMOUNT_SOLD AS SOLD_PER_AWARD
        FROM (
            SELECT i.BOOKID, i.TITLE, i.AUTHID, i.AUTHOR_NAME, i.SERIESID, i.AMOUNT_SOLD,
                   COUNT(j.Title) AS AWARDS_WON
            FROM (SELECT g.BOOKID, g.TITLE, g.AUTHID, g.AUTHOR_NAME, g.SERIESID,
                         COUNT(h.ItemID) AS AMOUNT_SOLD
                FROM (SELECT e.BOOKID, e.TITLE, e.AUTHID, e.AUTHOR_NAME, e.SERIESID, f.ISBN
                    FROM (SELECT c.BOOKID, c.TITLE, c.AUTHID, d.FIRST_NAME || ' ' ||
                                 d.LAST_NAME AS AUTHOR_NAME, SERIESID
                    FROM  (SELECT BOOKID, TITLE, AUTHID, SERIESID
                           FROM BOOK a
                           JOIN INFO b ON a.BookID = b.BookID1 || b.BookID2) c
                    JOIN AUTHOR d ON c.AUTHID = d.AUTHID) e
                JOIN EDITION f ON e.BOOKID = f.BOOKID) g
            JOIN SALES h ON g.ISBN = h.ISBN
            GROUP BY g.BOOKID, g.TITLE, g.AUTHID, g.AUTHOR_NAME, g.SERIESID) i
        JOIN AWARD j ON i.Title = j.Title
        GROUP BY i.BOOKID, i.TITLE, i.AUTHID, i.AUTHOR_NAME, i.SERIESID, i.AMOUNT_SOLD);


SELECT *
FROM SALES_PER_AWARD;
```

<br />

What this produced was a secure view based on [Tableau's bookshop data](https://help.tableau.com/current/pro/desktop/en-us/bookshop_data.htm) looking like this:

![]({{site.baseurl}}/images/230621_result_1.PNG)

## STEP 2: CREATING THE READER ACCOUNT

Before you can actually start sharing, it's a good idea to set up the reader account at this point. Now, when I say <em>**the reader account**</em> I mean the <em>**the admin reader account**</em>. I didn't realise beforehand that this was a thing, but I think it's important for me to stress this concept. Your account creates the admin reader account, creates the share, and then grants that share to the admin reader account.

The admin reader account is then responsible for adopting the share (more on that later), creating reader accounts, creating roles, granting database objects to those roles, and allocating roles to users.

![]({{site.baseurl}}/images/240621-reader.PNG)

You can either create this account with SQL, or use the web interface. If you want to use SQL you can do so like the below:

```sql
CREATE MANAGED ACCOUNT reader_admin
    ADMIN_NAME = reader_admin , ADMIN_PASSWORD = '<password>',
    TYPE = READER;
```

<br />

Or if you want to use the web interface, you can do so as below:

![]({{site.baseurl}}/images/230621_STEP_2.PNG)

After either, execute the following command...

```sql
SHOW MANAGED ACCOUNTS;
```

... and take note of the account_url and locator.

## STEP 3: CREATE THE SHARE AND GRANT IT TO YOUR READER ADMIN ACCOUNT

Next, you want to execute the following commands. What you are doing is:

1. Creating the share - replace BOOKSHOP_SHARE with your own bookshop name

```sql
CREATE SHARE BOOKSHOP_SHARE;
```

<br />

2. Grant usage on the database (replace DEMO with your database name and BOOKSHOP_SHARE with your share name), then on the schema (replace DEMO.BOOKSHOP with your schema name), and then on any view or tables you wish to include in the share (replace DEMO.BOOKSHOP.SALES_PER_AWARD with the view/table etc you wish to share)

```sql
GRANT USAGE ON DATABASE DEMO TO SHARE BOOKSHOP_SHARE;

GRANT USAGE ON SCHEMA DEMO.BOOKSHOP TO SHARE BOOKSHOP_SHARE;

GRANT SELECT ON DEMO.BOOKSHOP.SALES_PER_AWARD TO SHARE BOOKSHOP_SHARE;
```

<br />

3. Just check your grants and make sure your share has access to everything you want it to.

```sql
SHOW GRANTS TO SHARE BOOKSHOP_SHARE;
```

<br />

4. Now you need to establish the link between your share, and your reader admin account. To do this, you need to alter the share and add the **locator** value you noted down when executing the SHOW MANAGED ACCOUNTS command.

```sql
ALTER SHARE BOOKSHOP_SHARE ADD ACCOUNTS = <insert_reader_account_admin_locator>;
```

<br />

## STEP 4: SET UP THE SHARE IN THE READER ADMIN ACCOUNT

Now that the account and the share are linked, navigate to the URL provided by the SHOW MANAGED ACCOUNTS command. There, if you click on **Shares** at the top - make sure you've assumed the ACCOUNTADMIN role - you should be able to see the share. If so, then you know the previous steps have all worked out.

![]({{site.baseurl}}/images/240621-share.PNG)

All that's left is for your reader admin account to adopt the share. It's important for me to point out here that just because a share has been shared with you, this doesn't mean that you can instantly access the tables/views/etc... in the share, you have to adopt it or, put another way, issue a few commands to accept the share.

1. Create a warehouse that will be used to process the data - everything will look pretty blank until you execute this command. Replace READING_WH with whatever you wish to call your warehouse.

```sql
CREATE WAREHOUSE READING_WH;
```

<br />

2. Create a database using the share - replace READING_DB with whatever you wish to call your database, and YT1837."BOOKSHOP_SHARE" with the name of the share; the TT1837 bit is the locator of the data provider account.

```sql
CREATE DATABASE READING_DB FROM SHARE YT1837."BOOKSHOP_SHARE";
```

<br />

3. Import the privileges attached to the share to your role(s) - replace READING_DB with your database name as chosen in 4.2, and replace ACCOUNTADMIN with whichever role you want to allocate the imported privileges to (I definitely suggest granting it to ACCOUNTADMIN though)

```sql
GRANT IMPORTED PRIVILEGES ON DATABASE READING_DB TO ROLE ACCOUNTADMIN;
```

<br />

4. Test your share!

```sql
SELECT *
FROM READING_DB.BOOKSHOP.SALES_PER_AWARD;
```

<br />

Mine is looking pretty good!

![]({{site.baseurl}}/images/230621_result_2.PNG)

From here, you want to create non-admin reader accounts that you will distribute to people you want to share your data with, grant them roles and, by extension, privileges. All of that, however, is outside the scope of this blog.

Reader accounts and data shares, in my opinion, are a wonderful feature of Snowflake. What they allow is a dramatic democratisation of data. Not only can you distribute data outside your organisation, but many organisations have the issue of how to share data with people who know very little SQL inside their organisation. With the above methodology, you can create reader accounts for those users, create shares with all the complex SQL they would need to execute, and just allow them to execute simple SELECT \* FROM X statements. Often, these are people organisations don't want to grant full access to a data warehouse, which is easily done with roles and access control, but I think reader accounts are a nice alternative way of granting read-only accounts to internal users.
