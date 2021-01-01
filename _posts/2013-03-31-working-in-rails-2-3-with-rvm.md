---
layout: post
title: 'Working in Rails 2 & 3 with rvm'
published: true
author: alexander
comments: true
date: 2013-03-31 05:03:39
tags:
    - rails
    - rails 2
    - ruby
    - rvm
categories:
    - uncategorized
permalink: /working-in-rails-2-3-with-rvm
---
I had wanted to play with Rails 2 a bit, without breaking any of the Rails 3 apps I&#8217;m working on. I use rvm anyway, so I thought it would be pretty straightforward. I found [instructions][1] for setting it up, but ran into a couple snags.

First I installed Ruby 1.8.7 and created a gemset.

<pre>
  <samp>$ </samp><kbd>rvm install 1.8.7</kbd>
  <samp>$ </samp><kbd>rvm use 1.8.7</kbd>
  <samp>$ </samp><kbd>rvm gemset create rails2</kbd>
  <samp>$ </samp><kbd>rvm use 1.8.7@rails2</kbd>
</pre>

Then installed rails 2.3.11:

<pre>
  <samp>$ </samp><kbd>gem install rails --version 2.3.11</kbd>
</pre>

The first problem I ran into was that rvm had put rake 10.0.4 in the global gemset for ruby 1.8.7. This caused an error when running rake tasks:

<pre>
  <samp>ERROR: 'rake/rdoctask' is obsolete and no longer supported. Use 'rdoc/task' (available in RDoc 2.4.2+) instead.</samp>
</pre>

To fix this:

<pre>
  <samp>$ </samp><kbd>rvm use 1.8.7@global</kbd>
  <samp>$ </samp><kbd>gem uninstall rake</kbd>
  <samp>$ </samp><kbd>rvm use 1.8.7@rails2</kbd>
  <samp>$ </samp><kbd>gem install -v=0.8.7 rake</kbd>
</pre>

It was also using the latest rubygems. This caused another error when running the gems:install rake task:

<pre>
  <samp>undefined method &#96;name&#39; for "actionmailer":String</samp>
</pre>

To fix:

<pre>
  <samp>$ </samp><kbd>rvm rubygems 1.6.2</kbd>
</pre>

Now you should be able to play. You may want to brush up on the [differences][2] between Rails 2 and 3 if it&#8217;s been a while.

<pre>
  $ rails r2test
  $ cd r2test
</pre>

[uncomment the sqlite3-ruby line in config/environment.rb]

<pre>
  <samp>$ </samp><kbd>rake gems:install</kbd>
  <samp>$ </samp><kbd>script/generate scaffold User name:string</kbd>
  <samp>$ </samp><kbd>rake db:migrate</kbd>
  <samp>$ </samp><kbd>script/server</kbd>
</pre>




Voilà.

Pro tip: set up an rvmrc for your Rails 2 projects so rvm automatically loads the correct ruby version & gemset when you cd into the directory.

<pre>
  <samp>$ </samp><kbd>rvm rvmrc create 1.8.7@rails2</kbd>
  <samp>$ </samp><kbd>rvm rvmrc trust</kbd>
</pre>

[1]: http://stjhimy.com/posts/10-five-quick-steps-to-set-up-rvm-with-rails-2-and-rails3
[2]: http://rubyonrails.org/screencasts/rails3
