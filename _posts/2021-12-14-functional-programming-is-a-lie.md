---
layout: post
title: Is Functional Programming Is a Lie? 
image:  /img/blog_function.png
tags: [Functional Programming, Elixir]
---

# Background

Recently I read some articles with catchy headlines like: Pure Functional Programming is a Lie, Functional Programming is not Real, etc.

A deeply philosophical statement posed more as a question, and something I’ve pondered now and then - especially since I started using Erlang/Elixir, which will become apparent :)

Most of these articles are often written by experts, i.e. developers or computer scientist who have years of experience in pure functional programming (Haskell mainly) - I wonder where the disillusionment is coming from.

Statements like this: In the real world computers have mutable state, so trying to program without mutable state goes against the natural order of things and makes programming difficult.

Smart people (developers) need justification for their (langauage) decision - it can't just be "follow the crowd", or "it's easier". But do we really need to bring _reality_ into the discussion? Software development is anything but real.

## Functional languages and Pure functional languages

Javascript is a functional language. Why isn't it a "pure" functional language. 

Pure functional languages are decendants of Lisp, like Haskell and F#. There's no exact definition of pure purity, but generally it's these things:
* Immutable state, 
* No side-effects (substitution) 
* First class functions (functions can treated as data)
* etc. 

Javascript fails purity because it allows for mutation of state. Actually, most mainstream languagse fail for this same reason: Java, Go, C#, Rust, Python, Ruby, etc.

So is the world of programming languages divided only into pure and impure based on mutation? More on this later...

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

But pure functional programming does match mathematics. That means that it's easier to prove what happened in a mathematical in a conclusive way. In pure functional languages beers don't just disappear - there's a reason that can be traced! (very useful!)

## Real computers 

I don't know why it's important for a programming language to actually represent how they are implemented in physical computers. As far as I am concerned our goal as programmers is primarily to write programs that both humans and computers can understand and that work reliably.

I’d agree that _pure_ functional languages are not very practical, but don't throw the baby out with the bathwater. Just because you drop the "pure" part of "pure functional programming" doesn't mean you should give up an all the tennants (immutability, substitution (no side effects), first-class functions, etc.) 

You can do functional programming in many languages, for example Javascript is very functional. 

# Categories of purity

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

* In a pure functional language, x being 25 will always give 5 as an output. (in fact the whole thing can be susbtituted for the output)
* In a "impure" functional language (one that allows side effects), sometimes when I put 25 in, the output might be 5 and maybe other times it comes out as 3.1415. This is called a having side effects. (for example getTime() will always yeild a different result)
* In a language with mutation, sometimes 25 comes out 5 or 3.1415 and _sometimes x is might magically change to 52_ 

Of these three statements, mutation is the hardest one to understand because the black box can no longer be treated as a black box - it can affect the world outside the box. We must look into the black box to understand what it did to x. 

# Mutation is a Lie

Remember the beer analogy, i.e. that the shelf has state, that computer memory has state.

Mustation is not real. The real world is _immutable_. Real computer systems are immutable.

How can this be true? It's because we forgot to consider _time_ and _distribution_ in our world view.

_The real world is immutable when viewed at any point in time_ - the past can’t be changed. 

And there is no such thing as global state in the real world (you don’t know what’s happening somewhere else - it has to be communicated), or any computer system. _This is especially true nowadays as computing has become increasingly distributed and parallel (multi-core systems which each have their own cache, memory, etc.)_ 

Algorithms increasingly must be understood in the context of parallel multiprocessor/distributed systems. Entire books discuss algorithms and big O complexity without ever mentioning that all bets are off when you introduce parallelism because it becomes mathematically “difficult”. If one introduces threads sharing mutable state, it becomes basically intractable (see “The Problem with Threads” from Lee).

State arises from recursion in time. Original computer memories were exactly this, delay lines, even CRTs that simply played back a state put into them.  Here's a logic diagram of a flip-flop. A flip flop is a 1 bit computer memory and is how RAM is built in a _real_ computer.

![Elixir]({{ site.url }}/img/flipflop.png)

You don't have to understand this diagram, but notice how the lines feedback on themselves. This is recursion in time. There's all the proof you need: state arises from recusion in time :)

So, given that we have immutable, yet distributed state, then you just have to accept side-effects, i.e. given same inputs a function can return different results. A good example is get_time(). This function returns a different result each time you call it because it might be the result of an external system and not part of the local state -  Introducing side effects means you can no longer use substitution in a pure functional context.

So, in the Erlang/Elixir view of the world, you program with local immutable state in processes. Process have recusion so that they do have observable state. To know the state, you just have to send a message to the process and ask for what the current "state" is. Each process is a world unto itself and has nothing shared with other processes.

The nice thing about this is that a single process can be cleanly terminated at any time and guarantees that the system as a whole can continue to run predictably. Programming languages that have shared mutable state (Java, Go, Python, Ruby, C#, C++, etc.), or even the possibility to do use  “unsafe” constructs (Rust), (threads) cannot provide this guarantee.

_Interestingly, the decision to have immutability in Erlang was not actually necessary. Processes could have had local mutable state even with processes. The decision to use immutability exclusively in Erlang (and hence Elixir) was done for reliability - again so both humans and computers could understand and execute efficently - local function calls and remote function calls have the same guarantees of immutability._

# Imperative is Simpler

I think a lot of developers with experience in pure functional languages might be surprised with Elixir because often the code looks like an imperative language! Here's an example:

```
file = File.read!("my_file.txt")
strs = String.split(file, " ", trim: true)
subs = Enum.map(strs, fn str -> if str == "tom" do "Tom" else "Tom" end)
joined = Enum.join(subs, ",")
Database.write(joined) #just an example
IO.puts "done"
```

If Elixir is functional, where are the callbacks, promises, awaits, etc? The answer is that in Elixir we don't care - I have no idea if File.read is asynchronous or synchronous and I don't care because I need the result of the file in order to start processing the returned strings from the file and that's all the matters. So we can write code simply and imperatively.

Processes (and threads) were invented for two purposes: 
* To make it possible to write code simply and imperatively (for humans)
* To make it possible to run more efficiently, concurrently or in parallel (for computers)

In my previous blogs https://tomjoro.github.io/2017-02-03-why-reactive-fp-sucks/ some readers have complained that I don't understand the difference between concurrent and parallel because I seem to use them interchangably - there's a reason. In Elixir there really is no difference between concurrent and parallel. Anything that is done with processes can either be run concurrently or in parallel (because of immutability!) - we leave it up to the scheduler to decide andjust write everything in parallel by default, i.e. the question is "do I need the result of the last statement (expression) for the next statement (expression)". 

I digress. Anyways, the point here is that you can program imperatively in a functional language. In an imperative language code is executed from top to bottom, one statement (expression actually) after another. However, Elixir is functional and declarative.

So, while I agree that imperative code is easier to understand (for humans), I just disagree that this implies that it requires mutable constructs.

I know this seems completely backwards compared to most languages -  but that doesn't mean it's not correct. The world is changing and languages like Erlang & Elixir which might have historically been impractical in many contexts, have suddenly become relevant and practical (Why? See my other Blog https://tomjoro.github.io/2017-01-31-world-changed/ _Hint: distributed and parallel are no longer special cases._

So, there is some middle ground between pure-functional languages like Haskell and common mutable state languages like Go, Java, etc.: this middle ground is Elixir - it doesn't require a complete change in your world view. I belive this is what drives the continued growth of Elixir, i.e. there is nothing else in the same category.

We can make simple reliable programs when using processes as first-class citizens and having immutable state - which to me feels like a perfect analogy to the real world. 

# Space and Time 

The most important thing (now) in the future of programming (is) will be _reliability_ and the way to make reliable programs is to make programs that both humans and computers can understand and that work reliably in the real world  - and the real world is distributed in _time_ and _space_.

So, it feels like this philosophy fits my Dr. Whovian view of the world prefectly - now where did I leave my Tardis.

# Disclaimer

Just to set the record straight - Haskell is useful, Go is nice, I love Ruby, C++ is efficient, C# is handy, Java is solid, Rust makes sense, etc. - I've been programming for over 40 years and it's not my intention to bash on other languages - comparisons that evoke emotions promote thinking and discussion :) and so happy coding!

(c) 2021 Thomas O'Rourke