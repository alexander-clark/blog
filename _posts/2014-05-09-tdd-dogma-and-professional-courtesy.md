---
layout: post
title: TDD, Dogma, and Professional Courtesy
published: true
author: alexander
comments: true
date: 2014-05-09 12:05:23
tags:
    - hard fucking science
    - railsconf
    - tdd
categories:
    - uncategorized
permalink: /tdd-dogma-and-professional-courtesy
---
I had the privilege of attending RailsConf last month and seeing DHH&#8217;s [keynote][1]. A lot of feathers were ruffled. Mine weren&#8217;t. Maybe that&#8217;s why I&#8217;ve been only peripherally aware of the ensuing debate. But I did watch today&#8217;s [discussion][2]. It reinforced what I thought was going on: people are stuck in their own point of view and talking past each other.

In his keynote, DHH challenged the idea of programming as hard science. I personally do think of myself as a software engineer and programming as hard science.

Hard. Fucking. Science.

But &#8211; if I understand correctly &#8211; I see what he&#8217;s getting at. Programmers learn design patterns, principles, and techniques and assume they are universal truths because science. The reality is, however, that even in hard science there&#8217;s often room for subjectivity and multiple approaches. Even in mathematics &#8211; [_the_ hard science][3] &#8211; there is often more than one way to arrive at a solution. There may even be many correct solutions, any one of which can be used, depending on what you&#8217;re doing.

I don&#8217;t think becoming French poets or software writers will make us immune to this sort of black-and-white thinking &#8211; text editor holy wars have been around as long as text editors have.

What will? Professional courtesy. If we see ourselves as craftspeople &#8211; whether that craft be  writing, engineering, or otherwise &#8211; shouldn&#8217;t we treat our fellow programmers with the respect we&#8217;d like to receive ourselves? Oughtn&#8217;t we withhold judgment of their process until we&#8217;ve seen the deliverable &#8211; the code?

Can we let go of our absolutes? Absolutes like &#8220;If you don&#8217;t write your tests first, your code is shit.&#8221; And like &#8220;Writing tests first leads to bad design.&#8221;

I&#8217;ll start.

I love TDD. The rhythm works for me. For many things. But sometimes I write tests last. And sometimes I don&#8217;t write tests at all. Good design drives good tests and good development &#8211; regardless of which happens first or whether testing happens at all. Testing is a tool, not the deliverable. Every problem is not a nail and testing is not a silver bullet.

I love using mocks and isolating the thing I&#8217;m testing. It makes my tests run really fast and I can easily isolate what I broke and where. But they sometimes come at a cost and sometimes that cost isn&#8217;t worth it.

I like RSpec and TextMate 2, but there are plenty of other great tools for writing code. I like design patterns and principles and learning them has made me a better coder. But there isn&#8217;t a design pattern for every programming problem. Sometimes principles conflict and there are tradeoffs to be made. Good code is a little bit subjective. We sometimes have to choose between DRYer code and code that&#8217;s easier to read. Between code that&#8217;s faster and code that&#8217;s more easily reusable.

If everyone did things the same way, how on earth would we learn from each other?

 [1]: http://www.confreaks.com/videos/3315-railsconf-keynote-writing-software
 [2]: https://www.youtube.com/watch?v=z9quxZsLcfo
 [3]: http://xkcd.com/435/
