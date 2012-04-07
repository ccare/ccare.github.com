---
layout: post
title: "Stub Http Server"
date: 2012-03-07 20:25
comments: true
categories:
- Testing
- HTTP
- Java
- Jersey
- Jetty
- Acceptance Tests
---

At [Talis](http://talisplatform.com) we've been building our RDF platform
following a distributed SOA
architecture. Most of our integrations are through RESTful HTTP APIs, and in our
Java projects we tend to use Apache HttpClient to consume them.

As a result, when writing tests I find myself having to frequently stub-out a
remote service. Sometimes we do the usual thing and create mocks. This is fairly
easy, but does involve quite a lot of plumbing - you need a mock HttpClient, then
a mock GetRequest, then a mock entity, and so on... it can certainly create
quite a lot of boilerplate.

I think this sort of test works much better with a _real_ HttpClient, making
real HTTP calls. We've been using this pattern quite a lot, and found that the
JettyRule class from the junit-rules project is a great way of embedding an
HTTP server into a unit test. Choosing a random free port for this server helps
make the test more robust and independent.

However, I found myself writing more and more 'mock-like' assertion code into my
handlers. Implementing the happy path was easy, but if I, for example, wanted to
test fail-and-retry logic, I needed to make a number of requests fail, then one
succeed, and assert no other spurious requests were made. Traditionally this is
the perfect use-case for a mock - but I wanted to make these kind of assertions,
and, at the same time, exercise a real HttpClient.

To help, I wrote [stub-http](http://github.com/ccare/stub-http), a really simple stub HTTP Server that is
embeddable in a unit test and supports the recording and playback of a sequence
of Requests and Responses. I based the code on the interface of EasyMock, and
like EasyMock, the stub can be run in either 'strict', 'regular', or 'nice'
mode. When running in strict mode, unexpected requests result in an assertion
failure, whereas in nice mode, they return an HTTP 404 code. Switching between
'regular' and 'strict' reflects whether ordering of requests matters.

Here's an example test:

```java
public class MyTest {

    @Rule
    public StubHttp stubHttp = new StubHttp();
    
    @Test
    public void clientMakesHEADRequestFollowedByGET() {
        File testZipFileLocation = new File(...);
        stubHttp.expect("HEAD", "/foo.zip").andReturn(200);
        stubHttp.expect("GET", "/foo.zip").andReturn(200);
        stubHttp.replay()
        
        // ...test client call here...      
    }
    
    @Test
    public void clientDoesntDoGETIfHEADRequestIs404() {
        File testZipFileLocation = new File(...);
    
        // Record a single HEAD (returning followed by 404),
        //  followed by nothing else
        stubHttp.expect("HEAD", "/foo.zip").andReturn(404);
        stubHttp.replay()
        
        // ...test client call here...        
    }
    
    
    @Test
    public void clientRetriesOn503() {
        File testZipFileLocation = new File(...);
        // The first couple of HEAD requests result in a 503 "Temporarily unavailable"
        //   response, this is then followed by a 200 "OK" response, and a subsequent GET
        stubHttp.expect("HEAD", "/foo.zip").andReturn(503);
        stubHttp.expect("HEAD", "/foo.zip").andReturn(503);
        stubHttp.expect("HEAD", "/foo.zip").andReturn(503);
        stubHttp.expect("HEAD", "/foo.zip").andReturn(200);
        stubHttp.expect("GET", "/foo.zip").andReturn(200);
        stubHttp.replay()
        
        // ...test client call here...    
    }

}
```

