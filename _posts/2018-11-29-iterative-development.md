---
title: Iterative development
date: 2018-11-29 22:00:00 +0100
categories: software development
---

The way I approach developing a new project/feature has changed over the years. I would describe my current style as very iterative but for comparison I will first describe how I used to do it.

## "I wanna learn"

When I started out programming my main concern was to get things working. A lot of trial and error, a lot of frustrations when things didn't work and some confusion when things suddenly started working.

## "I get it now"

In the next phase I was much more comfortable and the surprising errors and successes didn't happen as much as before.
My first instinct when I saw a slight hint of duplication, was to quickly extract the code to a function, method, class, whatever.
I was now (sort of) a programmer and code had to be DRY!

## "I know change is coming"

This is the current phase for me. At this point I have worked on a lot of my own hobby projects and
more importantly I have experienced working on production systems which are critical to the customers.

What I have come to learn is that things will change in the future, usually in unexpected ways. If a certain project or feature is not actively worked on it usually means that nobody is using it. And if you're lucky enough to work on something that people actually use, then that thing you originally envisioned and developed will probably need to change.

Change is a good thing. So the best thing you can do is to let change come easily. How?

- Write code that can easily be replaced. I think putting your pride aside and admitting that it is OK if your original design is not the right anymore, is completely fine.
- Solve the problem at hand with the knowledge you have *now*. Do not try to think too much ahead and be too clever because you think your idea will help extending the code later on.
- Write less code. The less mass a project/feature has, the easier it is to work with.

So the reason I would call the approach "iterative development" is because I now embrace that the code I write will grow and morph in iterations over time. Nothing with regards to code is set in stone.

I can recommend [this episode](http://www.fullstackradio.com/101) of Full Stack Radio where Ben Orenstein talks about similar approaches and "shortcuts".

*PS: How you store your data is something that is not easy to change like application code. But the same thinking applies: you can try to think ahead in terms of how you should store and retrieve data to optimize performance but going too far can end up hurting and confusing more than it benefits.*
