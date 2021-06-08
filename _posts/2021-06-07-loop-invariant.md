---
layout: post
title: 'Loop Invariant'
published: true
author: alexander
comments: true
date: 2021-06-07 15:05:33
tags:
    - invariants
categories:
    - invariants
permalink: /loop-invariant
---
An invariant is a property of a program - or part of a program - that is always true. Ensuring that an invariant holds is one way to ensure that an algorithm works as intended.

One way invariants are employed is in loops. Because loops, by their very nature, involve lots of repetition with different values, it can be hard to write unit tests that anticipate the ways things might go wrong.

Sometimes developers include the invariant in the form of a comment, just to make their intentions clear to the next developer.

```ruby
Class BrokenList
  attr_accessor :arr

  def initialize(arr)
    @arr = arr
  end

  def max
    current_max = arr[0]

    arr.each_with_index do |current, i|
      current_max = highest(current, current_max)
      # Invariant: current_max == arr[0..i].max
    end

    return current_max
  end
end
```

What we're saying is that before the `each` block starts, at the end of every step within the `each` block, and after the `each` block has finished, it is true that `current_max` is equal to the maximum value in the array so far (i.e., up to the current index).

Let's check this with an arbitrary example. Say `arr` is equal to `[1, 3, 2]`.

Before the loop we set `current_max` equal to `arr[0]`, or `1`. `1` is indeed the largest number in `[1]`, so the invariant holds.

The first iteration is the same as our setup. It's a bit redundant, but it's one of several possible ways to prevent `current_max` from being nil.

For the second iteration, we use some magical `highest` method to get the larger of `current` or `current_max`, and assign the result to `current_max`. Assuming `highest` works as expected, we're setting `current_max` to `3`, which is the largest number in `[1, 3]`.

Finally, we use `highest` again to get the larger of `current` and `current_max`. `2` is not larger than `3`, so `3` is still the `current_max`.

This is a bit of a contrived example. If `arr[0..i].max` is a good invariant, `arr.max` is a perfectly suitable implementation.

We could make the invariant more specific if we wanted. One possible way of implemeting max would be to sort arr in place and returning the last item. Programmers usually know to avoid these kind of side effects, but we could make it explicit.

There's another way to take this a step further.

Imagine we have a subtle bug somewhere in our code, and that `max` returns an incorrect value a very small percentage of the time. We know that bad data is appearing very occasionally, and that it's somehow related to the max method, but we have no idea when or why. By raising an exception whenever the invariant is violated, we get helpful information - and maybe even a backtrace - in our logging utility.

```ruby
  def max
    current_max = arr[0]

    arr.each_with_index do |current, i|
      current_max = highest(current, current_max)

      invariant = current_max == arr[0..i].max
      raise "Violation: expected #{current_max} to equal #{arr[0..i].max} at index #{i}, current value: #{current}" unless invariant
    end

    return current_max
  end
```

If we're reluctant to raise in production, we could combine the change with a property-based testing tool such as Rantly.

```ruby
require 'rantly'
require 'rantly/rspec_extensions'
require_relative 'broken_list'

describe "BrokenList" do
  it "accepts an array of integers" do
    property_of {
      Rantly { array(range(0, 100)) { range(0,9000) }}
    }.check { |a|
      bl = BrokenList.new(a)
      expect(bl.max).to eq(a.max)
    }
  end
end
```

It doesn't take very many runs to find the culprit:

`Violation: expected 0 to equal 8899 at index 57, current value: 99`

If our code truly works for every number except 99, and if it silently continues with bad data instead of breaking, this could be a very hard bug to find. Thanks to a loop invariant and property-based testing, we found it in minutes.
