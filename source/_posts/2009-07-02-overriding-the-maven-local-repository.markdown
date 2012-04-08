---
layout: post
title: "Overriding the Maven local repository"
date: 2009-07-02 14:17
comments: true
categories:
- Maven2
- Java
---

Playing with Maven today - have been trying to establish a build from a clean repository (not .m2). Guessed that you could set the local repository, but took ages to find the prop...

    mvn -o install -Dsettings.localRepository=/var/tmp/m2

does the trick