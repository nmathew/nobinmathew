---
author: Nobin Mathew
comments: true
date: 2016-01-23 04:53:59+00:00
layout: post
slug: lock-less-programming-and-transactional-memory
title: Lock less programming & Transactional Memory
wordpress_id: 7
tags:
- LockLess
- programming
- Transactional
- LLSC
- CAS
- Memory
- Atomic
---

Some good articles collected from Internet on lock free programming and Transactional Memory

[IBM’s new transactional memory: make-or-break time for multithreaded revolution](http://arstechnica.com/gadgets/2011/08/ibms-new-transactional-memory-make-or-break-time-for-multithreaded-revolution/)

ABA Problem:  
[Introduction to lock-free/wait-free and the ABA problem](http://www.hergert.me/blog/2009/12/25/intro-to-lock-free-wait-free-and-aba.html)   
[ABA Prevention Using Single-Word Instructions](https://www.research.ibm.com/people/m/michael/RC23089.pdf)

Intel's Haswell RTM (Restricted Transactional Memory):  
[Transactional memory going mainstream with Intel Haswell](http://arstechnica.com/business/2012/02/transactional-memory-going-mainstream-with-intel-haswell/)  
[Hardware Lock Elision on Haswell](http://brooker.co.za/blog/2013/12/14/intel-hle)  
[Restricted Transactional Memory on Haswell](http://brooker.co.za/blog/2013/12/16/intel-rtm.html)  
[Lock-free Data Structures. Basics: Atomicity and Atomic Primitives](http://kukuruku.co/hub/cpp/lock-free-data-structures-basics-atomicity-and-atomic-primitives)  
[Lock-free Data Structures. 1 — Introduction](http://kukuruku.co/hub/cpp/lock-free-data-structures-introduction)

[Understanding Atomic Operations](https://jfdube.wordpress.com/2011/11/30/understanding-atomic-operations/)  
[Understanding Memory Ordering](https://jfdube.wordpress.com/2012/03/08/understanding-memory-ordering/)

[LL/SC Details from Computer Architecture: A Quantitative Approach](https://books.google.co.in/books?id=pqYl3SWkA64C&pg=PA240&lpg=PA240&dq=LLSC+instruction+implementation&source=bl&ots=0Q8SDcCERJ&sig=IErD0Zm0nZPBTgt9f-NTU4jiBIA&hl=en&sa=X&ved=0ahUKEwjA9paPlL_KAhWBj44KHZNuBmcQ6AEITzAI#v=onepage&q&f=false)

Jeff Preshing's blogs:  
[An Introduction to Lock-Free Programming](http://preshing.com/20120612/an-introduction-to-lock-free-programming/)  
[An Introduction to Lock-Free Programming | Hacker News Discussion on above article](https://news.ycombinator.com/item?id=4099834)  
[Acquire and Release Semantics](http://preshing.com/20120913/acquire-and-release-semantics/)  
[Memory Barriers Are Like Source Control Operations](http://preshing.com/20120710/memory-barriers-are-like-source-control-operations/)  
[Weak vs. Strong Memory   Models](http://preshing.com/20120930/weak-vs-strong-memory-models/)  
[Memory Ordering at Compile Time](http://preshing.com/20120625/memory-ordering-at-compile-time/)
