---
layout: post
title: Is Functional Programming Is a Lie? 
image:  /img/blog_function.png
tags: [Functional Programming, Elixir]
---

# Background

Recently I read some articles with catchy headlines like: Pure Functional Programming is a Lie, Functional Programming is not Real, etc.

A deeply philosophical statement posed more as a question, and something I’ve pondered now and then - especially since I started using Erlang/Elixir, which will become apparent :)

Given that these articles are often written by experts, i.e. developers or computer scientist who have years of experience in pure functional programming (Haskell mainly) and have often been strong advocates - I wonder why the change of heart? 

Is this a case of "if you can't beat 'em, join 'em"? - If I must work in an impure language such as Go, Java, Javascript, Ruby, Python, etc. then do I need a philosophical justification for this change (so I don't feel like I have just given up and followed the crowd).


# Functional Programming is a Lie

The premise of these articles is that pure functional programming doesn't sufficiently represent what happens in a real world; the real world being as realized by computers. And therefore pure functional programming is a lie. Basically concluding that although programming in pure functional programming makes mathematical sense, in practical situations they can become overly complex and difficult to understand.

The argument is made that real computers have state, i.e. you store things in memory locations. The value 42 is stored somewhere in memory, maybe at adress 0x10000000. And so I can update and change that value by replacing 42 with 43 and now the value 43 is at 0x10000000 (mutation). In pure functional programming values don't magically change - they stay the same and only functions produce results (immutable). Specially, the argument is that real computers have mutable state, so therefore programming languages should have mutable state. 

I don't know why it's important for a programming language to actually represent how they are implemented in physical computers. As far as I am concerned our goal as programmers is primarily to write programs that both humans and computers can understand and that work reliably. _Reading bewteen the lines: obviously I do care otherwise I wouldn't have written this blog._

I’d agree that _pure_ functional languages are not very practical, but don't throw the baby out with the bathwater. Just because you drop the "pure" part of "pure functional programming" doesn't mean you should give up an all the tennants (immutability, substitution (no side effects), first-class functions, etc.) _The definition of "pure" is difficult, but generally pure functional languages do not allow for side effects (in addition to all the other "functional" things)._

You can do functional programming in many languages, for example Javascript is very functional. However, Javascript also has mutation which it has in common with most other "practical" languages like Java, Go, C++, C#, Ruby, Python, Rust, etc. But there are a _few_ languages that are in a different category: not-pure (allow side-effects), immutable, and functional.

Elixir is the most widely used computer language that is 1) functional, 2) does not allow any mutation, and 3) allows for side effects. That'a a bit of a hidden joke because I really don't know any other languages in this category -  maybe readers of this Blog can help me out. However, just because Elixir might be unique in these regards - it's still a very practical language and is widely used (arguably).

# Mutation

This Blog is not about X-Men, but we do talk a lot about x.

Reliable programming languages (and even languages that have mutable constructs) use immutability to make programs easier to understand for both humans and computers.

For a primer: here's an example of immutable in action in Elixir:
```
  map = %{test: 1}
  Map.put_new(map, :fun, 2)
  IO.inspect map   

  # Output: %{test: 1} -- the value of map did not change, but rather 'Map.put_new' returned a new Map (which we stupidly ignored) 
```

This is how can do it:
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
* In a unpure functional language (one that allows side effects), sometimes 25 comes out as and maybe other times it comes out as 3.1415. This is called a side effect. (for example getTime() will always yeild a different result)
* In a language with mutation, sometimes 25 comes out 3.1415 and _sometimes x is magically changed to 52_ 

Of these three statements, the mutation is the hardest one to understand. Because the black box can no longer be treated as a black box. We must look into the black box to understand what it did to x. 

# Mutation and Global State Are Lies

The real world is immutable. Real computer systems are immutable. 

What’s missing from mutable view of the world is _time_ and _distribution_. _The real world is immutable when viewed at any point in time_ - the past can’t be changed. Futhermore, there is no such thing as global state in the real world (you don’t know what’s happening somewhere else - it has to be communicated), or any computer system. This is especially true nowadays as computing has become increasingly distributed and parallel (multi-core systems which each have their own cache, memory, etc.). Trying to understand global state is an artifical construct that Einstein proved is wrong - there is only observable state.

Algorithms increasingly must be understood in the context of parallel multiprocessor/distributed systems. Entire books discuss algorithms and big O complexity without ever mentioning that all bets are off when you introduce parallelism because it becomes mathematically “difficult”. If one introduces threads sharing mutable state, it becomes basically intractable (see “The Problem with Threads” from Lee).

State arises from recursion in time. Original computer memories were exactly this, delay lines, even CRTs that simply played back a state put into them. Logic gates are the same, they model state recursively. State is observable at a point in time and is therefore immutable. Any system, when viewed from the outside, will have observed state, even if internally it has immutable state which is functionally defined (with recursion).

Given this view, then one must accept that functions (and real systems) can have side effects, i.e. given same inputs a function can return different results. A good example is get_time(). This function returns a different result each time you call it because it might be the result of an external system and not part of the local state -  Introducing side effects means you can no longer use substitution in a pure functional context.

So, in the Erlang/Elixir view of the world, you program with local immutable state in processes. As a system you communicate with other processes (by sending messages) that also have local and immutable state  - but process have observable state, and so does the system. Another advantage of this is that a single process can be cleanly terminated at any time and guarantees that the system as a whole can continue to run predictably. Programming languages that have shared mutable state (Java, Go, Python, Ruby, C#, C++, etc.), or even the possibility to do use  “unsafe” constructs (Rust), (threads) cannot provide this guarantee.

Interestingly, the decision to have immutability in Erlang was not actually necessary. Processes could have had local mutable state without breaking the premise of actors, i.e. that actors have their own state and only communicate with the outside world by either sending or receiving messages. The decision to use immutability exclusively (you can't mutate) in the langauge was done for reliability - again so both humans and computers could understand and execute efficently respectively, for example local function calls and remote function calls have the same guarantees of immutability.

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

In my previous blogs some readers have complained that I don't understand the difference between concurrent and parallel because I seem to use them interchangably - there's a reason. In Elixir there really is no difference between concurrent and parallel. Anything that is done with processes can either be run concurrently or in parallel (because of immutability!) - we leave it up to the scheduler to decide and by default just write everything in parallel by default, i.e. the question is "do I need the result of the last statement (expression) for the next statement (expression)". 

I digress. Anyways, the point here is that you can program imperatively in a functional language. In an imperative language code is executed from top to bottom, one statement (expression actually) after another. However, Elixir is functional and declarative!

So, while I agree that imperative code is easier to understand (for humans), I just disagree that this implies that it requires mutable constructs.

I know this seems completely backwards compared to most languages -  but that doesn't mean it's not correct. The world is changing and languages like Erlang & Elixir which might have historically been impractical in many contexts, have suddenly become relevant and practical (Why? See my other Blog "The World Changed..." Hint: distributed and parallel are no longer a special cases.)

We can make simple reliable programs when using processes as first-class citizens and having immutable state - which feels like a perfect analogy to the real world.

# Space and Time 

The most important thing (now) in the future of programming (is) will be _reliability_ and the way to make reliable programs is to make programs that both humans and computers can understand and that work reliably in the real world  - and the real world is distributed in _time_ and _space_).

So, it feels like this philosophy fits my Dr. Whovian view of the world prefectly - where did I leave my Tardis.

# Disclaimer

Just to set the record straight - Haskell is cool, Go is nice, I love Ruby, C++ is efficient, C# is handy, Java is solid, Rust makes sense, etc. - I've been programming for over 40 years and it's not my intention to bash on other languages - Elixir is differerent (See my Blog) and comparisons that evoke emotions promote thinking and discussion :) 

(c) 2021 Thomas O'Rourke