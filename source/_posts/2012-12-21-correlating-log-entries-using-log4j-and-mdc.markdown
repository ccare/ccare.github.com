---
layout: post
title: "Correlating log entries using Log4J and the Mapped Diagnostic Context"
date: 2012-12-21 12:31
comments: true
categories: 
-  java GWT log4j logging
---

<p class="headline-box">This post was originally written for the <a href="http://www.blackpepper.co.uk/blog/">Black Pepper Software</a>
blog. The <a href="http://www.blackpepper.co.uk/posts/correlating-log-entries-using-log4j-and-the-mapped-diagnostic-context/">original</a>
was posted on 20/12/2012.</p>

Over the last couple of years, it has become increasingly common for web applications to be built using client-side frameworks that interact with a remote server via asynchronous requests. Unlike a traditional app, that makes a single request per page, a modern web application will probably make many asynchronous HTTP requests. This poses a challenge when analysing server logs, since it can be difficult to identify which requests belong to the same client.

In this blog post I'll describe a simple technique we've applied to retro-fit a client-aware correlation token to the server logs of an existing web application.
<!-- more -->

The problem
-----------

Our web application communicates with a Java application server over HTTP. The majority of client-side code is written in Java, using Google Web Toolkit (GWT), which is then compiled into JavaScript. The application server exposes both RESTful and GWT-RPC-based web services.

For traditional webpages that follow a simple request-response pattern, log analysis is very straightforward. In a standard J2EE implementation, each request is served by a separate web server thread allocated from a pool. Once a request has been fully completed, that thread is returned to the pool so that it can process future requests. A convenient result of this 'one-thread-per-request' model is that log lines can be correlated using the thread's id. So a single request, handled by a thread named "http-8080-3", might produce the following lines of output (I've shortened all timestamps in the log-line snippets):

```
17:25 [http-8080-3] INFO Service1.execute(77) | Handling request for service
17:25 [http-8080-3] INFO Service1.execute(77) | Requesting data from database
17:25 [http-8080-3] INFO Service1.execute(77) | Completed request
```

When the server is busy, these log lines will inevitably become interleaved, but they're easy to unravel by looking at the thread id. Here we can see that there are two threads (http-8080-3 and http-8080-6) processing concurrent requests to the same service.

```
17:25 [http-8080-3] INFO Service1.execute(77) | Handling request for service
17:25 [http-8080-6] INFO Service1.execute(77) | Handling request for service
17:25 [http-8080-3] INFO Service1.execute(77) | Requesting data from database
17:25 [http-8080-3] INFO Service1.execute(77) | Completed request
17:25 [http-8080-6] INFO Service1.execute(77) | Requesting data from database
17:25 [http-8080-6] INFO Service1.execute(77) | Completed request
```

Thread-based correlation is a useful technique for identifying all log entries for a **single** request. However, in this example we don't know whether these requests came from the same web client or from different clients. In order to get more value from our application logs, we really need to correlate all requests from a single client.

Consider the following scenario: we have two customers connecting to our site, and when they load the home page, each of their browsers will make two web service requests, one to **Service1** and another to **Service2**. If both clients load at the same time, we may get an interleaving like this.

```
17:25 [http-8080-3] INFO Service1.execute(77) | Handling request for service 1
17:25 [http-8080-6] INFO Service1.execute(77) | Handling request for service 1
17:25 [http-8080-3] INFO Service1.execute(77) | Requesting data from database
17:25 [http-8080-2] INFO Service2.execute(77) | Handling request for service 2
17:25 [http-8080-3] INFO Service1.execute(77) | Completed request
17:25 [http-8080-6] INFO Service1.execute(77) | Requesting data from database
17:25 [http-8080-1] INFO Service2.execute(77) | Handling request for service 2
17:25 [http-8080-2] INFO Service2.execute(77) | Requesting data from database
17:25 [http-8080-2] INFO Service2.execute(77) | Completed request
17:25 [http-8080-1] INFO Service2.execute(77) | Requesting data from database
17:25 [http-8080-6] INFO Service1.execute(77) | Completed request
17:25 [http-8080-1] INFO Service2.execute(77) | Completed request
```

It's clear from these logs that there were two calls to each of our services, but it's not easy to identify which ones belong to the same customer. Was our first customer using threads http-8080-3 and http-8080-2, or do the requests relating to threads http-8080-3 and http-8080-1 go together?

One way to distinguish between them might be to print the client IP address, although that would rely on our load balancer passing through the source IP (and wouldn't provide enough granularity if multiple sessions were coming from the same source). In any case, if both clients were behind a firewall we might see them as originating from the same address. Another approach might be to make use of server-side sessions to correlate requests. However, this wouldn't distinguish between separate page loads, perhaps initiated in multiple browser tabs. We would also prefer to avoid maintaining server-side state.

We finally settled on a solution based on generating a (reasonably) unique identifier for each application session client-side. This identifier can be passed as an HTTP header with each request and ultimately appended to each log line. In order to add the identifier to each log-line we used Log4J's Mapped Diagnostic Context (MDC), which provides a simple key/value mechanism to capture small amounts of custom diagnostic data. Since it's built into the logging framework, it's really easy to add values from the MDC to each log line. For instance, here we have the same log fragment, but with an additional identifier field (here: 'ae0021f' and 'f3wev1x') to distinguish between the different clients:

```
17:25 [http-8080-3] INFO Service1.execute(77) [ae0021f] | Handling request for service 1
17:25 [http-8080-6] INFO Service1.execute(77) [f3wev1x] | Handling request for service 1
17:25 [http-8080-3] INFO Service1.execute(77) [ae0021f] | Requesting data from database
17:25 [http-8080-2] INFO Service2.execute(77) [ae0021f] | Handling request for service 2
17:25 [http-8080-3] INFO Service1.execute(77) [ae0021f] | Completed request
17:25 [http-8080-6] INFO Service1.execute(77) [f3wev1x] | Requesting data from database
17:25 [http-8080-1] INFO Service2.execute(77) [f3wev1x] | Handling request for service 2
17:25 [http-8080-2] INFO Service2.execute(77) [ae0021f] | Requesting data from database
17:25 [http-8080-2] INFO Service2.execute(77) [ae0021f] | Completed request
17:25 [http-8080-1] INFO Service2.execute(77) [f3wev1x] | Requesting data from database
17:25 [http-8080-6] INFO Service1.execute(77) [f3wev1x] | Completed request
17:25 [http-8080-1] INFO Service2.execute(77) [f3wev1x] | Completed request
```

The implementation
------------------

The implementation doesn't intrude into the business logic of our application and involves 3 simple steps:

Firstly, when the application starts, we generate an identifier client-side, and add it to all subsequent HTTP requests. As I mentioned earlier, our application is built using GWT and interacts with the server using both GWT-RPC and ordinary AJAX calls to JSON APIs. Adding the HTTP header to our GWT RPC calls is a simple one-liner if you're using com.google.gwt.http.client.RequestBuilder:

```java
requestBuilder.setHeader("X-BLACKPEPPER-SESSION-ID", globalClientId);
```

And for our JSON API calls, we just add the same header to our JQuery ajax calls:

```javascript
	$.ajax({
    ...
    headers: {
        "X-BLACKPEPPER-SESSION-ID": globalClientId,
        ...
    }
    ...
}
```

Secondly, the next step is to capture the identifier from each request. In order to not interfere with our existing business logic, we implemented a ServletFilter that captures the custom header and stores it in the Log4J MDC with the key 'CLIENT_ID'.

```java
import org.apache.log4j.MDC;
...
public class ClientAwareLoggingFilter implements Filter {
    ...
    @Override
    public void doFilter(final ServletRequest req,
                   final ServletResponse res,
                   final FilterChain chain)
                throws IOException, ServletException {
       if (req instanceof HttpServletRequest) {
           final HttpServletRequest httpRequest = (HttpServletRequest)req;
           final String clientSessionId =
                        httpRequest.getHeader("X-BLACKPEPPER-SESSION-ID");
           if (clientSessionId != null) {
               MDC.put("CLIENT_ID", clientSessionId);
           }
       } else {
           MDC.put("CLIENT_ID", null);
       }
       chain.doFilter(req, res);
   }
   ...
}
```

Finally, we edit our Log4J configuration to print out the contents of our MDC field. This is accomplished using the %X pattern and referencing the CLIENT_ID key:

```
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
  <appender name="FILE">
    ...
    <param name="Append" value="true" />
    <layout>
        <param name="ConversionPattern" 
           value="%d [%t] %p %c.%M(%L) [%X{CLIENT_ID}] | %m%n"/>
    </layout>
  </appender>
</log4j:configuration>
```

Correlating acceptance tests
----------------------------

Although we originally added the correlation to help with the diagnosis of live logs, we've also found it really useful when investigating acceptance test failures. At Black Pepper our continuous integration server runs large numbers of automated UI tests. For this project we use Selenium's WebDriver, and follow the fairly standard pattern of launching our application once for the entire test run. As a result, we have a single application log for the entire acceptance suite, so if a single test fails it can be difficult to find the log entries relating to the failure. Since we launch a new client for each test, the correlation token helps, as long as we can identify which token belonged to each test. We can make a further improvement by using the actual test name as the correlation token. We modified our application so that the client identifier could be overridden with a browser cookie, and added a JUnit @Rule to store the name of the current test in the cookie:

```java
@Rule
public TestName name = new TestName() {
   @Override
   protected void starting(final Description description) {
       super.starting(description);
       final String methodName = description.getMethodName();
       ...
       if (webDriver.manage().getCookieNamed("CLIENT_ID_SLUG") == null) {
           webDriver.manage().addCookie(
                              new Cookie("CLIENT_ID_SLUG", methodName));
       }
   }
}
```

Now, when we run the tests it is clear which log lines relate to which test:

```
 17:25 [http-8080-3] ... [asACustomerICanOpenHomePage] | Handling request for service 1
 17:25 [http-8080-6] ... [asACustomerICanBuyMyProduct] | Handling request for service 1
 17:25 [http-8080-3] ... [asACustomerICanOpenHomePage] | Requesting data from database
 17:25 [http-8080-2] ... [asACustomerICanOpenHomePage] | Handling request for service 2
 17:25 [http-8080-3] ... [asACustomerICanOpenHomePage] | Completed request
 17:25 [http-8080-6] ... [asACustomerICanBuyMyProduct] | Requesting data from database
 17:25 [http-8080-1] ... [asACustomerICanBuyMyProduct] | Handling request for service 2
 17:25 [http-8080-2] ... [asACustomerICanOpenHomePage] | Requesting data from database
 17:25 [http-8080-2] ... [asACustomerICanOpenHomePage] | Completed request
 17:25 [http-8080-1] ... [asACustomerICanBuyMyProduct] | Requesting data from database
 17:25 [http-8080-6] ... [asACustomerICanBuyMyProduct] | Completed request
 17:25 [http-8080-1] ... [asACustomerICanBuyMyProduct] | Completed request
 ...
```

Conclusion
----------

Making a simple change to our application to submit a client identifier in an HTTP header has helped us correlate our server-side logs. The facilities in the Log4J Mapped Diagnostic Context made this really easy to implement. Furthermore, using a cookie-based override allowed us to make our acceptance test logs more readable.

MDC is a really handy feature of Log4J, and there are alternatives in other logging frameworks. The SLF4J facade library, for instance, provides an interface to the MDC when a suitably featured logging library is used (at the time of writing, these were just LogBack and Log4J). For libraries that don't support this feature SLF4J provides a BasicMDCAdapter class which provides a basic implementation using ThreadLocal storage. Developers using .Net will find an MDC implementation in the NLog and log4net libraries.

Links
-----

MDC documentation: <a href="http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/MDC.html">http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/MDC.html</a>

SLF4J's implementation: <a href="http://www.slf4j.org/manual.html#mdc">http://www.slf4j.org/manual.html#mdc</a>