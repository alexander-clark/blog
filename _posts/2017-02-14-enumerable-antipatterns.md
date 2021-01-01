---
layout: post
title: Enumerable Anti-Patterns
published: true
author: alexander
comments: true
date: 2017-02-14 03:02:36
tags:
    - active record
    - enumerable
    - rails
    - ruby
categories:
    - uncategorized
permalink: /enumerable-antipatterns
---
[Enumerable methods][1] are one of my favorite parts of Ruby. Enumerable anti-patterns, however, can make things not-so-fun. Let&#8217;s explore some ways these helpful tools are abused, and how to fix them.

## Enumerable anti-patterns #1: Don&#8217;t use a shovel as a hammer

```ruby
old_array = [1, 2, 3]
new_array = []

old_array.each do |i|
  new_array << i*2
end
```

Instead:

```ruby
old_array = [1, 2, 3]

new_array = old_array.map { |i| i * 2 }
```

Explanation:

Whenever you initialize a variable to a new array, then shovel into that array from an each block, be suspicious. There are cases where this might actually be necessary, but usually what you&#8217;re doing is re-implementing a method that already exists: `#map`.

## Enumerable anti-patterns #2: Don&#8217;t use `#map` when you mean `#each`

```ruby
def send_notifications(mailer, email_addresses)
  email_addresses.map |address|
    mailer.enqueue_email(address)
  end
end
```

`#each` acts on every item in a collection without caring about the return value. It returns the collection you&#8217;re iterating over. `#map` acts on every item in a collection and **does** care about the return value. It returns a new collection containing the return value of all the operations it performed.

If you iterate over a collection, calling a method on each item where the **side effects** are the primary purpose of the operation &#8212; that is,  where you don&#8217;t care about the return values &#8212; using map can be confusing. In the example above, because we&#8217;re mapping over a collection on the last line of the method, the return value of the map operation will be the return value of our method. This could potentially delude a reader of our code into thinking we care about this information when, in fact, we do not.

Instead:

```ruby
def send_notifications(mailer, email_addresses)
  email_addresses.each |address|
    mailer.enqueue_email(address)
  end
end
```

By using `#each`, we&#8217;re telling our reader that we don&#8217;t care about the return values of our enqueue operations.

Counter example:

```ruby
def send_notifications(notifier, users)
  success = users.all? |user|
    notifier.deliver_notification!(user)
  end

  return success
end
```

Here we want to know whether every notification was delivered successfully, and to return to the caller whether or not this was the case. Using an explicit return, while not strictly necessary, indicates in no uncertain terms what our intentions are: we are returning a value.

## Enumerable anti-patterns #3: Selecting instead of detecting

```ruby
collection.select { |item| item.id == id }.first
```

Instead:

```ruby
collection.detect { |item| item.id == id }
```

`#select` goes through the entire collection, finding all matching entries. If there should only be zero or one matches, or if you only care about the first match you find, use `#detect` instead. Similarly, if you only want to know **whether** there are any matches, use `#any`?:

```ruby
# Not optimal
collection.select { |item| item.id == id }.any?

# Not optimal
collection.detect { |item| item.id == id }.present?

# Better
collection.any? { |item| item.id == id }
```

Counter example:

```ruby
big_items = collection.select { |item| item.tags.include?('big') }
```

If you really do want all the matches, by all means use `#select`

You might be noticing a pattern here: if you find yourself chaining enumerable methods together, there might be room for improvement.

Speaking of select:

## Enumerable anti-patterns #4: Over-using enumerable methods with ActiveRecord

```ruby
Post.all.to_a.detect { |post| post.id == params[:id] }
```

The projects we work on as developers influence our comfort level with various parts of Ruby and/or Rails. If you&#8217;re more comfortable working with collections and Enumerable, you might not immediately notice what&#8217;s wrong with the above. We&#8217;re searching a collection of posts for the first one that matches params[:id].

If you&#8217;re used to using ActiveRecord, you would probably write this as

```ruby
Post.find(params[:id])
```

For a small hobby app with a few dozen posts, the difference in performance is likely negligible. But for a production app with millions of records, the latter is the clear winner. SQL &#8211; particularly with a primary key &#8211; is purpose-built for this kind of task. Finding a single record using an index with extremely high [cardinality][2] is about the cheapest operation in SQL.

Pulling all records from the database, loading them into memory, then running each through a block until one returns true, on the other hand is rather inefficient.

Enumerable methods are better for smaller collections, cases where SQL operations aren&#8217;t available, or where you aren&#8217;t operating directly on database tables or ActiveRecord models.

## Final Thoughts

These are just a few examples of using enumerable methods in ways they&#8217;re not meant to be used. The reason these misuses are bad is not because they&#8217;re &#8220;wrong&#8221; or even because they&#8217;re inefficient. Code is meant for people as much as (if not more than) it is for computers. We&#8217;re communicating our intentions to the next person to come along. Taking the time to communicate clearly what we intend makes it easier to review pull requests. It also makes it easier to make modifications down the road. Be kind to others.

 [1]: https://alexander-clark.com/blog/shorten-map-commands/
 [2]: https://en.wikipedia.org/wiki/Cardinality_(SQL_statements)
