---
layout: post
title: "Creating a simple AMF client for testing"
date: 2013-05-09 12:31
comments: true
categories: 
- AMF
- Debugging
- Flash
- Flex
- Java
- Wire formats
---

<p class="headline-box">This post was originally written for the <a href="http://www.blackpepper.co.uk/blog/">Black Pepper Software</a>
blog. The <a href="http://www.blackpepper.co.uk/posts/creating-a-simple-amf-client-for-testing/">original</a>
was posted on 30/04/2013.</p>

A few months back I had to debug an Adobe Flex application that used Adobe Message Format (AMF) for client-server communication. For readers unfamiliar with Adobe's Flex/Flash stack, AMF is a binary RPC communication protocol, usually powered by Adobe's BlazeDS technology. The application I was working on used BlazeDS to expose a number of AMF web services, backed by a fairly standard Java/Spring/Hibernate stack.

As I started investigating the bug, I quickly began to suspect that the root cause was data related. Since many of the application screens were data-driven, I conjectured that there was inconsistent or incomplete data being returned from the web services. At the time, I didn't have access to a stable version of Adobe FlexBuilder for Linux, so I wasn't able to easily jump in and attach a debugger to the Flex application. However, I was able to read the code and understand a bit about the RPC service calls it was making. The server-side access logs also provided a good idea of the client-server interactions that were happening. In order to gain further insight into the scenario under test, I found myself wanting to 'see' the data that was flowing between the two applications.
<!-- more -->

Even the simplest client-server app is a form of a distributed system, and when debugging a distributed system, I'm a big advocate of understanding the actual data flowing over the wire. I find it helps build up a picture of what an application is actually doing. Most of the systems I work with use human-readable formats in their web-services and, as a result, I always keep tools like tcpdump and wireshark to hand. However, with a binary format such as AMF, it's really difficult to 'see' what data is being transferred.

Building a client
-----------------
Thankfully, there are client-side Java bindings for BlazeDS which can interact with an RPC endpoint and handle the marshalling/unmarshalling of data into Java POJOs. This allowed me to bypass flex altogether and hack together a simple Java client to interact with my webservices. In order to get started, I started a new Maven project, and added the following dependencies:

```xml
<dependency>
    <groupId>com.adobe.blazeds</groupId>
    <artifactId>blazeds-core</artifactId>
    <version>3.2.0.3978</version>
</dependency>

<dependency>
    <groupId>com.adobe.blazeds</groupId>
    <artifactId>blazeds-common</artifactId>
    <version>3.2.0.3978</version>
</dependency>

<dependency>
    <groupId>com.adobe.blazeds</groupId>
    <artifactId>blazeds-remoting</artifactId>
    <version>3.2.0.3978</version>
</dependency>
```

Connecting to the AMF endpoint then just required a few lines of Java code:

```java
AMFConnection amfConnection = new AMFConnection();
try {
    String url = "http://myapp.example.com/myservice/amf";
    amfConnection.connect(url);
} finally {
    amfConnection.close();
}
```
Once I had a connection, it was simple to call one of the web services. For instance, in the application I was interested in, there was a UserService with an remote method named getUserByEmailAddress. The method took a single parameter: the user's email address. To invoke this service, I did the following:


```java
Object result = amfConnection.call("UserService.getUserByEmailAddress", 
										"charles@example.com");
```

This all looked good. However, I wasn't quite finished. In this particular example, the service returned a specific type of object that needed to be unmarshalled. Let's say that the return type of getUserByEmailAddress() was a class with a qualified name of **com.example.model.User**. In order for the unmarshalling code to work, the AMF bindings required me to create a client-side Java class for this type. But this was not too difficult, I just needed to provide the relevant fields, setters, and getters. For instance, something like this:

```java
package com.example.model;

public class User {

    private final String name;
    private final String emailAddress;

    ...
}
```

Of course, what we have here is a classic Data Transfer Object (or DTO), and I soon found that I'd need to re-create a large number of DTO classes, far more boiler-plate code than I wanted to write. In some projects it might be possible to share some class definitions with the server project. However, in my particular example, the server-side classes were entities containing Hibernate/JPA annotations and were not suited to be used in a client-side environment.

Using HashMap-backed Data Transfer Objects
------------------------------------------

In fact, there is a far easier approach. Although the client requires local DTO classes, they can all be backed by a HashMap, drastically reducing the boiler-plate required. For instance, in the case of the User class, all I needed to do was to extend **java.util.HashMap**. I was able to drop all of the declarations and just have a empty class:

```java
package com.example.model;

public class User extends HashMap {
    // No further declarations required
}
```

Then I was able to use simple Map getters to access the data and have a look at what was going on.

```java
User result = (User) amfConnection.call("UserService.getUserByEmailAddress",
											"charles@example.com");
System.out.println(result.get("name"));
System.out.println(result.get("email"));
```

Summary
-------

Building a lightweight Java client provides an alternative way of accessing a BlazeDS endpoint, which I found very useful when debugging and comprehend AMF services. Having this a client also enabled me to write a programmatic API test that didn't rely on running a Flash Player. Finally, using DTOs that are specialisations of HashMap reduces boilerplate and allows you to make faster progress.

And the bug? Well, it was data related, and it did get fixed.