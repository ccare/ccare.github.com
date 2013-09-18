---
layout: post
title: "Custom reverse proxying with Node.js"
date: 2013-09-18 13:36
comments: true
categories: Node.js
---
On a number of occassions over the last year or so, I've found myself writing
wanting to create a reverse proxy to front a number of HTTP services. For me,
the usual requirement is that I want to access these services via AJAX requests,
and using a reverse proxy allows me to not need to worry about cross-site scripting
(XSS) or configuring cross-origin resource sharing (CORS). Node.js is a fantastic
tool for doing this. Particularly if you lean on the well-regarded
node-http-proxy module.

If all you need is static proxying/routing, then node-http-proxy project
supports a variety of
[https://github.com/nodejitsu/node-http-proxy](configuration options).
However, I usually find myself turning to this library when I want to
do custom routing. So in this post, I focus on configuring the proxy with
custom code.

In this blog post, I'll share some code snippets to create a simple reverse proxy
in Node.js and then move onto a slightly more complicated example with some routing
and express middleware. Finally, I'll show how to base the routing on an asynchronous
call to another service (e.g. a service-discovery service for looking up endpoints ).
<!-- more -->

Simplest proxy
--------------

The simplest proxy you can write is one that routes all requests to the same
back-end service. There might be a number of reasons to do this, the last time I
wrote something like this was to add a couple of HTTP headers to each outbound
request in order to authenticate against an API.

```javascript
var httpProxy = require('http-proxy');

var proxy = httpProxy.createServer(function (req, res, proxy) {
	if (match) {
	req.headers["X-CUSTOM-API-KEY"] = 'my-api-key';
    proxy.proxyRequest(req, res, {
	    host: 'backend.example.com',
        port: 9000
    });      
})

proxy.listen(8000);
```

Simple Routing
--------------

Perhaps your front-end app needs to route different paths to different backends,
that's easy too. E.g. perhaps paths beginning /api/ need to route to api.example.com

```javascript
var httpProxy = require('http-proxy');

var proxy = httpProxy.createServer(function (req, res, proxy) {
	var backendHost;
	if (/^\/api/.exec(req.url)) {
		backendHost = 'api.example.com';
	} else {
		backendHost = 'backend.example.com';		
	}
	req.headers["X-CUSTOM-API-KEY"] = 'my-api-key';
    proxy.proxyRequest(req, res, {
	    host: backendHost,
        port: 9000
    });
})

proxy.listen(8000);
```

Routing based on an async call
------------------------------

Routing can be based on code with arbitrary complexity. For instance, it may be
necessary to call another service to lookup the host to route to. In Node.js, this
lookup will likely be an async function, so the actual routing will happen in
a callback. In order for the callback to have access to the correct request and
response streams, the http-proxy library provides a buffer() method.
The buffer() method returns a buffer reference that is then passed into
the call to proxyRequest();

```javascript
var httpProxy = require('http-proxy');

var proxy = httpProxy.createServer(function (req, res, proxy) {
	var requestBuffer = proxy.buffer();
	discoverHostFor(req.url,
		function(err, result) {
			// NB: should implement some error handling in case of
			// err being non-null.
			proxy.proxyRequest(req, res, {
				host: result.host,
				port: 9000,
				buffer: requestBuffer
			});
		})
	}
})

proxy.listen(8000);
```

References
----------

* The node-http-proxy library [https://github.com/nodejitsu/node-http-proxy]

