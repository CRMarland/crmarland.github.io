---
layout: post
title: "Tips On Passing Your SnowPro Core Exam"
date: 2021-05-26
image: 20210526.snow-header.jpg
tags: [Snowflake]
---

Photo by <a href="https://unsplash.com/@tomcoomer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Tom Coomer</a> on <a href="https://unsplash.com/s/photos/snow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

The other day, I passed my SnowPro Core Exam! What surprised me is that although the exam sounds pretty daunting, it wasn't one I found very hard at all. While some exams are quilty of trying to trip you up, or include unreasonable questions, this one was completely grounded. What I liked about the exam was it was a series of 100 quick-fire questions that had unambiguously right or wrong answers - there were no edge cases where **maybe** it's a or b if you kinda-sorta read it in this or that way.

All that said, I think people might still benefit from some tips and a bit of insight into places where I fell down.

### Resources

In terms of resources to use, Snowflake University and Snowflake's exam guides are the most immediately obvious places, and I definitely suggest using them. However, I think some supplementary resources are a good idea.

- Firstly, I think these [flashcards on Quizlet](https://quizlet.com/380510774/snowflake-certification-flash-cards/) are amazing (if ever-so-slightly dated), they help attach the names to the concepts and there are some golden nuggets of information I hadn't seen anywhere else.

- I also produced a PowerPoint to create some structure to my notes, but has also become a resource that quite a few of my colleagues at work have appreciated and used, I'd say 95% of the questions I got on my exam were covered by it.

<iframe src="https://onedrive.live.com/embed?cid=031EFAC83F985321&resid=31EFAC83F985321%21499&authkey=AFJ2Z-bQeFiYenM&em=2" width="402" height="327" frameborder="0" scrolling="no"></iframe>

### General Tips

1 - I'd say about 1/3 or 1/4 of the exam was centred around the differences between editions and layers, questions like "X is available from which edition upwards?" and "where is Y stored?" - I suggest really getting to know the different layers and editions, what each layer does and what's available in each edition.

2 - The other 1/3 or 1/4 of the exam is very statistics based - all about the key figures of Snowflake. "How many MBs is X?" - "How much bigger is Y from Z?". I think really remembering all the core statistics is really worth your time, these are the statistics you normally find in bold on Snowflake's documentation pages and hear thrown around a lot in video content Snowflake produces.

3 - One topic I didn't expect to come up much (and hence isn't covered hugely in my PowerPoint slides) is the intricacies of DML operations, especially MERGE. I would suggest reading up on some of the documentation here and you won't be tripped up by these questions.

4 - Another topic I wasn't expecting was the Snowpipe REST API. I've done a lot of playing around with Snowpipes, and even [did a video on them](https://www.youtube.com/watch?v=-MqbS83GInw), but the REST API isn't something I knew too much about when I did the exam, and I got at least three questions on it.

5 - In terms of approach, my approach was to answer the questions I immediately knew the answers to as quickly as possible, and then go back and spend as much time as I needed answering the questions I didn't immediately know the answers to. After that, I just went through every question to check I hadn't made any silly mistakes (I'd made two).

6 - I really think the best way to learn Snowflake is to play around with it. [Snowflake's free trial is incredibly generous](https://signup.snowflake.com/) and gives you ample time and credits to try out different things and experiment in an environment where you have complete control (which you probably can't do with your employer's environment).

7 - I would also advise against trying to rush into an exam. It's probably possible to pass the exam in one week if you have a lot of time, but I think it's worth getting to know Snowflake as opposed to learning just how to regurgitate facts - those statistics are going to make a lot more sense if you've spent the time playing around with it. Remember that exam badges aren't just collection items, if an employer or client expects you to know Snowflake, you don't want that badge to be misleading.

Good luck!
