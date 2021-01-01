---
layout: post
title: "Override a Model's destroy Method Without Losing Callbacks"
published: true
author: alexander
comments: true
date: 2013-06-14 01:06:26
tags:
    - active record
    - callbacks
    - rails
    - rails 2
    - ruby
    - soft delete
categories:
    - uncategorized
permalink: /override-a-models-destroy-method-without-losing-callbacks
---
I needed to do this in Rails 2, and had trouble finding any anything about how it could be done. I did eventually find [some information][1], so I thought I&#8217;d share.

Here&#8217;s how it looked for me:

```ruby
class User < ActiveRecord::Base

  has_many :attributes

  def before_destroy
    ...
  end

  def destroy_without_callbacks
    unless new_record?
      update_attribute(:deleted_at, Time.now)
    end
  end
end
```

Also came across an [article][2] talking about some of the downsides to soft delete. Definitely a good reminder.

 [1]: http://blog.crowdint.com/2011/05/02/How-to-override-destroy-with-callbacks.html
 [2]: http://richarddingwall.name/2009/11/20/the-trouble-with-soft-delete/
