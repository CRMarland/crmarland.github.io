---
layout: post
title:  "Creating Spatial Objects in Alteryx with CMs and Degrees"
date:   2020-09-16
image:  images/160920.blueprint.jpg
---

As we were nearing the end of lockdown in England, there was concern about how social distancing regulations would impact places of worship – could these old and heavy buildings contain entire congregations of worshipers in full? Well, possibly. What helped was that the government had specified that households could sit together. Perhaps by grouping households together, the availability of space could be maximised without putting anyone in any danger of catching corona virus.

To leverage data analytics to do this, before even beginning to analyse capacity and create seating plans, we would need to first create a spatial object that represented the church. My church, St Bart’s in London, was happy to play guinea pig and very kindly offered me their blueprints.

Much of what I’ve done I’ve done via adapting the formulae described in this web page written by [**Chris Veness**][chris-veness]  which proved an invaluable resource.

First thing’s first, I took out a pencil, ruler, and protractor. My methodology was to pinpoint a start-point at the centre of the building and then identify points for each and every twist, turn and corner of the building. 

My data ended up looking like this:

![]({{site.baseurl}}/images/160920.start.PNG)

From my centre point I was calculating the distance in cm to each point – every corner – and then the angle with north being 90° and south 180°.

Each point had a POINT ID I could later use for the sequential order of my points. 

In terms of data prep, the first step was to convert those angles into degrees bearing and then into radians, which I did in three steps:


##### Degrees Bearing
{% highlight ruby %}
[ANGLE] - 90
{% endhighlight %}

##### Degrees Bearing
{% highlight ruby %}
IF [Degrees Bearing] < 0 THEN
360 - ABS([Degrees Bearing])
ELSE [Degrees Bearing]
ENDIF
{% endhighlight %}

##### Degrees Bearing
{% highlight ruby %}
[Degrees Bearing] * PI()/180
{% endhighlight %}

![]({{site.baseurl}}/images/160920.degrees.PNG)

Next up, what I had to do was convert my CMs into kilometres, this was done by simply multiplying the CMs by the scale of the blueprints (which we got wrong at first, but never mind, easily changed once we turned this into an app and allowed the user to input their own scale) and then divide that figure by 1000.

##### Kilometers
{% highlight ruby %}
([CM] * 4.73)/1000
{% endhighlight %}


Once all that was done, I used a map input to select a random point in the church (which would act as my centre-point, there was no need to actually be accurate at all here, but some accuracy does help if you want to compare).

![]({{site.baseurl}}/images/160920.map.PNG)

I then used the spatial info tool to extract the longitude and latitude (named Lat and Long) and then appended this to my church points data.

Before I moved on I had to do was just figure out the earth’s bearing in London, not a nice task, but luckily a [**website**][website] was willing to do it for me.

![]({{site.baseurl}}/images/160920.fullflow.PNG)

Then came the big stuff:

1. 	I had to turn my latitudes and longitudes into radians

##### Lat
{% highlight ruby %}
[Lat] * PI()/180
{% endhighlight %}

##### Long
{% highlight ruby %}
[Long] * PI()/180
{% endhighlight %}

2.	Work out the new longitudes and latitudes using my centre point, angles, CMs and the earth’s bearing in London (6365.086, so replace this number with that of your own location).

##### New Lat

{% highlight ruby %}
ASIN(SIN([Lat]) * COS([Kilometers]/6365.086)+COS([Lat]) * SIN([Kilometers]/6365.086) * COS([Degrees Bearing]))
{% endhighlight %}

##### New Long

{% highlight ruby %}
[Long] + ATAN2(SIN([Degrees Bearing]) * SIN([Kilometers]/6365.086) * COS([Lat]), COS([Kilometers]/6365.086) - SIN([Lat]) * SIN([New Lat]))
{% endhighlight %}

3.	Convert these new longitudes and latitudes back (i.e. not radians)

##### New Lat
{% highlight ruby %}
[New Lat] * 180/PI()
{% endhighlight %}


##### New Long
{% highlight ruby %}
[New Long] * 180/PI()
{% endhighlight %}

![]({{site.baseurl}}/images/160920.latlong.png)

The next step was to use the Create Points tool to turn my New Long and New Lat into spatial points, and then use PolyBuild to use those spatial points to create a polygon object (with my Point ID as the Sequence Field).

![]({{site.baseurl}}/images/160920.mppd.png)

Our end product was exactly what we wanted:

![]({{site.baseurl}}/images/160920.end.PNG)

Because our scale was wrong, the polygon object ended up being smaller than the actual church, but once we started using the workflow as an app, that was easily solved. The next step was to go through this process again (just copying and pasting a lot of tools) to create 'exclusion zones', objects within the church that made it impossible to sit in a certain location.

In a following blog, I’ll explain how I used this to create an app that allocated socially-distanced seating.



[chris-veness]: https://www.movable-type.co.uk/scripts/latlong.html
[website]: https://rechneronline.de/earth-radius/
