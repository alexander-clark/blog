---
layout: post
title: 'Rails Event Store Minimal Example'
published: true
author: alexander
comments: true
date: 2025-06-21 07:57:58
tags:
    - rails
    - event sourcing
    - ruby
categories:
    - uncategorized
permalink: /rails-event-store-minimal-example
---
Sometimes, when learning a new tool or framework, it can be helpful to see the concepts at work using a very small example. Since event sourcing is especially complicated, I wanted to write the blog post I couldn't find when I was first trying to grasp these concepts.

This post will be more helpful if you're familiar with building Rails applications the "normal" way, but I've tried to include enough detail in the instructions that anyone can follow along. I won't be explaining how event sourcing works, just how to build an event sourced application with Rails Event Store.

First, a bit of background. I'll be using CQRS - command query responsibility segregation - in this app. In practice, this means two separate models -  a write model, and a read model. The read side will look quite familiar if you've built with Rails before - `index` and `show` actions and an `ActiveRecord` model - but the write side is quite different.

I've adopted some peculiar conventions here, which which are not standard practice in the Rails Event Store community, in order to make things easier to grasp for those familiar with Rails, but not with event sourcing.

* I added some new folders to the `app/` directory: `command_handlers/`, `commands/`, `denormalizers/`, and `events/`, as well as `read/` and `write/` subdirectories in the `models/` folder. Typically, the read model would go directly in the `models/` folder, and the rest of these objects would live one level up from the Rails application to encapsulate these concerns separately from the plumbing provided by Rails.
* The concept of a denormalizer is not usually distinguished from any other event handler within the RES community.

### Creating the App

The quickest way to get up and running with Rails Event Store is to use the Rails template. For this example, I'll use the name 'res-todo' since, you guessed it, it's a to-do list app.

```sh
rails new -m https://railseventstore.org/new res-todo &&\
cd res-todo
```

### Controller
I'll start with the controller.

You can see CQRS in action here. As mentioned, the `index` and `show` actions will use the read model, so we let Rails do its thing with those. For the write action, though, we're taking a step away from CRUD and embracing the full flexibility of REST. Instead of a `create` action using a model for persistence, I've added an action to add tasks, `POST add`, where we send a command. The command takes the place the model normally would in terms of accepting arguments and providing validation. More on command objects in the next section.

Since we're event sourcing, events will be published to a stream for the object we're initializing. In order to have the "added" event in the correct stream, we already need an ID before the event exists, so we can't use the normal Rails mechanism to grab the auto-increment ID after creation. Because of this, the command expects us to provide an ID, in this case a UUID. If this were an API, the UUID could come from the frontend.


app/controllers/tasks_controller.rb
```ruby
class TasksController < ApplicationController
  def index
    @tasks = Read::Task.all
    @command = AddTaskCommand.new
  end

  def show
    @task = Read::Task.find(params[:id])
  end

  def add
    command = AddTaskCommand.new(**task_params.merge(task_id: SecureRandom.uuid))
    if command.valid?
      Rails.configuration.command_bus.call(command)
      flash[:success] = 'Task added'
      redirect_to tasks_path
    else
      @command = command
      @tasks = Read::Task.all
      flash[:error] = 'Task could not be added'
      render :index, status: :unprocessable_entity
    end
  end

  private

  def task_params
    params.require(:task).permit(:name).to_h.symbolize_keys
  end
end
```
### Command
Let's look at the command.

A command object encapsulates a user instruction. In this case, the instruction is to add a task. "Add" is just one possibility for the verb here. Contrary to the normal Rails way, we want to be a bit intentional about explicitly not using "create," but instead trying to match the language that would be used by someone talking about performing the action in the real world. You might "add" something to a to-do list. Those familiar with GTD might also talk about "capturing" a task. The point is, when you're not approaching things with a CRUD way of thinking, there's very little that gets "created."

app/commands/add_task_command.rb
```ruby
class AddTaskCommand
  include ActiveModel::Model
  include ActiveModel::Validations

  validates :name, presence: true

  def initialize(task_id: nil, name: nil)
    @task_id = task_id
    @name = name
  end

  attr_reader :task_id, :name
  alias :aggregate_id :task_id
end
```

I included `ActiveModel::Model` and `ActiveModel::Validations` to allow us to use Rails' normal `validates` method. Because we're using CQRS, the write model doesn't inherit from `ActiveRecord::Base`. As we saw in the previous section, the controller action also doesn't call the write model directly, but instead fires this command. So one of the roles of the command is to work like a form object.

But what actually happens when we send the command as an argument to `command_bus.call`?

It gets handled by a command handler, of course. Let's build that next.

### Command Handler

app/command_handlers/add_task_command_handler.rb
```ruby
class AddTaskCommandHandler
  include CommandHandler

  def call(command)
    with_aggregate(Write::Task.new, command.aggregate_id) do |task|
      task.add(task_id: command.aggregate_id, name: command.name)
    end
  end
end
```

Here we're working with an aggregate - a write model. We need to provide an ID, which we've already collected in the command. The `with_aggregate` method, implemented in the included `CommandHandler` module, is a bit like `find_or_initialize_by_id`. If an object of the specified type with the specified ID doesn't exist, it's instantiated with the parameters provided. If it does exist already, then there would be an event stream. All of the events in that stream are replayed in order to get a write model in its current expected state.

Once we have an up to date write model, we can make changes. Since we decided "add" was the language we wanted to use for creating a task, `#add` is the method that does that. Had we chosen capture, that would be the method.

Let's take a look at the write model.

### Write Model
app/models/write/tasks.rb
```ruby
module Write
  class Task
    include AggregateRoot

    def initialize(task_id: SecureRandom.uuid)
      @task_id = task_id
    end

    def add(task_id:, name:)
      apply TaskAdded.new(data: { task_id:, name: })
    end

    on TaskAdded do |event|
      @task_id = event.data.fetch(:task_id)
      @name = event.data.fetch(:name)
    end

  end
end
```

We include aggregate root, which allowed us to use `with_aggregate` in the command handler. The initialize method provides only the bare minimum data needed for an event stream - in this case, just the ID. The `add` method (and corresponding `TaskAdded` event) is the first thing that happens in the lifecycle of a task. If it didn't happen, the object doesn't exist, so we can pretty much rely on it being there and setting any initial data. So that's where the name gets set. The `add` method's only job is to fire an event. That event gets handled in the on block, whether it's happening in real time, or when replaying events stored in the stream in a `with_aggregate` block. Here we actually set properties of the write model - instance variables. This is the model, but it's also very close to being a PORO.

OK, so how does this get persisted? There are two answers. The first is, because we're doing event sourcing, applying an event saves it to the event stream for the write model within the event store. So the next time we call with_aggregate using `Write::Task.new` and the same UUID, the `TaskAdded` event will get replayed and handled, setting the id and name.

Because we're doing CQRS, there's a second answer. We can also update a separate read model or cache to make it fast and easy to look up the current state of lots of records at once (think index action). How do we do that?

As we saw, the write model has a built-in event handler in the on block. But we can add more event handlers. There's a special type of event handler, sometimes called a denormalizer that's designed to update another model.

By the way, because we can have any number of event handlers, we can also have any number of read models. CQRS usually just has one, but the point of a read model is to store the data in a shape that's convenient for retrieval. Usually this is OLTP retrieval within the application, but it could also be one or more OLAP tables in a data warehouse.

What goes in an event?

### Event
Nothing. (Though we could add validations if we wanted to.)

app/events/task_added.rb
```ruby
class TaskAdded < RailsEventStore::Event
end
```

Easy-peasy. Let's look at a denormalizer.

### Denormalizer

A denormalizer is a type of event handler. Its job is to respond to events related to a write model by updating the associated read model.

app/denormalizers/task_denormalizer.rb
```ruby
class TaskDenormalizer
  def call(event)
    case event
    when TaskAdded
      task = Read::Task.new
      task.id = event.data.fetch(:task_id)
      task.name = event.data.fetch(:name)
      task.save!
    end
  end
end
```
For every type of event that might affect the read model, we have a corresponding `when` in the `case` statement. Because we're using Rails, the read model will just be a normal `ActiveRecord` model. However, we should never write to it directly - it should only be updated by the denormalizer.

### Read Model

app/models/read/task.rb
```ruby
class Read::Task < ApplicationRecord
end
```

This will almost always be pretty simple. Validations live in the command, event handlers take the place of ActiveRecord callbacks, and business logic tends to be in the write model.


### Migrations

If the read model is an `ActiveRecord` object, obviously we need a migration or two. The first one just enables UUIDs in Postgres.

To make it easy, you can run `rails g migration EnableUuidExtension`, then paste the contents of the `change` method into the generated file.

```ruby
class EnableUuidExtension < ActiveRecord::Migration[7.0]
  def change
    enable_extension 'pgcrypto'
  end
end
```

Then one to create the table for the read model. Run `rails g migration CreateTasks name:string`

```ruby
class CreateTasks < ActiveRecord::Migration[7.0]
  def change
    create_table :tasks, id: :uuid do |t|
      t.string :name

      t.timestamps
    end
  end
end
```

Edit the generated file to include `id: :uuid` so that we're using a UUID for the id. We set the id in the denormalizer so it matches the id of the write model.

Don't forget to run `rails db:migrate`

### Wrapping Up


All that's left is a bit of wiring up. We need to map the commands and events to their appropriate handlers and add a rails route.

config/initializers/rails_event_store.rb
```ruby
require "rails_event_store"
require "aggregate_root"
require "arkency/command_bus"

Rails.configuration.to_prepare do
  Rails.configuration.event_store = RailsEventStore::Client.new
  Rails.configuration.command_bus = Arkency::CommandBus.new

  AggregateRoot.configure do |config|
    config.default_event_store = Rails.configuration.event_store
  end

  # Subscribe event handlers below
  Rails.configuration.event_store.tap do |store|
    store.subscribe(TaskDenormalizer.new, to: [ TaskAdded ])

    # ...
  end

  # Register command handlers below
  Rails.configuration.command_bus.tap do |bus|
    bus.register(AddTaskCommand, AddTaskCommandHandler.new)
  end
end
```

config/routes.rb
```ruby
Rails.application.routes.draw do
  # ...

  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html
  resources :tasks, only: [ :index, :show ] do
    post :add, on: :collection
  end
  # ...
end
```

Including this module in our command handler lets us use `with_aggregate`, which makes things easier to understand.

lib/command_handler.rb

```ruby
module CommandHandler
  def initialize
    store = Rails.configuration.event_store
    @repository = AggregateRoot::InstrumentedRepository.new(
      AggregateRoot::Repository.new(store),
      ActiveSupport::Notifications
    )
  end

  def with_aggregate(aggregate, aggregate_id, &block)
    stream = stream_name(aggregate.class, aggregate_id)
    repository.with_aggregate(aggregate, stream, &block)
  end

  private

  attr_reader :repository

  def stream_name(aggregate_class, aggregate_id)
    "#{aggregate_class}$#{aggregate_id}"
  end
end
```

And of course a couple views

app/views/tasks/index.html.erb
```erb
<h1>Tasks</h1>

<div class="tasks">
  <% @tasks.each do |task| %>
    <%= render partial: 'task', locals: { task: task } %>
  <% end %>

  <h2>Add Task</h2>

  <%= render 'form', command: @command %>
</div>
```

app/views/tasks/_task.html.erb

```erb
<div class="task">
  <h2><%= task.name %></h2>
</div>
```

app/views/tasks/_form.html.erb

```erb
<div class="task">
  <%= form_for(command, as: :task, url: add_tasks_path) do |form| %>
    <div class="form-group">
      <%= form.label :name %>
      <%= form.text_field :name, class: (command.errors[:name].any? ? 'is-invalid form-control' : 'form-control') %>
      <% command.errors.full_messages_for(:name).each do |message| %>
        <div class="invalid-feedback d-block"><%= message %></div>
      <% end %>
    </div>
    <%= form.submit 'Add task', class: 'btn btn-primary' %>
  <% end %>
</div>
```

### See it in action

Run `rails s` and open `http://localhost:3000/tasks` to use the application. If you try to add a task without a name, you'll get a validation error, just like you would with a validation on an `ActiveRecord` model. After successfully adding a task, you can navigate to `http://localhost:3000/res`, to see the `TaskAdded` event.

### Next Steps

You might have noticed I didn't add an `update` action. This wasn't an oversight, but rather a way of keeping things simple. Just as I avoided a generic `create` action, `update` is likewise too generic. In fact, because a model is created only once, but may be updated multiple times in different ways, there could be several different actions for updating an object. In our todo list example, we might `rename` a task, then `complete` it, only to later `uncomplete` it, then `remove` (destroy) it. Thanks to event sourcing, we could even `restore` it after deleting, even if we had deleted the row from the read model's table. Each of these would require a route, a method in the controller, a command, a command handler, a method and `on` block in the write model, a case statement in the denormalizer, and views.

In case something didn't work or you'd like to see the final code, I've uploaded it to [GitHub](https://github.com/alexander-clark/res-todo)
