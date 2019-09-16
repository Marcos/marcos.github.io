---
layout: post
title:  "64-bits variables and atomicity"
date:   2019-09-16
categories: java threads
---

> This post are part of my notes notes abouth the book [Java Concurrency in Practice](http://jcip.net/). 

### Nonatomic 64-bit Operations
The JVM always requires reading and writing varibales to be atomic, but there is an exception for 64 bits variables, like double and long. On 64 bits variables, the JVM treats the read and write as two separate 32 bits operations. It implies that one thread could be reading a value different of the one written by another thread. There are two ways to ensure atomicity for mutable shared long and double variables, using a lock or declaring the variable as volatile.

On this code, the long number is not thread safe, because of the 64-bit nonatomic operations.
```java
public class LongNotAtomic {

  private long longNumber = 0;
 
  public void setLongNumber(long n) {
    longNumber = n;
  }
 
  public void getLongNumber() {
    return longNumber;
  }
}
```

### Volatile Variables
Volatile variables are not cached in registers or in caches where they are hidden from other processors. When declared as volatile the variable is always read from the main memory, so a read of a volatile variable always returns the most recent write by any thread. 

Even being considered a lighter-weight synchronization mechanism than synchronized blocks, volatile variables still are less safe than a lock. Locking can guarantee both visibility and atomicity, volatile variables can only ensure visibility.

On this code, the long number visibility is guaranteed by the volatile modifier:
```java
public class LongAtomic {

  private volatile long longNumber = 0;
 
  public void setLongNumber(long n) {
    longNumber = n;
  }
 
  public void getLongNumber() {
    return longNumber;
  }
}
```

On this code, the long number atomicity is guaranteed by the lock on synchronized modifiers:
```java
public class LongAtomic {

  private volatile long longNumber = 0;
 
  public void synchronized setLongNumber(long n) {
    longNumber = n;
  }
 
  public void synchronized getLongNumber() {
    return longNumber;
  }
}
```