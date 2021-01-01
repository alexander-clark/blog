---
layout: post
title: 'Rails Conventions - Singular or Plural?'
published: true
author: alexander
comments: true
date: 2013-07-10 12:07:59
tags:
    - rails
categories:
    - uncategorized
permalink: /rails-conventions-singular-or-plural
---
So you probably know that Rails typically uses a singular name for models, e.g., User, and plural names for controllers, like UsersController. But what about other things you&#8217;re likely to encounter? Here&#8217;s a handy cheat sheet.

|-|-|-|
| Controller | Plural | <kbd>rails g controller Users index show</kbd> |
| Helper | Plural | <kbd>rails g helper Users</kbd> |
| Mailer | Singular | <kbd>rails g mailer UserMailer</kbd> |
| Migration | Plural | <kbd>rails g migration AddEmailToUsers email:string</kbd> |
| Model | Singular | <kbd>rails g model User name:string</kbd> |
| Observer | Singular | <kbd>rails g observer User</kbd> |
| Resource | Plural[^1] | `resources :users, :only => [:index, :show]` |
| Scaffold | Singular | <kbd>rails g scaffold User name:string</kbd> |
| Table | Plural | ``SELECT * FROM `users`;`` |
| View | N/A | app/views/users/index.html.erb &#8211; comprised of controller (plural) and action (singular) |

[^1]:Singular resources can also be made, but should be specified as such. For example, say each User `has_one` Account. You could create the Accounts controller (still plural) and add `resource :account` (both &#8216;resource&#8217; and the resource&#8217;s name are singular) to routes.rb. This assumes that the controller can find the correct Account without an id, e.g. @account = current\_user.account. resource does not create an index route for obvious reasons. The same controller can be shared between a plural and a singular resource, e.g. resource :account and resources :accounts both point to the Accounts controller. More in the [routing guide][1].

 [1]: http://guides.rubyonrails.org/routing.html#singular-resources
