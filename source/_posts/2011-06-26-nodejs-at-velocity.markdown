---
layout: post
title: "NodeJS at Velocity"
date: 2011-06-26 21:17
comments: true
categories:
- velocityconf
- node.js
---

NodeJS seemed to be a passing theme in a number of the talks at the conference,
including:

* "Performance Enhancing Programming with NodeJS" by Tim Caswell
* Bryan Cantrill's "Instrumenting the real-time web: Node.js, DTrace and the Robinson Projection"
* "Cast - The Open Deployment Platform" by Paul Querna
* "Real-Time, Real-Fast" by Dylan Schiemann

I attended Tim Caswell's and Bryan Cantrill's sessions, both of which were great
introductions to Node.js hipe

Tim Caswell's (creationix on twitter/github) workshop was a great demo of
Node's scalability. He spun up a webserver on a private wlan and got the whole
room to start pinging various demo apps.

A few things discussed:

Running JavaScript on client and server can avoid the issue of doubling your
effort... You don't cross the boundary all the time if pieces of modular
functionality can be run either client side or server side.
There's an opportunity to run more in the server during the period that the
Browser's busy downloading content or engaged in pre-render tasks. Once the
app is fully up, you can then progressively do more in the client by shipping
the same scripts over the wire.
Non blocking IO. Using buffers and memcpy to avoid string builder overhead
Some discussion on tooling: node-inspector, DTrace, node -debug, and v8
graphical debug all sound interesting.

Bryan Cantrill's talk was, in a single word, energetic. Unsurprisingly we saw a
lot of DTrace and we heard about a Node.js app he'd build to monitor net traffic
during a hackday. I think that Cantrill's talk can be summarised as Node being
the union of the popularity of JavaScript, non-blocking IO, and the Unix APIs
(or, as Cantrill puts it "the APIs God intended").

In summary, there seems to be a lot of hype around Node. And it seems that
it's the mix of JavaScript, Closures, and Non-blocking IO that are capturing
the mood. Also because JavaScript never had any IO in the core language, you
don't have to drop the bits of the language that don't fit the Node.js model
(e.g. if you're using twisted you're only wanting to use a subset of python).
I'll definitely be playing with it to understand more.

Of course, the backbone of Node is the V8 JavaScript engine, and the theme of
JavaScript tuning was definitely strong at Velocity. Another interesting keynote
was Douglas Crockford on JavaScript. He was talking about the JavaScript
benchmarks and how a lack of good benchmarks had resulted in the wrong kind of
optimisations. Recently Crockford created a benchmark by running jslint over
jslint. He argued that this provided a much better benchmark as it touched more
widely-used patterns (such as closures and prototypal inheritance) than the more
common "how fast does this loop execute" tests often found in benchmarks. He
also shared some interesting stats on performance gains that the V8 team had
been able to make using his benchmark.