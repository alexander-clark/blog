---
layout: post
title: Test names and readability
published: true
author: alexander
comments: true
date: 2016-08-11 03:08:52
tags:
    - minitest
    - rspec
    - ruby
    - testing
categories:
    - uncategorized
permalink: /test-names-readability
image:
    feature: Screen-Shot-2017-12-30-at-9.02.11-AM.png
---
## What&#8217;s in a Name?

It&#8217;s important to choose a useful name for a test. If tests are documentation, test names are the chapter headings.

## Test Names Should Not Include Should

There&#8217;s nothing wrong with the word &#8220;should.&#8221; It&#8217;s just not necessary. It&#8217;s one more word to read when trying to understand a test. &#8220;It should divide by four&#8221; and &#8220;It divides by four&#8221; express the same idea in a test name.

```ruby
it "foo should return the correct answer when passing in a date" do
end
```

There are several issues with the above. First of all, it contains should. Second, it runs roughshod over &#8216;it.&#8217; If I read the line out loud, &#8220;it foo&#8230;&#8221; does not make sense. Also, that&#8217;s a lot of words. _As a developer, I want to be able to skim the tests so that I can find the test case I&#8217;m looking for._

```ruby
describe "foo" do
  describe "when passing in a date" do
    it "returns the correct value" do
    end
  end
end
```

Isn&#8217;t that better? In addition to being more readable, it makes it painfully obvious that we&#8217;re missing another test case. Let&#8217;s add it.

```ruby
describe "foo" do
  describe "when passing in a date" do
    it "returns the correct value" do
    end
  end

  describe "no date passed in" do
    it "returns the correct value" do
    end
  end
end
```

## Be Specific

Now we have another issue: two test cases called &#8220;returns the correct value.&#8221; Come to think of it, that&#8217;s also pretty generic. Probably most test cases could be called &#8220;it returns the correct value,&#8221; &#8220;it has the intended side-effects,&#8221; or &#8220;it has the intended side effects and returns the correct value.&#8221; The question is, how would you describe the value and/or side-effect we&#8217;re looking for? Not the actual value, but its significance or meaning.

```ruby
describe "foo" do
  describe "when passing in a date" do
    it "returns the count of bars before the date" do
      date = Date.today

      assert_equal 2, foo(date)
    end
  end

  describe "no date passed in" do
    it "returns the count of all bars" do
      assert_equal 4, foo
    end
  end
end
```

Cool. Now in this one-assertion test it doesn&#8217;t matter as much, but say we returned the bars themselves instead of the count and needed to make several assertions. Like test names, variable names in assertions improve readability.

```ruby
expected_count = 2
expected_dates = ['2011-04-15', '2017-01-01']

result = foo

assert_equal expected_count, result.count
assert_equal expected_dates, result.map(&:date)
```

## A Bit of Context

These tests are written for [Minitest][1]. If you&#8217;re [using RSpec][2], you have an additional tool at your disposal: The context block.

```ruby
describe "foo" do
  context "when passing in a date" do
    it "returns the count of bars before the date" do
    end
  end
end
```


Use `describe` when specifying the subject of the test &#8211; the class or method. Use `context` when talking about the circumstances under which we&#8217;re testing.

## Takeaways

Let&#8217;s write some tests for our tests:

  * it &#8220;begins with a verb, not a subject&#8221; (&#8216;it&#8217; is the subject)
  * it &#8220;reads like a sentence&#8221; (&#8216;it&#8217; is the first word of the sentence)
  * it &#8220;does not contain when or if&#8221; (those belong in describe blocks)
  * it &#8220;is short&#8221; (and skimmable)
  * it &#8220;doesn&#8217;t use should as the verb&#8221;
  * it &#8220;is specific about the expected result&#8221; (what is the significance of the correct or expected result?)

 [1]: https://github.com/seattlerb/minitest
 [2]: https://alexander-clark.com/blog/how-i-test-controllers/
