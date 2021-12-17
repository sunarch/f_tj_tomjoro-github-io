---
layout: post
title: Is Functional Programming a Lie? 
image:  /img/blog_function.png
tags: [Functional Programming, Elixir]
---

Recently I read some articles with catchy headlines like: Pure Functional Programming is a Lie, Functional Programming is not Real, etc. -- what's reality got do do with it? Some of these articles are often written by experts, i.e. developers or computer scientists who have years of experience in pure functional programming - I wonder where the disillusionment is coming from.

This seems to be their conclusion: _In the real world computers have mutable state, so trying to program without mutable state goes against the natural order of things and makes programming difficult and makes execution of those programs by real computers more difficult_

Some developers feel the need for deeper justifications when choosing a computer language - it can't just be "follow the crowd", or "it's easier to find examples". But do we need to bring _reality_ into the discussion? 

And are there really only 2 choices? Either we reject mutability and program in a pure functional language, or we just accept mutable reality and join the crowd? 

I've thought about this question sometimes, especially since I started using Elixir (5 years now), when trying to build a mental framework for categorizing languages.

## Functional languages and Pure functional languages

Javascript is a functional language. Why isn't it a "pure" functional language? 

Pure functional languages, e.g. Haskell, F#, OCaml, are typically descendants of Lisp. Back in the early days of computer science, a lot of work was done to try to use math to prove program correctness. Think about that - it would mean that when you delivered your program to a customer you could say "this is provably correct so there cannot be any bugs by definition", unlike today when we say "it has 80% test coverage and worked when I tried it" ;) 

There's no exact definition of fuctionally pure purity, but generally it's these things:
* Immutable state, 
* No side-effects (substitution) 
* First class functions (functions can treated as data)
* etc. 

Javascript fails purity because it allows for mutation of state. Actually, most mainstream languagse fail for this same reason: Java, Go, C#, Rust, Python, Ruby, etc. However, almost all of these languages have, or have recently, started to use a lot more functional and immutable constructs. 

So are there only two categories?
* pure immutable functional,
* impure and having on mutation.

Read on to discover the shocking truth! ;)

# State

## Mutatble state
I put 99 beers on the shelf. The shelf now has 99 beers. Hence, the real world has state.

In programming terms I take a variable "x" to be the shelf, and I give x a value of 99. If I take one away then x is 98 beers. The shelf has state and is mutable.

This is the same way computer memory works. I store value in a register or in a memory location and it's stays there. 

## Immutable state
Pure functional languages don't allow mutation. In our beer analogy, the shelf with 99 beers can never be anything else. If you want a shelf with 98 beers, you'll need a new shelf because the shelf with 99 beers _cannot_ mutate.

Not only is this difficult to understand, in a pure functional language state changes are realized with functions - which can get very complex as the entire state of the world (the program) needs to be traceable to every action that caused a change. 

## Who's lying?

Does the real world have mutable state? Certainly seems like the mutable state matches our _view_ of the world better and I can't argue with that -- this is the "functional programming is a lie" bit.

But pure functional programming does match mathematics. That means that it's easier to prove what happened in a mathematical and conclusive fashion. In pure functional languages beers don't just disappear - there's a reason that can be traced! (very useful!)

## Real computers 

I don't know why it's important for a programming language to actually represent how they are implemented in physical computers. As far as I am concerned our goal as programmers is primarily to write programs that both humans and computers can understand and that work reliably.

I’d agree that _pure_ functional languages are not very practical, but don't throw the baby out with the bathwater. Just because you drop the "pure" part of "pure functional programming" doesn't mean you should give up an all the tennants (immutability, substitution (no side effects), first-class functions, etc.) 

# Categories of Purity

There are three categories:
* Pure functional languages (Haskell, F#, Common LISP, OCaml) - also includes "mostly pure languages"
* Impure languages (Java, Go, Rust, C#, Javascript, Ruby, Python, etc.)
* Elixir

Elixir is the most widely used computer language that is 1) functional, 2) does not allow _any_ mutation, and 3) but does allow for side effects. 

That's what makes Elixir so hard to grasp for newcomers. Because if you're trying to put in the one of the pure or impure categories you will find it doesn't fit in either!

_That'a a bit of a hidden joke because I really don't know any other languages in this category -  maybe readers of this Blog can help me out. And that just because Elixir might be unique in these regards that doesn't mean it's not a practical language (it is), nor that it is not widely used (it is)._

# Mutation is Evil

Reliable programming languages (and even languages that have mutable constructs) use immutability to make programs easier to understand for both humans and computers.

For a primer: here's an example of immutable in action in Elixir:
```
  map = %{test: 1}
  Map.put_new(map, :fun, 2)
  IO.inspect map   

  # Output: %{test: 1} -- the value of map did not change, but rather 'Map.put_new' returned a new Map (which we stupidly ignored) 
```

If our goal was to add something to a map, then we need to do this:
```
  map = %{test: 1}
  map1 = Map.put_new(map, :fun, 2)
  IO.inspect map1   

  # Output: %{test: 1, fun: 2} -- Note, we could have reused the name "map", but don't confuse that with mutation 
```

Making this discussion as simple as I can: 

![Elixir]({{ site.url }}/img/blog_function.png)

* A function takes inputs and computes an output

In this drawing, x had an assigned value of 25 and after calling f(x) the output was 5. Possibly this function computes the square root, but we can't know because it's a black box. 

* In a *pure functional language*, x being 25 will always give 5 as an output. (in fact the whole thing can be susbtituted for the output)
* In a *"impure" functional language* (one that allows side effects), sometimes when I put 25 in, the output might be 5 and maybe other times it comes out as 3.1415. This is called a having side effects. (for example getTime() will always yeild a different result)
* In a *impure language with mutation*, sometimes 25 comes out 5 or 3.1415 and sometimes x is might somewhat magically change to something like 52.

Of these three cases, *mutation* is the hardest one to understand and compose because the black box can no longer be treated as a black box - it can affect the world outside the box. We must look into the black box to understand what it did to x (or to see if it will do something to x).

# The Real World

## Time
Remember the beer analogy, i.e. that the shelf has state, that computer memory has state?  

The real world is actually _immutable_. 

How can this be? It's because we forgot to consider _time_ in our world view - and time does exist!

The real world is immutable when viewed at any point in time
* the past can’t be changed and 
* state is only definable by observation (at a time).

State arises from recursion in time. When you look closely at how computer memories actually work, you can see this recursion _even at the level of the logic circuits that implement computer memory_

And this is confirmed historically - original computer memories were things like mecury delay lines, even CRTs, as recused and repeatedly played back a signal put into them. In the past this was more outwardly apparent, but now this recursion is hidden in logic circuits.

 Here's a logic diagram of a flip-flop. A flip flop is a 1 bit computer memory and is how RAM is built in a _real_ computer.

![Elixir]({{ site.url }}/img/flipflop.png)

You don't have to understand this diagram, but notice how the lines feedback on themselves. This is recursion in time. There's all the proof you need: state arises from recusion in time :) . 

And since computer memory is built with recursion, I don't see that computers have any difficulty in executing this.

## Distribution 

If time is real, it also prevents us from know the state of the entire world at any point in time. We can only know our own local state (or nearby). This is distribution.

If you accept distribution then you just have to accept side-effects, because state is going to change in other "parts" for reasons unbeknownst to our local reference. Introducing side effects means you can no longer use mathematical descriptions of the entire system (relativity - very Einstein)! What are these "parts", or local references?

In the Erlang/Elixir these independent "parts" that each have their own local and immutable state are *processes*. Processes have observable state, which can be observed from the outside by sending a message to the process and asking it for the state and the process in turn sending you a response. Each process is a consistent world unto itself and has nothing shared with other processes.

This matches the real world - If you can’t know what’s happening somewhere else - then state must be communicated to be known. _This is especially true nowadays as computing has become increasingly distributed and parallel even on a small scale. For example, in multi-core computers each core has it's own dedicated cached memory and so getting a value from another core requires requesting it and copying._

The nice thing about this is that processes can't interfere with other processes by definition, and hence can be cleanly terminated at any time. This in turn guarantees that the system as a whole can continue to run predictably regardless of the failure of individual parts. Programming languages that have shared mutable state, at any level and through any mechanism, cannot provide this guarantee. The only way to provide this guarantee is completely ban shared mutatable state (processes can only communicate by messaging).

# Imperative is Simpler

A lot of developers might be surprised with Elixir because most of the code you see looks very imperative. In an imperative language code is executed from top to bottom, one statement (expression actually) after another. 

For example:

```
file = File.read!("my_file.txt")
strs = String.split(file, " ", trim: true)
subs = Enum.map(strs, fn str -> if str == "tom" do "Tom" else "Tom" end)
joined = Enum.join(subs, ",")
Database.write(joined) #just an example
IO.puts "done"
```

If Elixir is functional, then where are the callbacks, promises, awaits, etc? In Elixir I don't care if File.read is run in my process or asynchronous in another process - I need the result of the File.read before I can do the next thing. So I can write code simply and imperatively. What enables this?

Processes (and threads) were invented for two purposes: 
* To make it possible to write code simply and _imperatively_ (for humans)
* To make it possible to run more efficiently, concurrently or in parallel (for computers)

In Elixir there really is no difference between concurrent and parallel https://tomjoro.github.io/2017-02-03-why-reactive-fp-sucks/. Anything that is done with processes can either be run concurrently or in parallel with other processes because of immutability and local state. Processes are lightweight and are used like as any other language construct. 

Processes use _dependent_ imperative steps to accomplish a task. Imperative code is easy to understand and makes programs more reliable. And you don't need mutation to do this.

_This "style" of programming might have historically been impractical in many contexts, but has suddenly become relevant and practical (Why? See my other Blog https://tomjoro.github.io/2017-01-31-world-changed/ _Hint: distributed and parallel are no longer special cases._

# Is Pure Functional Programming a Lie?

Back to the original question: Is pure functional programming a lie?  Yes it is a lie, but not because of immutability being unreal, but because time exists in the real world. 

# Elixir's Category 

There is some middle ground, a category, between *pure-functional languages* and *impure languages with mutable state* -- and that middleground is a category that Elixir owns. I belive this is what drives the continued growth of Elixir. 

Giving this category a name is difficult and it's referred to in many different ways: is it "impure functional with distrubted and immutable state" or "functional, dynamic, safe language", or ?. In my opinion the category Elixir is in should be called "Reliable Languages".

Why?

Erlang (and therefore Elixir) wasn't designed to be an accurate representation of physical computers, or to be mathematically provable, or scalable, or versatile, or fast, or even friendly and fun for programmers - its first and foremost goal was to be reliable. Every decision about the language (and execution environment) as it evolved was prioritized by reliability, 

Reliable software:
* is simple and understandable,
* is built from simple independent (possibly distributed) parts that are composed, 
* has predictable and consistent behavior (scheduler, memory management, etc.),
* is secure,
* is maintainable,
* scales - scaling is a bonus because simple composed independent components also scale better in many cases (nowadays)
* etc.

To be clear: I belive reliability is the most important criteria in a computer language. Of course, developers still need to write good reliable code - but reality (reality again?) is that we are only human and make mistakes. 

# Disclaimer

Just to set the record straight - Haskell is useful, Go is nice, I love Ruby, C++ is efficient, C# is handy, Java is solid, Rust makes sense, etc. - I've been programming for over 40 years and it's not my intention to bash on other languages - comparisons that evoke emotions promote thinking and discussion :) and so happy coding!

(c) 2021 Thomas O'Rourke