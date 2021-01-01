---
layout: post
title: Easier Bisecting with Tests
published: true
author: alexander
comments: true
date: 2015-09-01 06:09:28
tags:
    - git
    - ruby
    - tdd
    - testing
categories:
    - uncategorized
permalink: /easier-bisecting-with-tests
---
You&#8217;ve probably used &#8211; or at least watched someone use &#8211; [git bisect][1] before. It&#8217;s a great way to track down which commit introduced a bug. It usually looks something like this:

<kbd>git bisect start</kbd>

<kbd>git bisect good [ref before bug existed]</kbd>

<kbd>git bisect bad [ref after bug was introduced...often HEAD]</kbd>

[Check if bug is there&#8230;yup!]

<kbd>git bisect bad</kbd>

[Check for the bug&#8230;no.]

<kbd>git bisect good</kbd>

[Check for bug&#8230;no.]

<kbd>git bisect good</kbd>

[Bug? Yes.]

<kbd>git bisect bad</kbd>

…

[SHA where bug was introduced is isolated. Yay!]

<kbd>git bisect reset</kbd>

As you might know, there&#8217;s a way to have a script automatically do all the tedious good/bad checking for you: git bisect run. What you might not know is how easy it is to use.

When I go to fix a bug, usually the first thing I do is write a failing test that should pass when the bug is fixed. This forces me to think about the problem in very concrete terms, and gives me an indicator to know when I&#8217;ve fixed it. TDD.

As it turns out, this test is all you need to use git bisect run!

Normally, the test would probably go in an existing file in the test/spec directory. To start with, though, let&#8217;s put it in its own file. Because git bisect works by checking out lots of commits, a new file is your best bet for avoiding potential merge conflicts.

_C&#8217;est tout._ That&#8217;s it. We&#8217;re ready to use git bisect run.

<kbd>git bisect start</kbd>

<kbd>git bisect good [ref before bug existed]</kbd>

<kbd>git bisect bad [ref after bug was introduced]</kbd>

<kbd>git bisect run [command to run your test]</kbd>

[Lots of automated checking and test output]

[SHA where bug was introduced is isolated. Yay!]

<kbd>git bisect reset</kbd>

Now that you&#8217;ve found the offending ref and run git bisect reset it&#8217;s safe to move your test to the appropriate file.

Quick addendum:

Why would you care when the bug was introduced? Why not just fix it and move on?

  * In many cases, you really don&#8217;t
  * If your project actively maintains multiple versions, knowing where the bug was introduced will tell you which are affected without having to test every last one
  * If you&#8217;re having a hard time figuring out what&#8217;s causing the bug, sometimes looking at the commit that introduced it can narrow it down
  * If you want to do some kind of post-mortem to figure out who wrote the bug, why a test wasn&#8217;t written, etc

 [1]: http://git-scm.com/docs/git-bisect
