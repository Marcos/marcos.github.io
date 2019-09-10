---
layout: post
title:  "Thread safety and immutability"
date:   2019-09-10
categories: java thread
---

> This post are part of my notes notes abouth the book [Java Concurrency in Practice](http://jcip.net/). 

### Thread Safety

Thread safe class is a class that has the correct behaviour in a single thread or in a multiple thread environment. All classes that works only in a single thread environment are considered automatically a no thread safe class. A class the often works without problems in a multiple thread environment is not considered a thread safe class, because an error in a multiple thread environment can take long time to arise and this is the reason why is so difficult to simulate. This kind of error is defined as eventual fail and mitigating concurrency-related issues through testing is no easy task.

There are some types of concurrency errors: race condition, deadlock, livelock. Concurrency errors exists when an object is stateful, stateless objects are always thread-safe and immutable, but keep in mind that not all immutable objects are stateless and thread-safe, because they can have exactly one state, the initial state. It is a common mistake to assume that synchronization needs to be used only when writing to shared variables and even if an object is immutable and thereby thread safe, the reference to this object may not be thread safe.

### Immutability

The cheapest way to  ensure thread safety is using immutable objects, because they have only one state, which is the initial state. There is no way in Java to formally declare an object as Immutable, so the programmer should be very careful, because even having all field as final, it still could hold a reference to an object that it is not immutable and could be changed after the construction of the object. An object is immutable when it can not be changed in any way after it is constructed.  
It is also better to restrict as much as possible mutable fields. If one object is not totally immutable and has only one or two immutable fields, it still is better than having all fields as immutable fields. It is a good practice to use the final modifier as much as possible.
If an object is immutable, it still could have inconsistency through threads if it is not safely published. One thread could visualize the object while it still is in construction, so it is necessary to ensure a safely initialisation for immutable objects: using volatile, AtomicReference, final fields or fields properly guarded by a lock.
