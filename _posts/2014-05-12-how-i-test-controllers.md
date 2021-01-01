---
layout: post
title: How I Test Controllers
published: true
author: alexander
comments: true
date: 2014-05-12 05:05:20
tags:
    - controllers
    - rails
    - rspec
    - ruby
    - testing
categories:
    - uncategorized
permalink: /how-i-test-controllers
---
I&#8217;ve been trying to improve the specs I write lately. My method before was mostly copying and pasting from my past projects. Somewhere way back I adapted them from an early version of Michael Hartl&#8217;s Rails Tutorial, making various modifications over the years.

Recently I decided to scrap my specs and start over from scratch. Since my controller specs were particularly ugly, I decided I&#8217;d start with them.

I wanted to share what I learned, so here goes. In true test-driven fashion, we&#8217;ll write a little one-controller app, specs first. Our app will be a list of favorite books. We&#8217;ll start with the most basic skeleton:

```ruby
require 'spec_helper'

describe BooksController do

  describe '#index' do
  end

  describe '#show' do
  end

  describe '#new' do
  end

  describe '#edit' do
  end

  describe '#create' do
  end

  describe '#update' do
  end

  describe '#destroy' do
  end
end
```

Because this app is so simple, there will be nothing at all interesting in our model, and thus nothing to test. We&#8217;ll trust ActiveRecord to do its job since that code is already tested ;) Here&#8217;s our model:

```ruby
# == Schema Information
#
# Table name: books
#
# id :integer not null, primary key
# title :string(255)
# author :string(255)
# description :text
# created_at :datetime
# updated_at :datetime
#

class Book < ActiveRecord::Base
end
```

Now let&#8217;s start with index. What do we expect an index action to do?

  * We need to get an array of all books from ActiveRecord via the model.
  * We need to assign that array to a variable for the view
  * We need to render the index template
  * We need to indicate to the browser that the request was successful (HTTP status 200)
  * We don&#8217;t need to set any flash messages
  * There&#8217;s really only one path to test. The only abnormal instance would be an empty array of books, but the controller doesn&#8217;t really know or care whether this happens &#8211; just the model and the view.

In the past I would have done a Factory Girl create in a before block to test getting the array. But the controller isn&#8217;t really responsible for getting the ActiveRecord object, only asking the the model for it and passing the result on to the view. Besides, we should probably write integration specs to make sure everything works together, so we&#8217;re going to test that anyway. That&#8217;s why these days I like to use mocks in testing my controllers.

Let&#8217;s go ahead and create a Factory Girl factory anyway. We can use `attributes_for`  as a convenient way to get our params hash, and the factory will be useful later for our integration specs and possibly model if we add anything interesting like validations.

```ruby
FactoryGirl.define do

  factory :book do
    title 'Pride and Prejudice'
    author 'Jane Austen'
    description '...'
  end
end
```

Time to write our first failing spec

```ruby
let(:book) { mock_model(Book) }
let(:book_attributes) { FactoryGirl.attributes_for(:book).stringify_keys }

describe '#index' do

  before do
    expect(Book).to receive(:all).once.and_return([book])
    get :index
  end

  it "does something" do
  end
end
```

And we get our failure: Book didn&#8217;t receive all. Let&#8217;s modify the index action in the controller:

```ruby
def index
  Book.all
end
```

And it passes. Now let&#8217;s write our first real spec.

```ruby
describe '#index' do

  before do
    expect(Book).to receive(:all).once.and_return([book])
    get :index
  end

  it 'assigns the instance variable' do
    expect(assigns(:books)).to eq([book])
  end
end
```

Of course it fails, because while we called all on Book to get our first expectation to pass, we didn&#8217;t assign it to a variable for the view. Let&#8217;s fix that.

```ruby
def index
  @books = Book.all
end
```

And it passes. On to our next spec. It should respond with success. Because this is the normal response for rails, let&#8217;s set ourselves up for failure.

```ruby
def index
  @books = Book.all
  render status: 404
end
```

And write our spec:

```ruby
it { should respond_with(:success) }
```

Failure. Let&#8217;s fix it and set up our next failure.

```ruby
def index
  @books = Book.all
  render layout: false
end
```

Success.

```ruby
it { should render_with_layout :application }
```

Failure.

```ruby
def index
  @books = Book.all
  render layout: 'application', text: ''
end
```

Success.

```ruby
it { should render_template :index }
```

Failure.

```ruby
def index
  @books = Book.all
end
```

One could argue that it seems excessive to write code to make such simple specs fail, and I&#8217;ll readily admit I don&#8217;t always do this. But this is what the rhythm of TDD should feel like, and it&#8217;s not completely safe to trust specs you haven&#8217;t seen fail.

#show, #new, and #edit will be very similar &#8211; there&#8217;s only one path and they simply render a view. I&#8217;ll include the full code at the end.

The next interesting challenge is *#create. Let&#8217;s take a crack at that.

What do we expect a create action to do?

  * We need to instantiate a new Book with the params we&#8217;re given
  * We need to assign the object to a variable for the view in case we need to display it
  * We need to ask the book to persist itself
  * If the save reports success, we want to:
      * Set a flash message indicating success
      * Redirect to index
      * Because it&#8217;s a redirect, index is responsible for anything that happens after that. We&#8217;ve already tested that.
  * If the save fails, we want to:
      * Render the new template
      * Indicate to the browser that the request was successful (HTTP status 200) &#8211; despite the fact that the save was not
      * We don&#8217;t need to display any flash messages

```ruby
def create
  render :new
end
```

```ruby
describe '#create' do

  before do
    expect(Book).to receive(:new).with(book_attributes).once.and_return(book)
    post :create, book: book_attributes
  end

  it "does something" do
  end
end
```

Failure &#8211; Book didn&#8217;t receive new

```ruby
def create
  Book.new(book_params)
  render :new
end

#...

private

def book_params
  params.require(:book).permit(:title, :author, :description)
end
```

Success. Speaking of which, it&#8217;s time to branch our specs for success vs failure.

```ruby
describe '#create' do

  before do
    expect(Book).to receive(:new).with(book_attributes).once.and_return(book)
  end

  context 'success' do

    before do
      expect(book).to receive(:save).once.and_return(true)
      post :create, book: book_attributes
    end

    it "does something" do
    end
  end

  context 'failure' do
  end
end
```

It fails.

```ruby
def create
  book = Book.new(book_params)
  book.save
  render :new
end
```

It passes.

```ruby
context 'success' do

  before do
    expect(book).to receive(:save).once.and_return(true)
    post :create, book: book_attributes
  end

  it { should redirect_to books_url }
end
```

It fails.

```ruby
def create
  book = Book.new(book_params)
  book.save
  redirect_to books_path
end
```

It passes.

```ruby
context 'success' do

  before do
    expect(book).to receive(:save).once.and_return(true)
    post :create, book: book_attributes
  end

  it { should set_the_flash.to('Book was successfully created.') }
  it { should redirect_to books_url }
end
```

It fails.

```ruby
def create
  book = Book.new(book_params)
  book.save
  flash[:notice] = 'Book was successfully created.'
  redirect_to books_path
end
```

It passes. The failure path behaves virtually identically to the new action. We&#8217;ll look at a way to exploit this and other similarities to DRY up our specs in a follow-up post on shared examples. For now, notice that book is a local variable. This allows us to call save while still setting up our spec to check whether the book is assigned to an instance variable for failure.

Here is the final create action and corresponding spec:

```ruby
def create
  @book = Book.new(book_params)

  if @book.save
    flash[:notice] = 'Book was successfully created.'
    redirect_to books_path
  else
    render :new
  end
end
```

```ruby
describe '#create' do

  before do
    expect(Book).to receive(:new).with(book_attributes).once.and_return(book)
  end

  context 'success' do

    before do
      expect(book).to receive(:save).once.and_return(true)
      post :create, book: book_attributes
    end

    it { should set_the_flash.to('Book was successfully created.') }
    it { should redirect_to books_url }
  end

  context 'failure' do

    before do
      expect(book).to receive(:save).once.and_return(false)
      post :create, book: book_attributes
    end

    it { should respond_with(:success) }
    it { should render_with_layout :application }
    it { should render_template :new }

    it 'assigns the instance variable' do
      expect(assigns(:book)).to eq(book)
    end
  end
end
```

So what are the takeaways?

  * Write specs that test the thing you&#8217;re testing. Use mocks. This makes tests easier to write, easier to read, and super fast. On my machine, 29 specs run in less than two tenths of a second without Spork.
  * Describe the behavior of the thing you&#8217;re testing. Do this before you write your specs. When you&#8217;re done, your specs are documentation. Try rspec spec --format documentation
  * Follow the rhythm of TDD: red, green, refactor. For easy stuff like this, refactoring might not be necessary.

It should go without saying, but these are just guidelines. Fast tests are a means, not an end. There are cases where mocking becomes awkward. TDD is a tool, not a [dogma][1].

I&#8217;ve posted the [finished code][2] on GitHub.

I hope this is helpful to others, and I welcome feedback.

 [1]: https://alexander-clark.com/blog/tdd-dogma-and-professional-courtesy/
 [2]: https://github.com/alexander-clark/booklist
