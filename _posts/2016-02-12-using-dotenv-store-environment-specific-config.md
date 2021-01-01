---
layout: post
title: Using dotenv to store environment-specific config
published: true
author: alexander
comments: true
date: 2016-02-12 02:02:55
tags: [ ]
categories:
    - uncategorized
permalink: /using-dotenv-store-environment-specific-config
---
## What Is dotenv

[dotenv][1] sets [environment variables][2] from a .env file,  to be accessed via [ENV][3] in Ruby. Using dotenv allows you to specify important global, environment-specifc configuration settings like API keys and secrets, usernames and passwords, and URIs in a file that&#8217;s not checked in to version control. These settings can then be accessed at run time using `ENV['VARNAME']`.

### Oh you mean like config/database.yml or config/environments/**.rb in Rails?

A lot like that. However, those files are generally checked in to version control software. It&#8217;s not a good idea to have important production passwords in your repo.

## Why Should I Use dotenv

Fundamentally, configuration is not part of your application. It doesn&#8217;t belong in the same repo as the app itself. Putting it there can create numerous problems:

  * Developers accidentally or naïvely committing changes that work for them but not others to a shared development-specific config file, breaking other developers&#8217; dev environments
  * Configuration containing sensitive passwords getting accidentally checked into version control, potentially leading to a security breach
  * Creation of additional environments for QA, staging, etc. that are &#8216;just like production, except&#8230;&#8217; leading to bloat and churn in config files

## Using dotenv

  1. Add .env to your .gitignore
  2. Add dotenv to your Gemfile:
  `gem 'dotenv-rails'` if you&#8217;re using Rails or `gem 'dotenv'` otherwise
  3. Create the file .env in the root directory of your project and specify variables inside, one per line, like so: `MY_CONFIG_VAR=ABC123`
  4. If you&#8217;re using Rails, you should be able to access `ENV['MY_CONFIG_VAR']`, but certain cases may require more tweaking, so see the [readme][4] if you have trouble
  5. If you&#8217;re using another framework or building a plain old Ruby app, add this somewhere that will be loaded before you need to access any variables:
  ```ruby
  require 'dotenv'
  Dotenv.load
  ```
  6. Set the same variables in a .env file for each environment (staging, production, etc.) and place these directly on each server (or with the code for configuring/provisioning servers)
  7. You&#8217;re done! 

Strictly speaking, using dotenv in production is not recommended, so long term it may be worthwhile to find another way of setting the necessary environment variables. That said, many do use it in production without issue.

By way of example for step six, if you use Capistrano to deploy your app, the production .env file would live in the shared directory and be symlinked during deploy.

 [1]: https://github.com/bkeepers/dotenv
 [2]: https://en.wikipedia.org/wiki/Environment_variable
 [3]: http://ruby-doc.org/core-2.2.0/ENV.html
 [4]: https://github.com/bkeepers/dotenv/blob/master/README.md
