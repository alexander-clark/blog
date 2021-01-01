---
layout: post
title: "Benefits of Keyword Arguments"
date: 2016-03-20 20:38:11 -0500
tags: [ ]
categories:
     - Uncategorized
permalink: /benefits-keyword-arguments
---
When I first heard about keyword arguments in Ruby, I’ll admit I regarded them as a bit of a toy. Part of this was because they were only supported by Ruby 2.0 (2.1 if you wanted required kwargs), and at the time most of my projects were using 1.9. But I also didn’t see, at first, how they fundamentally differed from using a params hash.

## What are Keyword Arguments?

```ruby
# Normal ruby method
def foo(first_arg, second_arg={})
end

# Ruby method with params hash
def foo(params)
  first_arg = params[:first_arg]
  second_arg = params[:second_arg] || {}
  fail ArgumentError, 'first_arg is required' if first_arg.nil?
end

# Keyword args
def foo(first_arg:, second_arg: {})
end
```

What does this do for us? The second and third options allow us to make a little trade. Instead of having to remember the order of the arguments, we just have to remember their names.

{% highlight ruby linenos %}
# Call normal ruby method
foo('qux', bar: :baz)

# Call method that takes params
foo(first_arg: 'qux', second_arg: { bar: :baz })
# Out of order
foo(second_arg: { bar: :baz }, first_arg: 'qux')

# Call kwargs method
foo(first_arg: 'qux', second_arg: { bar: :baz })
# Out of order
foo(second_arg: { bar: :baz }, first_arg: 'qux')
{% endhighlight %}

## When Should I Use Keyword Arguments?

Is this a good trade?

It depends. From one perspective, [connascence][1] of name leads to looser coupling than connascence of position. Looking at it this way, keyword arguments are always a good trade.

On the other hand, look how much shorter and more readable option one is, both to define, and to call. There are particular circumstances that strongly indicate the use of keyword arguments:

First, if your method signature is likely to change. Adding new param, especially in the middle of the list, can break every piece of code that calls your method, or worse, introduce subtle bugs while appearing to continue to work.

Second, if you have a long list of params. It’s a lot easier to get two or three arguments right in a method call than six or seven.

## Another Use For Keyword Arguments
Keyword arguments can also be mixed and matched with other types of parameters. This could be used to write Smalltalk/ObjectiveC type methods, possibly in a DSL as an alternative to the traditional currying style:

Traditional:  expect(thing).to_receive(:foo).with(1, 2)

Smalltalk-esque:  expect(thing, to_receive: :foo, with: [1, 2])

[1]: https://practicingruby.com/articles/connascence
