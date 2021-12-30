---
layout: post
title: Elixir code doesn't look very functional 
image:  /img/blog_function.png
tags: [Functional Programming, Elixir]
---

__this article is still under development__

# Functional programming

Developers who use functional programming languages like Javascript are often surprised when they look at Elixir code because it doesn't look very functional! Instead it looks quite procedural, or imperative. There are just Modules and Functions and to make code work you just call them. Where are all the functional things?

I think the problem is that languages confuse *Reactive* with *Functional*. Just because you have to react to an event doesn't mean you should be passing closures to asynchronous functions!. 

Take for example this article: https://www.stackbuilders.com/news/concurrent-haskell-in-the-real-world

"Concurrent programming feels just a little bit more imperative than the rest of Haskell, because you compose actions that actually mutate something".


# Imperative is Simpler

In an imperative language code is executed from top to bottom, one statement (expression actually) after another. 

For example:

```
file = File.read!("my_file.txt")
strs = String.split(file, " ", trim: true)
subs = Enum.map(strs, fn str -> if str == "tom" do "Tom" else "Tom" end)
joined = Enum.join(subs, ",")
Database.write(joined) #just an example
IO.puts "done"
```

# More processes => less reactive code

If Elixir is functional, then where are the callbacks, promises, awaits, etc? In Elixir I don't care if File.read is run in my process or asynchronous in another process - I need the result of the File.read before I can do the next thing. So I can write code simply and imperatively. What enables this?

Processes (and threads) were invented for two purposes: 
* Wite code simply and _imperatively_ (for humans)
* Run more efficiently, i.e. concurrently or in parallel (for computers)

In Elixir there really is no difference between concurrent and parallel https://tomjoro.github.io/2017-02-03-why-reactive-fp-sucks/. Anything that is done with processes can either be run concurrently or in parallel with other processes because of immutability and local state. Processes are lightweight and are used like as any other language construct. 

Processes use _dependent_ imperative steps to accomplish a task. Imperative code is easy to understand and makes programs more reliable. And you don't need mutation to do this.

_This "style" of programming might have historically been impractical in many contexts, but has suddenly become relevant and practical (Why? See my other Blog https://tomjoro.github.io/2017-01-31-world-changed/ _Hint: distributed and parallel are no longer special cases._

# Async / await 

https://stackoverflow.com/questions/31186354/async-await-where-is-continuation-of-awaitable-part-of-method-performed

Continuations in C# can run in any thread. This is the same approach in C++.

I don't understand this. Now all my logic is broken up to arbitrary places. Why didn't I just write it on another thread in the first place.

# Conclusion

Reactive programming is a nice pattern. However, if you are using Reactive programming as a technique to share threads then you might be overcomplicating the logic.