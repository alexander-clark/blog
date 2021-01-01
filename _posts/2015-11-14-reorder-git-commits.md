---
layout: post
title: Reorder Git Commits Through Rebasing
published: true
author: alexander
comments: false
date: 2015-11-14 07:11:06
tags:
    - git
    - rebase
categories:
    - uncategorized
permalink: /reorder-git-commits
---
The other day I was working on a pull request when I discovered I had forgotten something trivially simple that should have gone in the commit before the one I just made. If I hadn&#8217;t have made that other commit, I could just --amend it. I could <kbd>git reset HEAD^</kbd>, amend my commit, then restage and redo my second commit, but that&#8217;s a lot of steps and could get ugly in a hurry if the target commit was more than one commit back.

What would be perfect would be just making an extra &#8216;oops&#8217; commit and squashing it later. But squash and fixup meld into the previous commit. If only there were a way to change the order of commits so I could squash as desired.

As it turns out, there is!

## Reorder Git Commits with an Interactive Rebase

[Reordering git commits][1] is about as intuitive as it gets. Simply:

  1. start an interactive rebase &#8211; in my case with git rebase --interactive HEAD~3
  2. change the order of the commits in your editor
  3. save and quit.

You can even squash or fixup at the same time.

It&#8217;s even right there in the comments, though a bit opaquely: &#8220;These lines can be re-ordered; they are executed from top to bottom.&#8221;

&#8216;Executed&#8217; refers to the commands in your rebase (e.g. pick). So the reason this works is the pick command actually adds the commit. Order matters. I guess I always thought pick meant &#8220;yes keep this one&#8221; rather than &#8220;add this one to the end of the queue.&#8221;

<figure>
  <a href="/assets/reorder-git-commits-before.png">
    <img src="/assets/reorder-git-commits-before.png">
  </a>
  <figcaption>Before</figcaption>
</figure>

<figure>
  <a href="/assets/reorder-git-commits-after.png">
    <img src="/assets/reorder-git-commits-after.png">
  </a>
  <figcaption>Reorder git commits and fixup</figcaption>
</figure>

As always, the standard history-changing caveat applies. If you&#8217;ve already pushed changes to a shared branch, you&#8217;re going to have to live with your oops commit. If you&#8217;re working on a private topic branch or you haven&#8217;t pushed yet, rebase away ;)

 [1]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Reordering-Commits
