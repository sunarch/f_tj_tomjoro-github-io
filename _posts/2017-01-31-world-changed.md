---
layout: post
title: Elixir - Erlang didn't change, the world did
image: /img/mutation1.png
tags: [elixir, erlang, functional programming, concurrent]
comments: true
---

I gave a presentation last week for the Elixir Finland meetup group where we talked about why Elixir is Different.

You can view the slides here: Slides: Why Elixir Is Different:
[Slides](https://www.tate.me/project#!project_id=5889e6119dc153885b23143f&page=1&version=1&mview=image&tryit)

_BTW: Tate.me is a collaborative slide share and annotations tool that I developed and am current porting to Elixir/Phoenix.
Read about it here: [Tate.me: Collaborative annotations]({{ site.baseurl }}{% link _posts/2017-01-15-tate.md %})_


Many well intentioned developers introduce Elixir as simply a step sideways from languages like Ruby. The similarity of Ruby and Elixir is deceiving and misleading. Whenever I see a performance test comparing Clojure, Go, Node.js, etc. to Elixir I cringe - how can you compare them? Elixir is a safe environment, without shared memory, threads or immutability problems that are ever present in other languages.

Languages like Scala and Clojure try to do the right thing - to take away the guns of mutable state and threads. But the guns are still there, just hidden a bit deeper.
Here's a nice info chart I dreamed up:

![Language comparison]({{ site.url }}/img/presentation.png)

## Guns

What are these "guns" and why are they dangerous? The "guns" are mutable constructs and threads (shared memory). You might find this statement offensive, since using these guns has been the most common way to write programs for a long time now and probably what you learned in school. Suggesting we can just give them up might get you labelled as a heretic. (Or one can short of a six-pack)

What's mutation anyways? Maybe a little example for those of you who haven't tried Clojure.

![Ruby]({{ site.url }}/img/ruby.png)

This is a Ruby snippet where I create an object, define a little hash with a and b, and then call a do_something method on the Fancy object. The question is: "What is the value of the hash after you call do_something"?

The answer is, of course, you don't know because in order to answer this you'd need to look at the code in Fancy. And if do_something changes hash, then this is what we call a mutation and a side-effect.

Now, if we use a language that does not have mutation, like Elixir, we'd be able to answer that question without looking at any other code - because it can't be mutated (or changed). Asking an equivalent question in Elixir:
![Elixir]({{ site.url }}/img/mutation2.png)

This doesn't even make sense. the "hash" is there, and once declared it can never change. No need to go looking anywhere elsewhere, do_something simply cannot change that hash. Very safe and understandable. This may seem to be a small difference, but the difference is huge when you consider concurrent programs - you can safely pass the hash by copying it to another process. And that's why immutable data structures are a good fit for concurrency.

Regarding threads: we're only human, sometimes we forget a mutex and a thread dies. Or maybe you don't "use" threads, but rather rely on Rails, etc. to provide them for you. Which works fine until you try to do something concurrent. Maybe you don't understand the threading model, and use a global variable, etc.

## Comparing the language guns

Not to pick on Ruby - I love Ruby, but I'm afraid it's in the category "Guns don't kill people, people kill people", or translated into developer terminology: "Code doesn't kill applications, coders do". Use them wisely, or bad things may happen. And when bad things happen, we have failed as coders, so we should write more test cases, get some more training, learn how to use the guns.

Moving to the right, languages like Scala and Clojure, understanding the danger of guns, decided that they should not be available by default. Let's set them aside, put them in a cabinet, at least put the safety on! Very nice ideals, but again we are only human, and accidents happen. Maybe it wasn't your fault: some junior coder kid who was doing some maintenance on a piece of code decided a reduce was too difficult, and used a mutable construct... Bam. JVM runs on threads and the definition of a thread is shared memory, think about that for a second.

Moving to Erlang. What if we just took away the guns: no threads, no mutability. Most programmers still think that this is just impractical. "You mean that I have absolutely no choice, that I must use reduce, recursion, etc.? That can't be practical for most applications". As one blogger said "How can I use a language that doesn't have a for-loop? gimme a break, what a joke". Must be just for those academics...

I have to agree I was in the same mind-set about 18 months ago. But I was wrong, and I (happily) admit it. I've been programming for a long time, and I felt like I was blindsided, why didn't I hear about this before? Because I had mistakenly assumed the world was the same - it is not.

What happened: Moore's law ends, lots of memory, speed, concurrency everywhere. Simple networked services that run reliably for long periods - sound familiar?
Languages like Haskell don't even know what we're talking about. Haskell takes one step further than Erlang and says not only is the language immutable, the functions are really mathematical functions. That means if you pass f(1) into a function f and it returns 42, then it should always return 42 given the input of 1. Erlang thinks this is going to far. To an Erlang programmer Haskell is an impractical language. How can you make a function to get the current time?

## So, what happened?

Another info graphic I dreamed up to try to explain how I felt after I learned Elixir:

![Ruby]({{ site.url }}/img/world.png)

Most coders, moving along their career paths had the constant assumption that "Erlang, i.e. a functional concurrent language is not practical for my needs". What are my needs? I do application backends that use databases, web applications servers, REST APIs, authentication, processing, etc., pretty typical "internet" stuff. As developers we live in an exciting time, and many developers are too busy learning Node.js and Go to consider Elixir right now (hence the "Look here").

Meanwhile, for every year since 1992, Erlang has become more practical. In part because Erlang got cool stuff like SMP support, etc., but fundamentally it's not Erlang that is changing but rather it's the world (computing environment) that's changing.

So what happened? The world changed, and continues to change:

* Moore's law ends,
* I am sitting in front of a 16 core SMP computer,
* It has more memory than I could ever use,
* It is fast (on a single process),
* I am writing simple networked services,
* Reliable code, not just features, differentiates,
* Scalability is standard, and not special.

Elixir brings Erlang to the masses. At it's core, Elixir's strength is Erlang. Elixir brings great libraries, and tools support, and wraps it all up in a very modern offering. I put Elixir's creation date on this chart just before the "Erlang is practical" line crosses the "Programmers who believe" line. And really, that's exactly what the core Elixir team did when they created Elixir - they saw the world changing around them, and they reached for something that would seemed perfect for the job - Erlang.

So, now we're at the tipping point in 2017, and unlike most languages that struggle upwards on the adoption curve, there is already a large pent-up-demand for Elixir. And combine that with Erlang's mature platform and extensive libraries and you understand why coders like me are excited. These factors together will drive rapid adoption. And Erlang is not a here-today-gone-tomorrow technology - it is here to stay. You probably already use something developed in Erlang everyday and just don't realize it.

When I first started with Elixir I really was a doubting Thomas (I am). So, I wrote a some test programs that required a lot of mutating data first in Ruby and then in Elixir (github: Ruby vs Elixir) I confidently thought to myself that there was no way the Erlang garbage collection could keep up, that it would be inefficient, that tail recursion made programs hard to understand, etc. - wrong, wrong, wrong and wrong. What happened was exactly opposite - the code was faster, the program was more understandable - I had to believe the evidence in front of me.

[Elixir vs Ruby Sample Code](https://github.com/tomjoro/elixir_map_pmap_apmap_example)

A lot of coders also mistakenly believe that Elixir/Erlang is all about concurrency and functional programming, and that hence this would be the only reason to use Elixir in a project. Erlang was created with a safety first idea - and scalable concurrency was a side effect (pun intended). It's nice to know your program is scalable, but it's nicer to know that it's simple and reliable. Still, you get a warm fuzzy feeling than seeing all your cores respond like a finely tuned machine.

![All Your Cores Belong To Erlang]({{ site.url }}/img/cores.png)

## I'm only human, after all

Joe Armstrong (father of Erlang) is a brilliant guy. Reading a lot of what he writes, I think he must have a similar world view: I am not the world's best programmer and I know people much better at it that me. But I know what I want to achieve, and I have tools and capability to make it happen, and I care.

But I am a realist, and have to admit sometimes:

"I have failed. I have made bugs and crappy code that has caused pain for users and other coders. Sorry, I'm only human after all. "
So, where's this leading? To this: programming in Elixir/Erlang is safer. It respects you as a human (who is a coder), one that makes mistakes. It takes away the most dangerous guns (mutability and threads) and even better it supports your endeavors as a coder in ways you never thought possible.

When something does go wrong (which still can happen of course, and does!), there is a sophisticated layer of error handling, reporting and recovering to back you up. I like to think of it as safety first, with well rehearsed rapid responding emergency services (OTP).
Uses pattern matching and other constructs to make coding more declarative, in order to make the intent clearer, and easier to understand.
I could go on all day about the cool stuff, like the how sweet the pre-emptive scheduler is, how the processes work, the compiler, etc, but it will sound like a raving madman. Maybe you should just go and try it for yourself. [Elixir Language Website](http://elixir-lang.org)

# Conclusion

Elixir is not for every program and coder in the world. If you are writing a backend with a huge mutable state, or doing image processing, then it probably is not practical. Then again, if you're doing that you're probably using C++ anyways!

But you'd be surprised how easy it is to write simple, safe and reliable code in Elixir if you gave it have a chance - If nothing else, it will make you a better programmer, and at best it will make you seem like the super-hero programmer cranking out functional concurrent scalable networked internet applications! Elixir is actually an easy language to learn, but as counter-intuitive as it sounds: you must understand why Elixir is different or you might give up in frustration trying to make a "for-loop".

And for all of you doubters. Yes, I have been using Elixir & Phoenix full time for over a year, and it is practical and does work great in production. In fact it's been the most stable, fun and capable production environment that I have ever used.

Thanks to Joe Armstrong, José Valim, Chis McCord, and everyone who has contributed to Elixir/Erlang. There is an incredible level of thought-maturity and leadership that is hard to imagine from such a fresh undertaking.
