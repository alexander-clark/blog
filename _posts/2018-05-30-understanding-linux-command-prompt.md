---
layout: post
title: Understanding the Linux Command Prompt
published: true
author: alexander
comments: true
date: 2018-05-30 01:05:44
tags:
    - command line
    - linux
categories:
    - linux-command-line
permalink: /understanding-linux-command-prompt
---
First, the preliminaries. In order to understand the Linux command prompt, it&#8217;s necessary to recognize the significance of the pieces it&#8217;s comprised of. Certain things are necessary in order to have a terminal session at all.

## Prerequisites

It&#8217;s not possible to use the command line without logging in first. In order to log in, one needs a user account.

It&#8217;s also necessary to have something to log in to. Every terminal session is predicated on being logged in to a particular computer.

So far, this sounds similar to using a modern operating system like Windows or OSX. But here&#8217;s where that metaphor breaks down. The command line isn&#8217;t like Windows or OSX (Linux is the operating system). It&#8217;s closer to Windows Explorer or Finder on OSX. It&#8217;s always tied to a folder or directory the user is working in. If there&#8217;s no working directory, there&#8217;s no terminal session.

Every user has a default working directory, called their &#8216;home directory.&#8217; On most Linux-like operating systems, this directory is /home/username, but on OSX[^1], it&#8217;s /Users/username.

Finally, there&#8217;s more than piece of software that works as a command prompt. These are called shells. Most of the time, the Bourne Again shell, colloquially referred to as &#8220;bash&#8221; is the default.

## The Default Linux Command Prompt

<pre><samp>user@host:~$</samp></pre>

<pre><samp>root@host:/#</samp></pre>

<pre><samp>joe@coolbox:blah%</samp></pre>

## Decoding the Prompt

[![](assets/ec2-prompt-300x142.png)](assets/ec2-prompt.png)

Usually, it indicates who&#8217;s logged in, and the name of the machine they&#8217;re logged into. A common default name for a computer is localhost.localdomain.

Next, it tells you what directory you&#8217;re currently in. Sometimes it&#8217;s spelled out completely, such as /usr/local/bin/, or sometimes it&#8217;s shortened to just the name of the folder, e.g. bin. The tilde &#8211; &#8216;~&#8217; &#8211; character is shorthand for the logged-in user&#8217;s home directory.

One user, called the superuser, has special privileges on a system. The username for the superuser is usually root. It&#8217;s generally not a good idea to log in as the superuser unless you&#8217;re doing work that absolutely requires those privileges, such as installing software system-wide.

A hash &#8211; &#8216;#&#8217; &#8211; character at the end of a prompt is used to remind you that you&#8217;re logged in as root and to be careful. The $ at the end of a prompt means you&#8217;re using bash or similar. A % means you&#8217;re using csh, tcsh, zsh, or similar.

## Continuation Prompt

When typing a long command, it&#8217;s possible to split the command onto multiple lines. Pressing enter will attempt to run the command unless the last character on the line is a backslash &#8211; &#8216;\&#8217; &#8211; which tells the prompt you want to type a new line on the screen instead. Not unlike &#8220;Shift+Enter&#8221; on many messaging apps.

<pre><samp>user@host:~$ echo \
&gt;</samp> <kbd>"hello"</kbd>
<samp>hello
user@host:~$</samp></pre>


The continuation prompt, by default, is a greater than sign &#8211; &#8216;>&#8217; &#8211; that indicates that more input is expected. In addition to after a backslash-enter combination, it also appears when parentheses or quotes are left open (unmatched).

[^1]: OSX is not Linux, but uses the bash prompt and ships with a similar suite of tools. Certain commands may have slight syntax differences, however.
