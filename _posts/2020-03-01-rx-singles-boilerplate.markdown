---
layout: post
title: "The thing you hate about RxJava is Singles"
date: 2020-03-01 07:13:00 -0500
permalink: "/damn-you-singles/"
---

I just started working on a new codebase that makes heavy use of RxJava and
what I realized is that the thing that's painful about codebases that do
everything in Rx isn't the fundamental Rx principles, it's that we're using
`Single` and `Completable` in places where we should just be doing imperative
programming.

You end up with a lot of flows that end up looking something like this:

```
mySingle 
    .flatMapcompletable { theThing -> 
        if (theThing.hasErrors) {
          Completable.error(MyCustomError())
         }
          else Completable.complete()
         }
    }
 ``` 

`Single`,`Completable`, and `Maybe` just carry a lot of boilerplate with
them that isn't necessary.

That's why I'm ultimately bullish on coroutines and Flow at this point. And to
be clear, I wrote a book about RxJava - I **love** RxJava. But suspending calls
remove all of the boilerplate that goes along with the above specified types,
and that's a really big win.

Just to hammer on the point, I think `Observable`s are an *amazing* abstraction
that really bring a powerful programming paradigm with them. Streams of data are
a really novel way of modeling your program, and it carries with it all of the
decoupling benefits, up to date data, and declarative programming models that we
all love about Rx.

It's just that we're trying to use it for one off things that don't make sense
as streams of data. A network call isn't a stream (assuming it's not a socket).
It's a one off imperative call. Same with a database insertion or deletion. It's
modeling this stuff with Rx that's making things so painful.
