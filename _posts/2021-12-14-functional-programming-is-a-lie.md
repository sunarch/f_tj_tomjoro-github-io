---
layout: post
title: Tate.me
image:  /img/mutation1.png
tags: [Functional Programming, Elixir]
---

# Functional Programming Is a Lie

Deeply philosophical question and something I’ve pondered quite a bit - especially since I started using Erlang/Elixir, which will be apparent from the following :) .

Recently I read some articles that state that "Pure Functional Programming is a Lie". The whole discussion seems a bit of sour grapes to me - Developing practical programs in Haskell is difficult, so before I start using Javascript I want a good/deep reason for abandoning pure Haskell. 

The argument is that because pure functional programming doesn't represent what happens in a physical computer / real world that it is therefore a lie (so I might as well use Javascript/Go/Rust/Ruby/Java/etc.).

Specially, the point is made that the real computers have mutable state, so therefore programming languages should have mutable state. 

The difficulties with pure functional languages is not the immutable state, nor that they don't model the real world closely enough.

What's immutable state got to do with it?

# Global State is a Lie

I’d agree that _pure_ functional languages are not practical, but disagree that this is because that it is because the world is actually mutable, and so therefore our computer languages should have mutation. The real world is immutable. Real computer systems are immutable. Reliable programs use immutability.

For a primer: here's an example of immutable in action:
```
  map = %{test: 1}
  Map.put_new(map, :fun, 2)
  IO.inspect map   

  # Output: %{test: 1} 
```

What’s missing from pure functional view of the world is _time_ and _distribution_. The real world is immutable when viewed at any point in time - the past can’t be changed. Also, there is no such thing as global state in the real world (you don’t know what’s happening somewhere else - it has to be communicated), or any computer system - especially true nowadays as computing has become increasingly distributed and parallel (multi-core systems which each have their own cache, memory, etc.).

Algorithms increasingly must be understood in the context of parallel multiprocessor/distributed systems. Entire books discuss algorithms and big O complexity without ever mentioning that all bets are off when you introduce parallelism because it becomes mathematically “difficult”. If one introduces threads sharing mutable state, it becomes basically intractable (see “The Problem with Threads” from Lee).

State arises from recursion in time. Original computer memories were exactly this, delay lines, even CRTs that simply played back a state put into them. Logic gates are the same, they model state recursively. State is observable at a point in time and is therefore immutable. Any system, when viewed from the outside, will have observed state, even if internally it has immutable state which is functionally defined (with recursion).

Given this view, then one must accept that functions (and real systems) can have side effects, i.e. given same inputs a function can return different results. A good example is get_time(). This function returns a different result each time you call it because it might be the result of an external system and not part of the local state -  Introducing side effects means you can no longer use substitution in a pure functional context.

So, in the Erlang view of the world, you program with local immutable state in processes. As a system you communicate with other processes (by sending messages) that also have local and immutable state  - but process have observable state, and so does the system. Another advantage of this is that a single process can be cleanly terminated at any time and guarantees that the system as a whole can continue to run predictably. Programming languages that have shared mutable state, or even the possibility to do this with “unsafe” constructs, (threads) cannot provide this guarantee.

I think a lot of developers with experience in pure functional languages might be surprised with Elixir because it looks like an imperative language! In that regard I agree - simple imperative constructs are easier to understand. And we can make simple code because we can use processes as first-class citizens and having immutable state - which feels like a perfect analogy to the real world.

The most important thing (now) in the future will be _reliability_ and the way to make reliable programs is to make programs that both humans and computers can understand and work in the real world (which is distributed in space and time).