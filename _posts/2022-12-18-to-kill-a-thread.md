---
layout: post
title: To Kill a Thread 
image:  /img/worlds.png
tags: [Functional Threads, Safety, Programming, Elixir]
---

Most of my blogs are long and rambling, so I’ll keep this one short.

# How to Kill a Thread

Scene:  There are hundreds of threads running, and one specific third party library goes bonkers and consumes too many resources, is hung, or otherwise behaves badly - I want to take action without restarting my entire program.

Question
* How to safely kill a thread in Java? (Or Go, Rust, C++, Scala, etc)?

Answer
* **No**

# Don't Even Try

Java struggled for years with method to safely kill an executing thread. In fact the original Java had a method to kill a thread. But eventually everyone realised that it’s just too darn unsafe and can leave your system in an unstable state. Instability is bad, because it means all bets are off, including security, etc. The only answer is to drop everything and start over.

It's interesting/amusing that Rust followed the same path - didn’t they learn from the struggles of Java? Rust had a kill thread and now it’s been deprecated because it’s not safe, and it' can't be made safe (link to long discussion thread).

You can save yourself years of work: If you're making a language with shared mutable state, then the answer is no.  Shared mutable state means one thread access the memory of another thread somehow, through any mechanism. Doesn’t matter if the mechanism is protected, or requires careful declarations, etc. If the mechanism exists in any form it’s a non-starter. 

There’s actually a proof for this somewhere - I read it a long time ago and it made my brain hurt, but I remember the conclusion and that’s what is important.

# Killing Safely

So, where do we see this principle in action: Operating systems. Most applications run isolated from other programs, and therefore can be shut down safely. That’s because the applications don’t share memory.

There are some computer languages that follow this same principle. Yes, Erlang and Elixir. Each process (thread of execution) in Erlang or Elixir running on the BEAM Virtual machine *cannot share memory with another process. Therefore, there is a safe way to kill a process - end of story. 

# Lord of the Rings Footnote

* The word cannot is chosen carefully here.

In Lord of the Rings, Gandalf actually says “You cannot pass”, whereas people quote from a movie line with “You shall not pass!”, or "None shall pass!" You cannot pass is actually a stronger statement which indicates that it is impossible to pass, but "shall not" sounds so much wiser and more grandiose. 

*“You cannot pass," he said. The orcs stood still, and a dead silence fell. "I am a servant of the Secret Fire, wielder of the flame of Anor. You cannot pass. The dark fire will not avail you, flame of Udûn. Go back to the Shadow! You cannot pass.”*


(c) 2023 Thomas O'Rourke