---
layout: post
title: "JPA and Kasabi"
date: 2012-03-20 18:04
comments: true
categories:
- Java
- Kasabi
- JPA
---

Over the last couple of days I've been trying to pull together a demo of
integrating an existing JPA application with Kasabi. It's been a fun journey
learning (and relearning) the JPA spec, and also working with some of the
internals of the Hibernate framework. It's interesting that when a framework
becomes really ubiquitous - like Hibernate, in this case - it's really easy to
find online docs and howtos about how to use it, but it's far harder to sift
through all that noise to find docs that help you extend it.

In the end, I did get it working: I re-used the annotation scanning plumbing
that Hibernate provides to get access to the JPA configuration, then I walked
the object graph marshalling entities as RDF (I used NTriples) before submitting
them to a Kasabi dataset.

Of course, to complete the demo, I needed an actual JPA application, so I've had
some fun re-learning Spring Roo and Grails in order to build a quick
database-backed webapp. Normally I dislike scaffolding-based frameworks, but
they're certainly useful for creating a quick functional app.

I'll submit a link to the code at some point soon.