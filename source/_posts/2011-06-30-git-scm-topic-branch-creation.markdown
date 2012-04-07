---
layout: post
title: "git-scm topic branch creation"
date: 2011-06-30 08:42
comments: true
categories: 
---

Way I tend to create a topic branch in git-scm for a specific issue:

    git checkout -b  # create a local branch
    # Do some stuff. We decide that its worth pursuing, so lets add the branch to the remote repo
    git push origin [branch-name] # push the local branch to the remote repo
    git branch --set-upstream [branch-name] origin/[branch-name] # make branch track remote
    # Carry on working in the branch