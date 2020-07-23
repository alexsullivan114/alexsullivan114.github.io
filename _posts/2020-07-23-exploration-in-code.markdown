---
layout: post
title: "Garbage code is the fastest way to beautiful code"
date: 2020-07-23 09:00:00 -0500
permalink: "/beautiful-garbage/"
---

I recently read a blog post about [writing easy to delete code](https://programmingisterrible.com/post/139222674273/write-code-that-is-easy-to-delete-not-easy-to). It
had a line about most code being exploratory. That got me thinking about how I typically approach a problem and how that's evolved over my career.

In the beginning, when I was given a new task or feature to develop I would start off with grand aspirations for how the code would be organized. I would begin to setup
all of the different abstractions that I would use and step by step I would make sure to write everything as cleanly, as well tested and as future proof as possible.

Predictably, that lead to a lot of incorrectly abstracted code. Often I would find that by the end of my work on that feature, assumptions I had made about the shape
of the work needed to build the feature in the beginning stages of working on it had been wrong, and the abstractions I had created on top of those earlier abstractions
made the code difficult to work with and unsatisfactory. I'd get to the end of that feature and feel frustrated because the code didn't feel particularly
elegant and I didn't have time to make changes.

I eventually realized that I'd need more time to get the code to a place that I was proud of. So my new pattern would be to architect the feature as well as I could and
then ultimately do another round of refactoring once the feature was built to fix up the poor abstractions I had designed earlier. That usually left me feeling much
better about the code at the end of the day, but it also meant spending a lot of time writing unit tests and functionality that changed several times over the course of
building that feature.

That eventually lead me to my third and current style of writing code. I first get the feature working in as little time as possible. That means writing absolutely
_garbage_ code. I mean code that you'd **never** **ever** show your coworkers. Code that you're hesitant to commit, even to your local development branch because
someone might at some point comb through your commits and see it. Global variables everywhere, giant god functions that do everything, no testing. Just beautiful garbage.

Then, once the feature is fully functional, I go back and actually clean things up into a respectable state.

I've found that at this point this is my best path forward because the reality is I almost never _know_ what will be required to build the feature. When I start on
a feature, I'm usually starting with a huge host of assumptions or unknowns that change and evolve over the course of building the thing. And those unknowns shape entirely how
the feature should be architected. So until I resolve those assumptions and unknowns, there's really no point in trying to come up with abstractions and architectures for the feature,
because they'll inevitably be the wrong abstractions once I learn more about the problem. 

This obviously only works if the feature is small enough that you can actually hack it out in a semi reasonable amount of time. 

Tl;dr: GARBAGE AHOY!
