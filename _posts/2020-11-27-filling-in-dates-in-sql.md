---
layout: post
title:  "Filling in Missing Dates in SQL"
date:   2020-09-28
image:  images/271120-cal.jpg
tags:   [SQL]
---

<span>Photo by <a href="https://unsplash.com/@waldemarbrandt67w?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Waldemar Brandt</a> on <a href="https://unsplash.com/s/photos/calendar?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

A few months ago, for the sake of learning SQL, I started doing Preppin’ Data challenges via SQL. A method that I found incredibly effective after months and months during lockdown where I’d read endless PDF textbooks about theory. 
While doing [**Week 7’s challenge**][pd-w7], I found myself using a technique I learn from [**Sanjay Mistry**][sanj], an old colleague and veritable master of SQL. The technique fills in missing dates when those missing dates are needed.To explain the challenge, let me show you the before and after. 

####BEFORE:

![]({{site.baseurl}}/images/271120-starts.png)

The start files consisted of a current employee table (left) and leavers (right)

####AFTER:

The challenge was to give a month-by-month account of how many staff were employed each month, average salary of those employees, and total salary

![]({{site.baseurl}}/images/271120-End.PNG)

What was necessary was to create a table of months, and use some joins to connect each employee currently employed during that month to those months – but how would I get that table of months in the first place? Well, here’s this amazing trick comes in. The trick consists of using a series of common table expressions (CTEs) and cross joins. First we DECLARE our @StartDate then begin.

![]({{site.baseurl}}/images/271120-begin.PNG)

![]({{site.baseurl}}/images/271120-Overall.PNG)

Key to understanding what is going on here is understanding what a cross join does. A cross join joins every instance in one table with every instance in another.

![]({{site.baseurl}}/images/271120-CrossJoin.PNG)

The diagram on the left demonstrates a more classic use of the cross join, but there’s nothing to stop you from using a cross-join on the same table, and that’s exactly what we’re doing here in this trick.
What you will notice is that on the diagram on the right, whilst our letter and number tables have 3 rows each, the outcome of our cross join has 9 rows - 3\*3 = 9.

That makes a cross join the perfect technique when you want to multiply rows.Our lvl1 table (A) simply creates two rows with 1s.

![]({{site.baseurl}}/images/271120-lvl1.PNG)

Lvl1/(A)/Yellow is just saying SELECT everything FROM (subquery where we create two rows that just say 1) AS col (the name of our column that we wrap in brackets with a name around t(col) which is just the standard when using VALUES in a sub-query).

With Lvl2/(B)/Pink, Lvl3/(C)/Light Blue and Lvl4/(D)/Red we’re creating more self-cross joins which leads to a continuous amplification in the number of rows.

![]({{site.baseurl}}/images/271120-lvls.png)

Eventually, with Lvl4/(D)/Red we have 256 rows – more than enough. With the n/(E)/Orange CTE we then give our 256 rows some row numbers.

n table/(E)/Orange

{% highlight ruby %}
n AS (SELECT ROW_NUMBER() OVER (ORDER BY(SELECT NULL)) AS RowID, 1 AS num
	  FROM lvl4),
{% endhighlight %}

![]({{site.baseurl}}/images/271120-n.PNG)

y table/(F)/Dark Blue

{% highlight ruby %}
y AS (SELECT RowID, DATEADD(month, (RowID -1), @StartDate) AS IndividualDate, num
	  FROM n),
{% endhighlight %}

![]({{site.baseurl}}/images/271120-y.PNG)

h table/(G)/Brown

{% highlight ruby %}
h AS (SELECT IndividualDate, [Employee ID]
	  FROM y
	  CROSS JOIN TU
	  WHERE IndividualDate < GETDATE())
{% endhighlight %}

![]({{site.baseurl}}/images/271120-h.PNG)

Now that we’ve created row numbers, in the y/(F)/Dark Blue table we can use the variable @StartDate we declared at the very start of the script and use it in a dateadd calculation: DATEADD(month, (RowID – 1), @StartDate) AS IndividualDate – this will use our RowID to progressively add another month to each row. 

Now that’s done, in our h table /(G)/Brown, we cross join to our start files (that I cleaned up and unioned in the TU CTE) giving us just the employee IDs (the rest will come later) so that each employee is attached to a separate set of months IndividualDate (which will repeat for each employee ID) and just to prevent a super explosion of data, I limit this to dates preceding today with my WHERE clause.

![]({{site.baseurl}}/images/271120-final_cleaning.PNG)

In my final two bits of cleaning, I just turn my Join Date and Leave Date columns to JoinMonth and LeaveMonth, both starting on the 1st of each month to make the next join much smoother. Once that’s done, I join my employee data to my h table on a.[Employee ID] = b[Employee ID], but I limit this with my WHERE clause to only retrieve rows where that IndividualDate, which contains my set of months, is greater than the start month and less than the leave month. 

As a result, I now have a table where each employee has a corresponding row for each and every month when they were employed (no more and no less).

![]({{site.baseurl}}/images/271120-TU_J_TOGETHER.png)


Now all I need to do is arrange the maths (find the average salary, sum salary and count the employees) for each month by grouping by my IndividualDate column. 

![]({{site.baseurl}}/images/271120-Fin.PNG)

And done! Whenever you need to look at dates between two dates provided this is how you do it in SQL.


[pd-w7]:https://preppindata.blogspot.com/2020/02/2020-week-7.html
[sanj]:https://www.linkedin.com/in/sanjay-mistry-a1a894/