---
layout: post
title: "Avoiding 'peer not authenticated' error in Apache Httpclient"
date: 2011-05-11 09:18
comments: true
categories: 
---

Ran into some issues with httpclient not accepting SSL certs for an Amazon S3 url. Wanted to temporarily turn off httpclient's certificate verification. This how-to was really helpful. 

http://javaskeleton.blogspot.com/2010/07/avoiding-peer-not-authenticated-with.html 