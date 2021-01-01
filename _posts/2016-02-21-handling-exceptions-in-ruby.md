---
layout: post
title: Handling Exceptions in Ruby
published: true
author: alexander
comments: true
date: 2016-02-21 01:02:51
tags: [ ]
categories:
    - uncategorized
permalink: /handling-exceptions-in-ruby
---
## Handling Exceptions in Ruby is Easy!

Exceptions are Ruby&#8217;s way of allowing us to attempt to recover from errors. The basic syntax for handling exceptions in Ruby goes something like this:

```ruby
begin
  do_something_risky
rescue
  recover_from_exception
end
```

We can gather more information from an exception like so:

```ruby
begin
  do_something_risky
rescue StandardError => e
  puts e.message
end
```

Notice we rescued StandardError and not Exception. There are many types of exception you might not necessarily want to rescue. For example, pressing CTRL+C or sending SIGTERM to the process would not work as expected and you might have to resort to SIGKILL.

## Implicit Contexts

It&#8217;s not always necessary to explicitly type begin. Class, module, and method definitions and ruby blocks serve as implicit rescue contexts.

```ruby
def safe_method
  do_something_risky
rescue StandardError => e
  logger.error e.message
  logger.debug e.backtrace.join("\n")
end
```

Rescuing exceptions is a good use case for a [logger][1].

## Multiple Exception Handling

You can handle multiple types of exception differently. Just be sure to put specific exception types before StandardError if rescue it.

```ruby
begin
  do_something_risky
rescue WhizBangError => e
  handle_specific e
rescue StandardError => e
  handle_generic e
end
```

## Re-raising Exceptions

You can re-raise an exception after doing something with it. raise knows to re-raise the exception in the rescue context.

```ruby
begin
  do_something_risky
rescue StandardError => e
  logger.error e.message
  raise
end
```

## Retrying

You can retry the error-prone task after rescuing.

```ruby
def safe_method
  retried = false
  begin
    do_something_risky
  rescue NotLoggedInError
    log_in
    unless retried
      retried = true
      retry
    end
  end
end
```


Just be sure to check whether you&#8217;ve already retried first to avoid an endless loop if the fix doesn&#8217;t work.

## Cleaning Up

In some situations there&#8217;s code that needs to run whether there&#8217;s an exception or not. The most common example is closing a file handle

```ruby
begin
  file = File.open 'file', 'w'
  write_to_file file
rescue
  handle_file_error
ensure
  file.close unless file.nil?
end
```

## Defining Your Own Exceptions

```ruby
class BarlessFooError &lt; StandardError
end
```

## Failing

fail BarlessFooError, 'Foo requires a bar.' if foo.bar.nil?

Use fail rather than raise to surface an exception unless you are reraising as above.

 [1]: http://alexander-clark.com/blog/the-ruby-logger/
