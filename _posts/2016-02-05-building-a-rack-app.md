---
layout: post
title: Building a Rack App / Ruby Web App
published: true
author: alexander
comments: true
date: 2016-02-05 02:02:27
tags:
    - rack
    - ruby
categories:
    - uncategorized
permalink: /building-a-rack-app
---
Have you ever wanted to build a super basic ruby web app? Maybe a single-page sort of thing, something that displays the temperature in your backyard or the number of emails in your inbox. The sort of thing where a simple HTML page won&#8217;t work, but a framework is overkill.

PHP got its start doing this sort of thing. While you can build object-oriented MVC frameworks with PHP, the reason so many people started using it was because if your server was set up for it, you could just rename index.html to index.php and paste in some code between . Done.

Alas, it&#8217;s never been quite that simple with Ruby, but we do have [Rack][1].

## About Rack

What&#8217;s Rack? It&#8217;s how frameworks like Sinatra, Rails, Hanami, etc. talk to web server software. Or as the Rack website puts it &#8220;Rack provides a minimal interface between webservers that support Ruby and Ruby frameworks.&#8221;

Also from the Rack website, here&#8217;s a super simple Rack app:

```ruby
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
```

This would go in the file config.ru. This file tells the rackup command what to do to run your application. (Ever notice that Rails apps have a config.ru?)

The command &#8216;run&#8217; tells it what to run. The thing it runs is a Proc that gets passed something called env, then returns an array with &#8216;200&#8217; (as in the HTTP status code OK), a hash of response headers, and an array of strings.

While it does a great job of showing just how simple the interface is, the code above isn&#8217;t very exciting or useful. Let&#8217;s make something that does stuff! Inspired by [isitchristmas.com][2], I bring you: Is it April Fools?

## Building a Rack App

To start with, let&#8217;s write an index.html.erb:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Is it April Fools?</title>
  </head>
  <body>
    <% if self.april_fools? %>
      <h1>YES</h1>
    <% else %>
      <h1>NO</h1>
    <% end %>
  </body>
</html>
```

Now let&#8217;s build something more classy/object-y than a proc. Here&#8217;s my\_rack\_app.rb:

```ruby
require 'erb'

class MyRackApp
  class << self
    def call(env)
      ['200', {'Content-Type' => 'text/html'}, view]
    end

    def view
      [ERB.new(template).result(binding)]
    end

    def template
      File.open('index.html.erb', 'r').read
    end

    def april_fools?
      Time.now.strftime('%m%d') == '0401'
    end
  end
end
```

In case you&#8217;re wondering, [ERB#result][3] accepts an optional instance of [Binding][4], which lets us pass in our current execution context. That&#8217;s how we&#8217;re able to use the self.april_fools? method within the erb file. If we didn&#8217;t need this, we could just call #result with no arguments, and it would be given a new context.

Because we defined self.call, the MyRackApp class will behave like a proc. This will allow us to make config.ru much prettier:

require './my_rack_app'

run MyRackApp

So easy!

## Running Our Rack App

To see it in action, use the rackup command in the same directory as your config.ru. rackup runs on port 9292 by default, so go to http://localhost:9292 in your browser to see it.

 [1]: http://rack.github.io
 [2]: https://isitchristmas.com
 [3]: http://ruby-doc.org/stdlib-2.3.0/libdoc/erb/rdoc/ERB.html#method-i-result
 [4]: http://ruby-doc.org/core-2.2.0/Binding.html
