---
layout: post
title:  "Building a Socially Distanced Seat Allocator in Alteryx"
date:   2020-09-28
image:  images/28092020.input.png
tags:   [Alteryx]
---

As mentioned in a [**previous blog**][prev-blog], I wanted to create a social distancing seat-allocator app so that we could maximise the numbers of people being able to attend a service, whilst still keeping them safe. To do so, we first had to create a spatial object representing the church, you can find that blog here.

Part of the issue was that we needed an app that could group households together, because by grouping them together we could fit more people inside the church while maintaining social distancing. 

I thought this blog would work best if I cover the logic of the process as opposed to deep-diving into the configuration of the hundreds of tools used in this process.

Firstly, there were two inputs:

1.	My church spatial data
2.	Ticket data (assuming households would book together, and thus, share the same Booking ID)

My church spatial data contained three things:

1.	The church’s shape as a polygon object
2.	The seating areas (this was the church polygon with cut outs made where seating was impossible – e.g. archways, altars etc…)
3.	A grid made out of the seating area with each square representing a potential seat

![]({{site.baseurl}}/images/28092020.priority.png)

The church spatial object was dealt with first:

1.	It was cleaned so it was ready.
2.	Decided which of the squares in my grid would be ‘Square 1’, around which every other seat would be allocated.
3.	Inserted the grid into a macro where every other square would be given a number according to how far away it was from Square 1.

![]({{site.baseurl}}/images/28092020.part1.png)

I then dealt with the ticket data:

1.	Cleaned the data.
2.	I inserted the data into a macro that would ensure that anyone marked ‘priority’ (people who need to be at the front) would be ordered at the top.
3.	I determined which tickets were households according to their shared booking ID. 

![]({{site.baseurl}}/images/28092020.seats.png)

These two streams of data were then put into an iterative macro:

1.	The macro would essentially filter the tickets which used the iteration number + 1; this meant that during iteration number 1, household number 1 would be true and all other false, during iteration number 7, household number 7 would be true. 
2.	The ‘true household’ would then go upstream, the first member of the household would be given a seat – this was then the first allocated seat.
3.	The macro would then search for the closest one/two/three etc seat(s) (depending on how many people were in the household), and then allocate those seats.
4.	A one/two metre buffer would then be created around the household. 
5.	Any seat that came within the buffer would then be filtered out of the spatial data (preventing them from being allocated). That was then sent back to the start, meaning that for every iteration the number of seats that could be allocated became smaller and smaller.
6.	The tickets with seats allocated would be output in a union with the name of the ticket holder, the location of the seat they had been allocated and a seat ID. 

![]({{site.baseurl}}/images/28092020.part2.png)

Part 2 essentially did two things:

1.	It allowed the user to chose whether they wanted to optimise their seat allocation (part 1 assumed first come first served would be fine, but there could be situation where reordering the household order would fit more people in, so the user could decide to optimise the allocation of seats), if they picked yes this would trigger a process where:
	a.	The macro would run every possible order of tickets and check which of these would fit the most people
	b.	It would then output that version of the seating plan
As you can imagine, this was a long process sometimes taking 30/40 minutes so only really a good idea during particularly busy one-off services and so it was decided to make optimisation optional (via radio buttons that would collapse certain containers).
2.	Use reporting tools to create a PDF that would display the physical location of seats (with their seat ID), followed by a table that showed who had been allocated to which seat ID.
In the end, the app was only needed the once to measure max capacity during an average Sunday with a 1 metre rule in place. Nevertheless, creating this app was an incredible learning experience and so I wanted to share that knowledge on a wider basis.


[prev-blog]:https://marlandata.com/2020/09/16/creating-spatial-objects-in-alteryx-with-cms-and-degrees/