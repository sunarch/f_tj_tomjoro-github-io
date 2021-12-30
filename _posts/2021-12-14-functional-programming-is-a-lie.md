

As more computer languages adopt functional constructs, e.g. closures are now available in Java, C++, etc., it seems pure functional languages have been pushed into a corner. 

Recently I've even read comments that Pure Functional programming just doesn't work in practice because real computers "don't work that way". By "That way", the point is made that real computers that execute programs have mutatable state, and if you don't have mutatable state then you're just making things very difficult for both execution engines and people who try to comprehend the programs. 

This seems to be borne out by the trend to

This all feels very polarizing - some developers feel the need for deeper justifications when choosing a computer language - it can't just be "follow the crowd", or "it's easier to find examples". But do we need to bring _reality_ into the discussion? 

And are there really only 2 choices? Either we reject mutability and program in a pure functional language, or we just accept mutable reality and join the crowd? 

I've thought about this question sometimes, especially since I started using Elixir (5 years now), when trying to build a mental framework for categorizing languages.

## Pure functional languages **

In the early days of computer science, a lot of work was done to try to use math to prove program correctness. Think about that - it would mean that when you delivered your program to a customer you could say "this is provably correct so there cannot be any bugs by definition", unlike today when we say "it has 80% test coverage and worked when I tried it" ;) 

There's no exact definition of fuctionally pure purity, but generally it's these things:
* Immutable state, 
* No side-effects (substitution) 
* First class functions (functions can treated as data)
* etc. 

Javascript is a functional language. Why isn't it a "pure" functional language? 

* Javascript fails purity because it allows for mutation of state. Actually, most mainstream languagse fail for this same reason: Java, Go, C#, Rust, Python, Ruby, etc. However, almost all of these languages have, or have recently, started to use a lot more functional and immutable constructs. 
* Rust fails because it has unsafe modes (you might argue that if you don't use unsafe then it is safe, but that's besides the point - we're talking about languages not usages)
* Erlang and Elixir fail this test because they are impure (they allow side effects)

So are there only two categories? Pure functional languages, e.g. Haskell, OCaml, and everything else that can do anything functional (Java, Rust, C++, Ruby, Javascript, etc.)?

Read on to discover the shocking truth! ;)

## Functional languages and Pure functional languages

Take a look at this list:

https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Pure

The list of functional languages has only two categories: *Pure* and *impure*.

![Elixir]({{ site.url }}/img/languages.png)

The list of "Functional Languages", or languages that have functional constructs is very long nowadays! However, the list of *pure* functional languages is a lot shorter, e.g. Haskell, Ocaml, etc.

Where in this list are the _Languages with immutable data_? Is mutation/immutable just a minor detail? ! ?

I even tried searching Stack Overflow (I use it very seldomly nowadays) - "Computer Languages with Only Immutability". Seems most people misunderstood the question, __or fail to believe it is possible to understand that there even is such a category__. 

Anyways, there are at least a couple of widely used languages that have immutability _only_. _The key word here is the word "only", which means "there is no unsafe mode, there is only immutable"._
* Haskell
* Erlang / Elixir

Here's some drawings I made that focuses on only Pure and Impure as defined by 1) Purity (no side effects), and 2) Mutable (or immutable) data:

### Only Pure and Impure
This is the categorization on Wikipedia:

![Elixir]({{ site.url }}/img/pure_impure.png)

But where is Erlang and Elixir in this drawing???

### Elixir and Erlang 

Elixir and Erlang are impure, yet they do have immutable (only) data.

![Elixir]({{ site.url }}/img/erlang_middle.png)

## Category: Impure, yet immutable

![Elixir]({{ site.url }}/img/erlang_category.png)


Here's what the list should look like:

Functional languages:
* Pure functional languages with immutable data
* Impure functional languages with immutable data
* Functional languages

## But "Mutation is a reality, the correct approach is disciplined mutation"

 I can hear a thousand voices calling out that: Yes, but Rust, Clojure, etc., have immutable data constructs and that if you just keep your mutations in a box then you're ok.
 
"Mutation is a reality, the correct approach is disciplined mutation. "https://news.ycombinator.com/item?id=18044656

Mutation is like a virus and has severe consequences in a language: if you allow any possibility to mutate data in any way (like "unsafe" mode) then the language looses the ability to provide certain guarantees as we will see in the next sections (and add threads to mutation and you have a recipe for nondeterminism) - https://www2.eecs.berkeley.edu/Pubs/TechRpts/2006/EECS-2006-1.pdf

The main issue here is that language designers often try to solve _all problems_ with a single language. Erlang and Elixir have a different answer: __If you need mutation use another language__. Seriously, that's not that different from "box mutation in unsafe mode" and for the record: most programs don't need mutability for anything other than _performance_, and that performance gained by using mutation is dimished when you consider highly parallel and concurrent systems (which is pretty much everything nowadays). https://tomjoro.github.io/2017-01-31-world-changed/

# State

What is mutable state/data (or immutable) anyways and how does it relate to the real world?

## Mutable state/data 
I put 99 beers on the shelf. The shelf now has 99 beers. Hence, the real world has state.

In programming terms I take a variable "x" to be the shelf, and I give x a value of 99. If I take one away then x is 98 beers. The shelf has state and is mutable.

This is the same way computer memory works. I store value in a register or in a memory location and it's stays there. 

## Immutable state/data
Pure functional languages don't allow mutation. In our beer analogy, the shelf with 99 beers can never be anything else. If you want a shelf with 98 beers, you'll need a _new_ shelf because the shelf with 99 beers _cannot_ mutabe changed (i.e. mutate).

This seems impractical (think of the garbage collection!), and difficult to understand. 

In a pure functional language this concept of immutable state is taken even one step further - we need to know how the state was changed. In a pure functional language state changes are realized with functions which can be seen and reduced - which can get very complex as the entire state of the world (the program) needs to be traceable to every action that caused a change. 

##  Real computers / real world

Does the real world have mutable state? Certainly seems like the mutable state matches our _view_ of the world better and I can't argue with that -- this is the "functional programming is a lie" bit.

But pure functional programming does match mathematics. That means that it's easier to prove what happened in a mathematical and conclusive fashion. In pure functional languages beers don't just disappear - there's a reason that can be traced! (very useful!).

I don't know why it's important for a programming language to actually represent how they are executed in physical computers. As far as I am concerned our goal as programmers is primarily to write programs that both humans and computers can understand and that work reliably.

# Mutation is an evil

Why all the concern about mutation? I thought mutation is easier to understand? It might might be on the surface, but let's go a bit deeper.

Reliable programming languages (and even languages that have mutable constructs) use immutability to make programs easier to understand for both humans and computers.
* C++'s standard library uses immutability for improving reliability and concurrency
* Clojure uses it for the same reason
* Rust uses it for the same reason

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

When I was first learning Elixir I had to find a deep justification for immutability. I could turn to Computer Science (everything is expressible with recursion) or math for justification. But what about the real world and real computers? If I looked at the problem it really seemed that the real world had mutation. And then I realized what I was missing.

## Time
Remember the beer analogy, i.e. that the shelf has state, that computer memory has state? You have to understand this:  

    The real world is actually _immutable_. 

How can this be? 

    It's because we forgot to consider _time_ in our world view - and time does exist!

The real world is immutable when viewed at any point in time
* the past can’t be changed and 
* state is only definable by observation (at a time).

State arises from recursion in time. When you look closely at how computer memories actually work, you can see this recursion _even at the level of the logic circuits that implement computer memory_

And this is confirmed historically - original computer memories were things like mecury delay lines, even CRTs, as recursed and repeatedly played back a signal put into them. In the past this was more outwardly apparent, but now this recursion is hidden in logic circuits.

 Here's a logic diagram of a flip-flop. A flip flop is a 1 bit computer memory and is how RAM is built in a _real_ computer.

![Elixir]({{ site.url }}/img/flipflop.png)

You don't have to understand this diagram, but notice how the lines feedback on themselves. This is recursion in time. There's all the proof you need: state arises from recursion in time :) . 

And since computer memory is built with recursion, I don't see that computers have any difficulty in executing this.

The fact that time is real has further implications. 

## Distribution 

If time is real, it also prevents us from knowing the state of the entire world at any point in time. We can only know our own local state (or nearby state). This is distribution and the source of side effects.

If you accept distribution then you just have to accept side-effects, because state is going to change in other "parts" for reasons unbeknownst to our local reference. In the beer example, you have to go to the shelf to check for beers as it is impossible for you to know without checking.

Introducing side effects means you can no longer use mathematical descriptions of the entire system (relativity - very Einstein)! What are these "parts", or local references?

In the Erlang/Elixir these independent "parts" that each have their own local and immutable state are *processes*. Processes have observable state, which can be observed from the outside by sending a message to the process and asking it for the state and the process in turn sending you a response. Each process is a consistent world unto itself and has nothing shared with other processes.

This matches the real world - If you can’t know what’s happening somewhere else - then state must be communicated to be known. _This is especially true nowadays as computing has become increasingly distributed and parallel even on a small scale. For example, in multi-core computers each core has it's own dedicated cached memory and so getting a value from another core requires requesting it and copying._

The nice thing about this is that processes can't interfere with other processes by definition, and hence can be cleanly terminated at any time. This in turn guarantees that the system as a whole can continue to run predictably regardless of the failure of individual parts. Programming languages that have shared mutable state, at any level and through any mechanism, cannot provide this guarantee. The only way to provide this guarantee is completely ban shared mutatable state (processes can only communicate by messaging).

In a way, Elixir and Erlang processes work similar to how an operating system works: there are processes that are isolated and have their own memory - that's why computers crash less often than our software! But Erlang goes a lot further and introduces consistent communication concepts, error handling, location (network) transparency, etc., resource control, etc.

# Is Pure Functional Programming a Lie?

Back to the original question: Is pure functional programming a lie? Saying pure functional programming is a lie is like saying math is a lie. It's not a lie, but it doesn't always map well to the real world of computers -  but that's not because of immutability, but instead stems from not taking *time* and *distribution* into account.




# Elixir's Category 

There is some middle ground, a category, between *pure-functional languages* and *impure languages with mutable state* -- and that middleground is a category that Elixir owns. I belive this is what drives the continued growth of Elixir. Elixir is not a "solve all problems" language, but it does work efficiently in more situations that many developers ever thought possible.

Giving this category a name is difficult: is it "impure functional with distrubted and immutable state" or "functional, dynamic, safe language", or ?. How about just plain "Reliable"?

_How is reliability defined? Reliable computing is correct functioning even if individual hardware or software components fail for any reason. The definition also includes secure computing which is about intentional attaks on your system._

Erlang (and hence Elixir) and the BEAM virtual machine that it runs on, first and foremost goal is to be reliable. Every decision about the language (and BEAM execution environment) as it evolved has prioritized having reliability as the most important criteria. 
* Simple code is reliable code because people understand it
* Imperative code is simple to understand  (https://tomjoro.github.io/2021-12-30-elixir-doesnt-look-very-functional/)
* Immutability is simpler and can be location transparent

Many other languages try to be fast and versatile at the detriment of reliability, and many functional languages even have escape hatches for programmers who dare to use unsafe features (usually to get more performance). But prioritizing reliability actually works in practice: Elixir it is fairly fast, extremely fair, quite versatile, scalable, easy to undestand and most importantly reliable. 



# Disclaimer

Haskell is useful, Go is nice, I love Ruby, C++ is efficient, C# is handy, Java is solid, Rust makes sense, etc. - I've been programming for over 40 years and it's not my intention to bash on other languages - comparisons that evoke emotions promote thinking and discussion :) and so happy coding!

(c) 2021 Thomas O'Rourke

