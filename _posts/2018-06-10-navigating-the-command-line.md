---
layout: post
title: 'Navigating the Command Line - Getting Around'
published: true
author: alexander
comments: true
date: 2018-06-10 01:06:46
tags:
    - command line
    - linux
categories:
    - linux-command-line
permalink: /navigating-the-command-line
---
*Author's note: I've been using the Linux command line since 2003, but I often meet developers who struggle to understand it. I've also found that teaching can be a great way to deepen one's own understanding. After starting to write this series, I ran across Julia Evans' [Wizard Zines](https://wizardzines.com). Julia has a rare gift for explaining incredibly technical topics in ways that are helpful for beginners and experts alike. I highly recommend her work.*

Navigating the command line is mostly a matter of moving between directories, and understanding where you are &#8211; the current working directory. As mentioned previously, when working on the command line, there is always a current working directory.

## Checking the Current Working Directory

Usually it&#8217;s displayed in the command prompt, but it can also be found at any time with the pwd command. pwd stands for &#8220;print working directory.&#8221;

<pre><samp>user@localhost:~$ </samp><kbd>pwd</kbd>
<samp>/home/user</samp>
<samp>user@localhost:~$</samp></pre>


To get a list of files in the working directory, use the ls command.

<pre><samp>user@localhost:~$ </samp><kbd>ls</kbd>
<samp>my_file  my_folder</samp></pre>


ls has lots of options &#8211; also called &#8216;flags&#8217; &#8211; that can be passed to change the behavior. The -a flag shows all files in a directory. Normally, files starting with a dot &#8211; &#8220;.&#8221; &#8211; are hidden. This includes two special folders found in every folder, &#8220;.&#8221; and &#8220;..&#8221;.

<pre><samp>user@localhost:~$ </samp><kbd>ls -a</kbd>
<samp>.  ..  my_file  my_folder</samp></pre>


These are usually pronounced as &#8220;dot&#8221; and &#8220;dot-dot.&#8221; Dot refers to the current working directory. Dot-dot refers to the parent of the current directory; that is, the directory that contains the current directory.

In addition to these two special files, any normal files that start with a dot will only show up if you use the -a flag. These are sometimes referred to as hidden files.

The cd command changes the working directory.

## An Example of Navigating the Command Line

<pre><samp>user@localhost:~$ </samp><kbd>pwd</kbd>
<samp>/home/user
user@localhost:~$ </samp><kbd>ls</kbd>
<samp>my_file  my_folder
user@localhost:~$ </samp><kbd>cd my_folder</kbd>
<samp>user@localhost:my_folder$ </samp><kbd>pwd</kbd>
<samp>/home/user/my_folder
user@localhost:my_folder$ </samp><kbd>ls</kbd>
<samp>user@localhost:my_folder$ </samp><kbd>ls -a</kbd>
<samp>.  ..
user@localhost:my_folder$ </samp><kbd>cd ..</kbd>
<samp>user@localhost:~$ </samp><kbd>pwd</kbd>
<samp>/home/user</samp></pre>


In this imaginary session, the user checks the working directory (even though it appears in the prompt indicated by ~). They list the files and see a folder called my\_folder. They cd into the folder, and the prompt changes to reflect the change. Checking pwd shows the full path to the folder. They use ls to list the contents of my\_folder, but it&#8217;s empty. They then employ the -a flag to show all files, including ones starting with a dot. Only our two special files are there, dot, and dot-dot. They cd dot-dot back up to their home directory and check the working directory again.

This example is a bit contrived, or perhaps just involves an inexperienced user. The more comfortable one becomes navigating the command line, the less frequently pwd is necessary. This is because it becomes easier to know what command to type to get to the desired directory, and to predict what the working directory will be after typing the command. cd and ls, on the other hand, are the bread and butter of navigating the command line for novices and experts alike.

## Absolute Paths and Relative Paths

There are two basic ways to specify a path: absolute and relative. A relative path is specified in terms of the working directory. An absolute path is specified in terms of the root directory, /. If the working directory is /home/user, my\_folder is a relative path. The absolute path would be/home/user/my\_folder.

The example above involves moving up or down one directory at a time. Both absolute and relative paths can be used to make larger jumps. For example, if the working directory is /home/user/my_folder, it&#8217;s possible to get to the root directory, /, in one command. The command using a relative path would be cd ../.. The command using an absolute path would be cd /

Why are there two ways? Relative paths are often (but not always) shorter to type. cd my\_folder is much shorter than cd /home/user/my\_folder. A counterexample from above would be cd ../.. vs cd /. In this case, the absolute path is shorter.

The main reason for using absolute paths, however, is that they are valid regardless of the working directory. cd my\_folder works if the working directory is /home/user, but not if it&#8217;s /. cd /home/users/my\_folder, on the other hand, works regardless of the current directory.

## Tilde paths

Just as the ~ character is often used in the prompt to indicate that the working directory is the user&#8217;s home directory, it can be used in specifying a path. For example, cd ~ or cd ~/my\_folder. It can also be used in conjunction with a username to specify another user&#8217;s home directory, for example cd ~otheruser/their\_folder. These shortcuts are like absolute paths, and can be used regardless of the working directory. Note that permissions have to be configured in such a way that you have access to the other user&#8217;s folder for this to work.
