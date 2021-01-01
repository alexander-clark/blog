---
layout: post
title: Generators in Rails 2.3
published: true
author: alexander
comments: true
date: 2013-08-15 01:08:56
tags:
    - generators
    - rails
    - rails 2
categories:
    - uncategorized
permalink: /generators-in-rails-2-3
---
The other day I started looking in to how to create generators in Rails 2.3. I came across [a couple][1] of [articles][2] on the topic that provided a good starting point, still was a bit confused about a couple things.

I couldn&#8217;t figure out exactly how to get information to templates at first. The solution: simply add an `attr_reader` corresponding to the instance variable in `initialize()`. In this case, hello.

```ruby
class FoosGenerator < Rails::Generators::Base
  attr_reader :hello
  attr_accessor :attributes

  def initialize(runtime_args, runtime_options = {})
    super
    @hello = runtime_args.first
    @attributes = []
  end

  def manifest
    m.template foo.erb
  end
end
```

and in templates/foo.erb, just put an erb tag with a local variable name.

```erb
<%= foo %>
```

At this point, you might be wondering, like I did, &#8220;What about erb files? How do you differentiate between erb to interpret now in the generation process and that for later when a template is rendered, say?&#8221;

```erb
<%% for later %>
<%%= also for later %>
<%%= <%= for now %> %>
<% for now %>
```

Note that the double % occurs only in the opening erb tag.

When I discovered that the first link above contained a link to the [code][3] for rails 2.3 generators and dug around a bit in there, most of this became clear pretty quickly.

Something that took a bit more effort to suss out was how to accept runtime options that had a string value instead of boolean.

```ruby
class FoosGenerator < Rails::Generators::Base
  ...
  protected

  def add_options!(opt)
    opt.separator ''
    opt.separator 'Options:'
    opt.on("--boolean-option", "Set boolean option to true") { |v| options[:boolean_option] = v }
    opt.on("-d", "--description=name", String, "Specify a description", "Default: mysql") { |v| options[:description] = v }
  end
end
```

Note that the &#8220;Default: mysql&#8221; bit is just part of the help output. You&#8217;d have to write logic to actually make this function as promised.

 [1]: http://blog.wyeworks.com/2010/9/23/creating-your-own-generators-on-rails-2-3/
 [2]: http://patshaughnessy.net/2009/9/2/rails-generator-tutorial-part-2-writing-a-custom-manifest-action
 [3]: https://github.com/rails/rails/tree/2-3-stable/railties/lib/rails_generator/
