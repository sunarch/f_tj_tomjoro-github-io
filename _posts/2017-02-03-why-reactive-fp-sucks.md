---
layout: post
title: It's Windows 3.1 All Over Again
image:
tags: [Elixir, Erlang, Functional programming, Rails, node.js, Javascript, Go]
---

A little rant. I'll add some diagrams to this article when I get some time...

# Rewind the clocks to Windows 3.1

In 1992 with Windows 3.1 we had this thing non-preemptive multitasking, i.e. "Cooperative Multitasking".
Cooperation is a funny term, it should actually be "Programmer Controlled Multitasking" because
you don't have to be cooperative if yo don't want to :)
There was an event loop that was controlled by the Windows system. You got message and you did work on those messages.
We repeated the rules: don't do any long computations, and if you have to do that,
break it up into a state machine that is then driven by messages. And most of all don't crash,
because you will make users very unhappy.  

In 1994 Microsoft started working on Windows NT. This was great, we had true pre-emptive
multitasking. But thread creation and context switching was really slow. We could only
spawn like 5 threads a second. So the "Fiber" was invented. This allowed threads to be
shared. But threads could not be pre-empted.

Nice. But what happened to fibers, was that the same problem that we had with
non-preemptive multitasking happened all over again. Eventually thread creation and
context switching got fast, and fibers were relegated to 'special problems'.

Reactive programming is also known as non-blocking I/O, but it is not the same thing.
Reactive programming has these characteristics:

 * It uses a central event loop to dispatch control.
 * It uses run-to-completion, i.e. there is no preemption.

# Present Day and Reactive Programming (Javascript and Go)

Operating systems are built around processes for reliability. Each (user mode) process is protected
has it's own memory and should not crash the entire system. This works.

So, why the need for Node.js and 'reactive' style programming? Why do Goroutines
share the same thread? Node.js works just like Windows 3.1. There is an event loop,
each time you run, it runs to completion, meaning that the code that is running cannot be interrupted.
So you have the same rules as we used back in Windows 3.1.

Goes does better, but it's still unable to pre-empt your code. Go can do a context switch
when you make a system call, but only if you call something that allows a context switch.
This means I need to know which calls would allow pre-emption, and which ones would not.
From the Go team:

```
It is by design. There are no plans to make the scheduler fully preemtive, in normal situations, this is not a problem.

The traditional work around is to unroll the loop and add a Gosched.

```
In Windows 3.1 we had this thing called "Yield()" which now has become a Javascript callback or "GoSched()".

Most programmers would agree that a sequential program, one that says do a) and then
b) and then c):

```
  10 A = 1
  20 A = A + 1
  30 PRINT "HELLO " + A
  40 PRINT "YES"
  50 GOTO 20
```

is easier to understand than one that breaks up the processing sequence
just because b takes too long (and we don't want everything else in the program to pause).

```
  10 A = 1
  20 A = A + 1
  30 PRINT_NON_BLOCKING "HELLO" + "A", when done callback { LINE 40 }
  40 PRINT "YES"
  50 GOTO 20
```
In this example, the PRINT statement is very slow and uses I/O. So to avoid us blocking
the entire program, we have call a non-blocking routine PRINT_NON_BLOCKING, and give it a callback on what
to do when it is complete.

A common quiz for Javascript is to ask: which is printed first? "YES" or "HELLO3"? It's
always going to be "YES" first, because there is no way for the callback even to try to
run until you _give up the thread, i.e. run to completion_.  Actually, in this sillly
example the callback can never run, because our code never returns :)

This means that the programmer must think about the scheduling in addition to the logic,
even if the piece of code he/she is working doesn't need to be 'responsive'.
Although, this does often result in faster executing programs because the
programmer themselves optimize the context switching (i.e. where the callbacks are in Javascript).

If we wanted to do this in Elixir it would work like this:

```
  10 A = 1
  20 A = A + 1
  30 PID = PRINT_NON_BLOCKING "HELLO" + "A"
  40 PRINT "YES"
  35 WAIT FOR MESSAGE ON PID
  50 GOTO 20
```

I remember an article (where is it) that showed that on Java using blocking threads was faster or basically
equivalent to Non-blocking I/O for most "things". But that was up to 10K connections and
processors were slower back then. Nowadays we have to do much better!

# Reactive programming, Rails, and Node.js

Why is Node.js so popular, and why does it work so well? Node.js was created because
threads are once again too slow & expensive. Rails uses threads, and if you create 2000 threads
you will find your system performance degraded. But the style of programming in Rails
is great. Very clear sequential code, easy to understand. And if you use JRuby you
get true multi-tasking.

Popular frameworks like Play on Scala achieve performance by mixing threads and
reactive programming (callbacks). And if you read the text on the Play website, it
says quite clearly that the reason that it has reactive routines, i.e. shares requests on the same
thread, is primarily for performance reasons. But watch of for nasty problems, like
remember that you can't do anything expensive in that callback or other things will stop
that share the same thread.

# What if Processes Were Not Expensive?

What if processes were not expensive? Would you use them instead of callbacks (reactive)?

Enter Elixir. In Elixir processes are not expensive and they are protected in the
same way operating system threads are protected. Of course you can have tons
of asynchronous stuff happening, but if all your processes are waiting on I/O, for example
waiting for a database request to complete, they all would be blocked.
But blocking is not the problem.
 - *it's the expense & overhead of the process (or thread)*.

 Node.js (Reactive programming) tell us that blocking is the problem that is being solved.
 Blocking is not the problem.

I've seen several articles about reactive programming with this kind of statement:
 `rather than hog a thread even while it’s blocked`. and `don't hog the thread`.
The first thing that comes to my mind is, for example, _"what else am I supposed to do when
I'm waiting for a result from the database"?_ I can't do anything else until I have
that result. I need the result to make give to a template to give to the renderer, etc.

What are you hogging and from whom are you hogging it?
These statements mislead coders into thinking that blocking is bad, when really
the problem is that threads are a limited resource and context switching (sharing)
(between threads) is expensive. Just for clarity: _there is nothing bad about blocking._
If your code needs to wait for a result, then it needs to wait for a result.
You can transpose this into a callback or promise, but you haven't changed
the logic.

In Elixir (thanks to Erlang) yo have true pre-emptive multi-tasking, and processes
are really efficient and inexpensive. You don't have to think about where
to break up your code because of long-running tasks etc. And an errant processes
can simply be shut down and restarted without taking down the entire system.

In Elixir most I/O functions block, so your code stays sequential. If you don't
want to wait, then don't. Why might you not want to wait? Maybe you want to start
2 or 3 things in parallel, or maybe there something useful you can do before
you get your result from the blocking function. Any blocking function can easily
be made non-blocking by simply wrapping it in a spawn() so that it
starts asynchronously. Note here that the reason for not blocking is because
_you can do something useful while you are waiting_, and has nothing to do
with hogging or stealing resources.

So, yes, often Elixir is slower than Go because sometimes Erlang will interrupt a
process and give control to another process even though it would have been more efficient not to do so.
But this is all about fairness. Erlang wants all processes to play nicely, so
no one process is allowed to hog the system.

_Usually, at this point someone will say "Yes, that may be true, but between
processes you're going to have to do a lot of copying - sharing memory
between threads is much more efficient"._ This is true, but sharing memory
is also a lot more dangerous and couples the code together (GC?).

You can read about that in one of my other posts:
 [The world has changed]({{ site.baseurl }}{% link _posts/2017-01-31-world-changed.md %})_

 _Short summary: Using Elixir doesn't work well for everything,
 but you'd be surprised how often it does_

# Conclusion

It's no wonder why programmers who have enough experience to remember the good
old days of Windows 3.1 are not the biggest fans of Node.js. _A tuned Windows 3.1
application often outperformed pre-emptive systems of the day!_

Languages like Elixir offer both fully pre-emptive sequential programming
(and are functional too), performance and protection. You get to focus on
making your algorithms and logic, and not the scheduling - let the scheduler
do the scheduling and you get on with on with your program. This is a joy to a programmer (or coder as we call them nowadays).

# Afterward

Some of you web developers might be thinking: "yeah, true, but you need to
keep things responsive for the user, so you do have to think about scheduling".

Very true. However, a complete application, whether it is a desktop application or
web application, usually does more than just respond to user clicks. There are long running
reporting jobs, talking to external REST API services, etc. In Rails or Node.js it's
typically an all or nothing decision about what should be run on background tasks and
requires using external databases like Redis, and external monitoring tools.

Elixir provides a much more predictive environment for your code. You can easily refactor
code that was previously synchronous to run asynchronously with little changes
to your sequential logic. Nothing prevents you from using Redis, etc., but use it only when you
need a persistent task, not just because something needs to run asynchronusly.
