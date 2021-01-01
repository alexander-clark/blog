---
layout: post
title: 'The Ruby Logger - Keep a log'
published: true
author: alexander
comments: true
date: 2015-12-18 01:12:53
tags:
    - logging
    - ruby
categories:
    - uncategorized
permalink: /the-ruby-logger
---
You&#8217;ve used the Ruby Logger if you&#8217;ve ever built a Rails app, or even started up a development server and watched the output scroll by. But because Rails provides logging out of the box, it&#8217;s not often necessary to think about it.

If you&#8217;re building a plain old ruby app, however, logging is something you might think about implementing at some point. When that time comes, Ruby&#8217;s built-in Logger class has your back.

## Instantiating a Ruby Logger

```ruby
require 'logger'

logger = Logger.new($stdout)
```

That&#8217;s all there is to it.

Of course instead of writing to stdout, you could also write to stderr, a file (and optionally set up rotation of log files) or even a custom class that implements write and close.

The custom class option is useful if you want to log to both stdout and a file, want to create a dummy log for testing, or if you are using something like [highline][1] to do your IO.

## Configuring your Log

### Severity threshold

Logger has the following severity levels from least to most severe:

`Logger::DEBUG`, `Logger::INFO`, `Logger::WARN`, `Logger::ERROR`, `Logger::FATAL`, and `Logger::UNKNOWN`

These are the constants you can set the severity threshold to. (They correspond to 0, 1, 2, 3, 4, and 5 respectively, but use the constants whenever possible.) Anything below the severity threshold will be ignored.

Set the threshold like so:

```ruby
logger.level = Logger::WARN
```

Better yet, set environment-specific values in the environment&#8217;s [dotenv][2] file.

### App name

```ruby
logger.progname = 'MyApp'
```

### Formatter

```ruby
logger.formatter = Formatter.new
```

The default is an instance of the Formatter class. If you want to specify your own, you can subclass Formatter, or use a proc. The following example would print only the message, ignoring everything else.

```ruby
logger.formatter = proc { |severity, time, progname, msg| msg }
```

## Sending messages to the Ruby Logger

The Ruby Logger has a method for each of the error levels

```ruby
logger.info "Doing things now!"
#...
logger.error "Failed to complete task"
```

For more information, check out the [documentation][3].

 [1]: https://github.com/JEG2/highline
 [2]: https://github.com/bkeepers/dotenv
 [3]: http://ruby-doc.org/stdlib-2.1.0/libdoc/logger/rdoc/Logger.html
