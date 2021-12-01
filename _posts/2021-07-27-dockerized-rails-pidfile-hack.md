---
layout: post
title: 'Dockerized Rails pidfile Hack'
published: true
author: alexander
comments: true
date: 2021-07-27 14:30:49
tags:
    - docker
categories:
    - docker
permalink: /dockerized-rails-pidfile-hack
---
I usually fix things quickly when they repeatedly add friction. But every once in a while, something slips through.

One of those things for me was Rails' server pidfile in a dockerized dev environment. Any time a container exited abnormally, the pidfile would be left behind, causing the the container to fail to start the next time:

<pre>
<samp>A server is already running. Check /var/www/my_app/tmp/pids/server.pid.
Exiting.</samp>
</pre>

My docker-compose.yml was configured to sync my local Rails app with the container so I could make changes locally and see them in the containerized app. Unfortunately, changes in the app container, such as that pidfile, also sync back to my local dev environment. There are several ways to handle this, but here's the one I settled on: creating a tmpfs mount for the pids directory.

```yaml
      - type: tmpfs
        target: /var/www/my_app/tmp/pids
```

In context:

```yaml
version: '2'

services:
  app:
    build: my-app
    command: bin/rails server --port 3000 --binding 0.0.0.0
    links:
      - db
    ports:
      - "3000:3000"
    volumes:
      - './my-app:/var/www/my_app'
      - type: tmpfs
        target: /var/www/my_app/tmp/pids
```
