---
layout: post
title: Shorten Map and Each Commands
published: true
author: alexander
comments: true
date: 2015-11-25 01:11:28
tags:
    - each
    - enumerable
    - map
    - proc
    - ruby
categories:
    - uncategorized
permalink: /shorten-map-commands
---
## Or: Pithy Procs for Working with Enumerables

Typing out code blocks every time you want to use map or each is a bit of a pain, and often unnecessary.

You may know that `ruby(1..5).map { |n| n.to_s }` can be rewritten as `(1..5).map &:to_s`.

But what does the latter really mean?

The ampersand lets Ruby know you&#8217;re passing something to be used as a block and not an argument. It also calls `#to_proc` on the something. `Symbol#to_proc` is implemented in C, but for our purposes, it maps to something roughly like this:

```ruby
def to_proc
  proc{ |caller| caller.send(self) }
end
```

Here&#8217;s another example:

`(1..5).each { |n| puts(n) }` can be rewritten as `(1..5).each &method(:puts)`.

`Method#to_proc` is also written in C, but the documentation includes a ruby equivalent:

```ruby
def to_proc
  proc{|*args|
    self.call(*args)
  }
end
```

What about something like `(1..5).map { |n| n * 2 }`? `(1..5).map &2.method(:*)`.

What else can we pass in? Anything with `to_proc`. Lambdas? Check.

```ruby
foo = ->(n, i) { puts "#{i}: #{n}" }
(1..5).each_with_index &foo
```

## Implementing #to_proc

Why not add a `to_proc` method to a command class?

```ruby
class PurchaseCommand
  attr_accessor :shipment

  def initialize(customer)
    @shipment = Shipment.new(customer)
  end

  def call(item)
    item.deduct_inventory
    shipment.items << item
   end

  def to_proc
    proc{ |item| self.call(item) }
  end
end

def make_purchase
  ...
  cart_items.each &PurchaseCommand.new(customer)
  ...
end
```

In the above, only one `PurchaseCommand` is initialized; the same command is used for all items in the cart. It might be a bit clearer to write it this way:

```ruby
cmd = PurchaseCommand.new(customer)
cart_items.each &cmd
```

This also allows us to hold a reference to the command for later use, if, for example we wanted do something with its shipment.

## Not just for map and each!

This isn&#8217;t limited to each and map.

```ruby
(1..10).partition &:odd?
```

Be careful, though. While `arr.reject &:nil?` is shorter than `arr.reject { |i| i.nil? }`, `arr.compact` is shorter still, and arguably more intention-revealing. At least if you know your enumerable methods.
