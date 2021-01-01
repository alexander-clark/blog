---
layout: post
title: Lessons Learned about Timestamps
published: true
author: alexander
comments: true
date: 2013-12-02 02:12:45
tags:
    - active record
    - postgres
    - rails
    - rails 2
    - ruby
    - time zones
    - timestamps
categories:
    - uncategorized
permalink: /lessons-learned-about-timestamps
---
I ran into an extremely frustrating issue lately where some Rails code periodically behaved in a very unexpected way.

After trying everything I could think of, I ran some code in the console to list the occurrences of the behavior, the user they happened to, and the time they happened at. A pattern quickly emerged. It was happening between 18:00 and 0:00 every day. Suspicious. Sounded like a time offset issue. It occurred to me to check something else, and sure enough: before daylight savings time ended, it happened between 18:00 and midnight. After it ended, it happened between 17:00 and 01:00. It perfectly matched my DST offset.

What was happening? Rails was writing timestamps to the database in local time, but when it went to pull them back out, it assumed they were in UTC and converted accordingly.

Why? One line of code in environment.rb:

```ruby
config.active_record.default_timezone = 'Mountain Time (US & Canada)'
```

The only explanation I can think of goes something like this: PostgreSQL doesn&#8217;t store UTC offsets with its timestamps by default. Rails 2.3 uses the default settings rather than forcing the type. When it encounters a timestamp without an offset, instead of checking default timezone, it assumes UTC.

As Rails 2 is no longer supported, there&#8217;s no sense in creating a pull request, but perhaps this will help someone else. My solution? Remove that line and let it save timestamps in UTC as recommended.
