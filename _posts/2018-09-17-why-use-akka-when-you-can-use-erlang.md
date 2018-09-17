---
layout: post
title: Why should we "reinvent the wheel" by using Akka with Java if we have Elixir/Erlang ready for production?
image: /img/use-erlang.png
tags: [elixir, erlang, functional programming, concurrent, akka, java]
comments: true
---

-- work in progress. I'm adding details... Based on a quora article

We shouldn’t.

I did a presentation about this, and this is one of my nice slides :) about my journey.

![Elixir]({{ site.url }}/img/use-erlang.png)

I wanted both an actor framework, and a robust web server / framework - and I wanted them to work together seamlessly.

I was working with Rails so considered Elixir/Erlang immediately, but didn’t want to be a fanboy and jump on the next “cool” thing. So, I went around the circle, investigating and trying similar solutions to Erlang’s actor framework in each, but always found something lacking.

What were some of the shortcomings? Well, for example

* Celluloid was cool, but missing the protection and maturity of Erlang
* JRuby is great, but not all libraries work with it
* Go is cool, but it’s not directly distributable and the scheduler is just not for actor frameworks
* Java, Java8, several actor framworks, all not that mature
* Clojure, I like a lot, but needed a web framework, and after examining the internals of the libraries and how it compiles to Java it was scary (take a look at the code for the parallel * map reduce). Plus guaranteed tail recursion optimization (this is a Java problem)
* Scala is great but very complex and actually is based a lot on Erlang ideas. Doesn’t give true process safety and I need a web framework too
* Play’s threading model is strange, mixing both reactive and threaded which can cause surprises.
* At the end of the day, I found myself back using Elixir/Phoenix/Erlang. A true robust, safe, scalable, fault recovering system which most of the other actor models copied anyways.

As far as maturity, Erlang is mature, and continues to get better - it’s much older than Java and has been used in production for a long time: Erlang, OTP and running on the BEAM. I like to use Elixir, it’s newer but has great libraries, and Elixir is essentially Erlang so you won’t find any issues using Erlang libraries directly, for example Cowboy (arguably the best web server ever made), is used directly in Phoenix and each web request is totally isolated from every other request.

In the end, I ended up back at my starting point concluding that Erlang does indeed have the best actor framework, and having given all the others a good try I felt confident with my decision! I have been using it ever since.

