---
layout: post
title: self in Ruby
published: true
author: alexander
comments: true
date: 2016-03-06 11:03:40
tags: [ ]
categories:
    - uncategorized
permalink: /self-in-ruby
---
Using self in Ruby comes naturally if you&#8217;ve been writing Ruby for any length of time. It&#8217;s easy to gloss over its meaning and significance, however.

Take, for instance, the following class.

```ruby
class Car
  attr_accessor :make

  def initialize(make)
    @make = make
  end

  def blend_in(new_make)
    make = new_make
  end
end
```

```ruby
car = Car.new('Chevy')
car.blend_in('Porsche')
#=> "Porsche"
car.make
#=> "Chevy"
```

Why didn&#8217;t #blend\_in work as expected? We missed a self. We should have written self.make = new\_make.

It&#8217;s easy and sometimes tempting to memorize heuristics like &#8220;always use self. when calling a setter.&#8221; While rote memorization can help keep beginners from getting mired in the details, at some point it&#8217;s important to dig into the why.

Obviously if we leave off self., we&#8217;re assigning a local variable. But why does Ruby do this?

Ruby specifically tries to make setters look like regular variable assignments. In most cases, this makes Ruby very simple and elegant. It can also introduce ambiguity. Does make = new\_make mean #make=(new\_make) or does it mean &#8216;assign the value of new_make to a new local instance variable called make?&#8217;

There&#8217;s no way for Ruby to know. Did you notice I clarified the second case by writing it out in English? That&#8217;s because there&#8217;s no clearer, more unambiguous way to write it in Ruby. Because there&#8217;s no other way to write it, if Ruby defaulted to using a setter when one existed, there would be no way to define a local variable with the same name as a setter.

While it&#8217;s generally not good practice to do so, there are cases where it might be useful.

## Using self in Ruby getters

So why don&#8217;t you need to use self. when calling a getter? In some cases, you do!

```ruby
class Car
  attr_accessor :make

  def blend_in(new_make)
    make = new_make
    puts make
    puts self.make
  end
end
```

```ruby
car = Car.new
car.make = "Chevy"
car.blend_in("Jaguar")
# prints "Jaguar" for make and "Chevy" for self.make
```
