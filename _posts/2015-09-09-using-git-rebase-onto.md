---
layout: post
title: Using git rebase onto
published: true
author: alexander
comments: true
date: 2015-09-09 01:09:35
tags:
    - git
    - rebase
categories:
    - uncategorized
permalink: /using-git-rebase-onto
---
Have you ever branched off of master, done some work, committed, pushed, and opened a pull request only to find out your code belongs in some other long-running branch?

What do you do? Delete your branch and rewrite your code? Stash it and apply onto a new branch? Cherry pick your commits to a new branch?

How about git rebase onto?

I don&#8217;t know what your git workflow looks like, but just to start with an easy example, let&#8217;s say your server runs the production branch and normal development happens against master. You&#8217;ve opened a PR against master, but now you&#8217;ve been asked to make it a hotfix into production. There are several commits in master (and thus, on your branch) that are not ready for prime time. How do you fix it in one command?

<kbd>git rebase --onto production master</kbd>

The syntax is just git rebase &#8211;onto \[target-parent\] \[current-parent\]. target-parent and current-parent can be any git ref.

This, of course, results in a non-fast-forward update, so in order to push your changes, you&#8217;ll need to do one of two things:

  1. If your organization doesn&#8217;t frown on it, use the force! <kbd>git push --force origin yourbranchname</kbd>
  2. If you or your organization are uncomfortable with force pushing, you can rename the branch before pushing. <kbd>git branch -m yournewbranchname</kbd>

A couple last words. Never, ever force push a shared branch. It will cause problems for someone else and everyone will hate you. Because git rebase onto requires force pushing, don&#8217;t do that on a shared branch either. But really, why would you ever do either one in the first place!? You shouldn&#8217;t be committing directly to a shared branch anyway. If you write all of your code in personal topic branches that get merged into the shared branches, you can use cool things like git push force and git rebase onto. This is why we _can_ have nice things.

Also, git rebase can be somewhat dangerous if you don&#8217;t know what you&#8217;re doing. More to your sanity than your code, but still. Make sure you know [what to do][1] if you fuck things up.

 [1]: http://stackoverflow.com/a/135614/2812480
