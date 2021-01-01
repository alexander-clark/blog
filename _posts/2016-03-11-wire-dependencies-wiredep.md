---
layout: post
title: Wire Up Your Dependencies With Wiredep
published: true
author: alexander
comments: true
date: 2016-03-11 12:03:45
tags: [ ]
categories:
    - uncategorized
permalink: /wire-dependencies-wiredep
---
One of my favorite tools for building Angular apps is Wiredep. Manually including javascript and CSS dependencies into an HTML file can be a pain.

It&#8217;s not that big of a deal, really, but figuring out which files a bower package provides, remembering their paths, and typing them into the right spot in the right file can break you out of your flow state in a hurry.

Assuming you&#8217;re already using bower to manage your dependencies, it will probably take less time to get wiredep up and running than it would to set up another dependency without it.

First, install wiredep:

<pre><kbd>npm install --save-dev wiredep</kbd></pre>

Next, edit your top-level html file to tell wiredep to put the dependencies:

```html
<!doctype html>
<html>
  <head>
    <title>My awesome app</title>
    <!-- bower:css -->
    <!-- endbower -->
  </head>
  <body ng-app="awesomeness">
    ...
    <!-- bower:js -->
    <!-- endbower -->
  </body>
</html>
```

Now run the following:

<pre><kbd>wiredep -s [path/to/top-level-html]</kbd></pre>

Wiredep will insert all the CSS and javascript include tags for the files provided by your installed packages:

```html
<!doctype html>
<html>
  <head>
    <title>My awesome app</title>
    <!-- bower:css -->
    <link rel="stylesheet" href="bower_components/coolthing/dist/coolthing.css" />
    <!-- endbower -->
  </head>
  <body ng-app="awesomeness">
    ...
    <!-- bower:js -->
    <script src="bower_components/coolthing/dist/coolthing.js"></script>
    <!-- endbower -->
  </body>
</html>
```

Rerun the command each time you install a package with bower instead of manually adding the includes.

Be sure to keep any non-bower includes outside the bower/endbower sections or they will be clobbered next time you run wiredep.

For bonus points, [add a Grunt task][1] so you can just type grunt wiredep . With grunt-wiredep, there&#8217;s even a way to update your karma.conf.js too.

 [1]: http://stephenplusplus.github.io/grunt-wiredep/
