---
layout: post
title: It's Windows 3.1 All Over Again
image:
tags: [elixir, erlang, functional programming, rails, node.js]
---

A little rant. I'll add some diagrams to this article when I get some time...

# Rewind the clocks to Windows 3.1

In 1992 with Windows 3.1 we had this thing non-preemptive multitasking. There was an event loop
that was controlled by the Windows system. You got message and you did work on those messages.
We repeated the rules: don't do any long computations, and if you have to do that,
break it up into a state machine that is then driven by messages. And most of all don't crash,
because you will make users very unhappy.  

In 1995 Microsoft started working on Windows NT. This was great, we had true pre-emptive
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

# Present Day and Reactive Programming (callbacks)

Operating systems are built around processes for reliability. Each (user mode) process is protected
has it's own memory and should not crash the entire system. This works.

So, why the need for Node.js and 'reactive' style programming? Node.js works just like
Windows 3.1. There is an event loop, each time you run, it runs to completion, meaning
that the code that is running cannot be interrupted. So you have the same rules as
we used back in Windows 3.1.

Most programmers would agree that a sequential program, one that says do a) and then
b) and then c), is easier to understand than one that breaks up the processing sequence
just because b takes too long and requires the knowledge of scheduling to be left up
to the programmer. Often though, this does result in faster programs because the
programmer themselves control the context switching by yielding at the appropriate
times.

I remember an article (where is it) that showed that on Java using blocking threads was faster or basically
equivalent to Non-blocking I/O for most things. But that was up to 10K connections.
Nowadays we have to do much better!

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

What if processes were not expensive? Would you use them instead of callbacks?

Enter Elixir. In Elixir processes are not expensive and they are protected in the
same way operating system threads are protected. Of course you can have tons
of asynchronous stuff happening, but they all need to wait for I/O or disk they
would all block. But blocking is not the problem - *the expense & overhead of the process is*

With true pre-emptive multi-processing you don't have to think about where
to break up your code because of long-running tasks etc. And an errant processes
can simply be shut down and restarted without taking down the entire system.

# Conclusion

It's no wonder why programmers who have enough experience to remember the good
old days of Windows 3.1 are not the biggest fans of Node.js.

Languages like Elixir offer both fully pre-emptive sequential programming
(and are functional too), performance and protection. You get to focus on
making the algorithms and logic, and not about the scheduler. This is a joy to
a programmer (or coder as we call them nowadays).

# Afterward

Some of you web developers might be thinking: "yeah, true, but you need to
keep things responsive for the user, so you do have to think about scheduling".

Very true. However, a complete application whether it is a desktop application or
web application usually does more than just respond to user clicks. There are long running
reporting jobs, talking to external REST API services, etc. In Rails or Node.js it's
typically an all or nothing decision about what should be run on background tasks and
requires using external databases like Redis, and external monitoring tools.

Elixir provides a much more predictive environment for your code. You can easily refactor
code that was previously synchronous to run asynchronously with no change to your
core logic. Nothing prevents you from using Redis, etc., but use it only when you
need a persistent task, not just because something needs to run asynchronusly.
