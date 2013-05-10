---
layout: post
title: "Playing with Java Timestamps"
date: 2012-10-19 12:31
comments: true
categories: 
- software
- Java
- Work
---
The shortcomings of Java's native date and time support is well known. Asking a developer about a date/time problem in Java is a good way of judging their exposure to building systems with it. For new projects, Joda time is excellent, but there's plenty of native code out there that uses the standard Java APIs. Today I ran into an interesting and unexpected behaviour relating to Timestamps. 

<!-- more -->

In Java the standard way to control object equality is to implement an equals() method (and the associated hashcode() method). Generally, you would normally expect .equals() to be Symmetric (as defined by the [Java documentation for Object](http://docs.oracle.com/javase/6/docs/api/java/lang/Object.html#equals\(java.lang.Object\)). However, when using a java.util.Timestamp (which is a specialisation of java.util.Date), you can create an interesting scenario where equality is not symetric. This is best illustrated by the following test:

```java
@Test
public void testTimestampEquality() {
    final Date date = new Date();
    final long epochTime = date.getTime();
    final Timestamp timestamp = new Timestamp(epochTime);
    Assert.assertTrue(date.equals(timestamp));
    // Normally, you'd expect date.equals(timestamp) to imply that 
    // timestamp.equals(date). However, this is not the case:
    Assert.assertFalse(timestamp.equals(date));
}
```

This is a [documented 'feature'](http://docs.oracle.com/javase/6/docs/api/java/sql/Timestamp.html#equals\(java.lang.Object\)) and is there for backward compatibility reasons. However, if you don't know you have an instance of java.util.Timestamp then this can be an interesting gotcha. I encountered this issue on a project where some client-side code used java.util.Date, but a deserialisation library had automagically created an instance of a Timestamp.