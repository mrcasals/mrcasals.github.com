---
layout: post
title: "An approach to DataMapper, an alternative to ActiveRecord"
date: 2012-02-27 17:47
comments: true
categories: [Ruby, ActiveRecord, DataMapper]
---

**Note**: _I published this post for the first time at the [Codegram blog][codegram-blog-post], so this is just a repost :)_

A few days ago we started a new project at Codegram. The project (which is still in development) is a good challenge for us and a real motivator: we all were tired of using always the same technologies, so we decided to use some tools we hadn't had opportunity to use. One of them is [**DataMapper**][dm-website].

## So what is DataMapper and why using it?
{% blockquote DataMapper.org http://datamapper.org %}
DataMapper is an Object Relational Mapper written in Ruby. The goal is to create an ORM which is fast, thread-safe and feature rich.
{% endblockquote %}

{% blockquote Wikipedia http://en.wikipedia.org/wiki/Datamapper %}
Datamapper is an object-relational mapper library written in Ruby and commonly used with Merb. It was developed to address perceived shortcomings in Ruby on Rails' ActiveRecord library.
{% endblockquote %}

So, as the Wikipedia article says, DataMapper is an alternative to ActiveRecord. So why using it instead of the default ActiveRecord?

  * No need to write structural migrations
  * Scoped relations
  * Lazy loading on certain attribute types
  * Strategic eager loading
  * Default support for composite and natural keys

As you can see, DataMapper has some really interesting features. This post will be a little introduction to this ORM, let's see some of them.

<!--more-->

## Using DataMapper with Rails 3.1
First of all, we have to add Datamapper to our Rails project. I haven't found any metagem to include Rails and DataMapper basic gems, so we'll have to use this in our Gemfile:

{% codeblock Gemfile lang:ruby %}
source :rubygems

gem 'actionmailer'
gem 'actionpack'
gem 'activemodel'
gem 'activeresource'
gem 'activesupport'
gem 'tzinfo'

%w{core constraints migrations transactions timestamps do-adapter rails active_model sqlite-adapter}.each do |dm-gem|
  gem "dm-#{dm-gem}", git: "git://github.com/datamapper/dm-#{dm-gem}"
end

{% endcodeblock %}

You can also use the `data_mapper` gem, which installs the most commonly used DataMapper gems. Visit [its GitHub page][data_mapper-github] for more info.

### Our first DataMapper model
No more config needed! _What now?_ Just start coding! Don't forget to `bundle install` first! Everything right? OK, then, here comes the important part: our first DataMapper model.

{% codeblock A typical DataMapper model (app/models/book.rb) lang:ruby %}
class Book
  include DataMapper::Resource

  # Attributes
  property :id, Serial
  property :title, String
  property :synopsis, String
  property :edition, Integer

end
{% endcodeblock %}

_Wow! Amazing!_ Yeah! Looks beautiful, right? If we check the model line by line, we find that:

  1. The model no longer inherits from ActiveModel. Oh, wait, we are not using ActiveModel anymore, so looks legit.
  1. We include the [Resource DataMapper module][dm-resource-module], which adds some basics methods and includes the Assertions module.
  1. We define the `id` attribute, which will be a Serial. You can find more info about DataMapper data types [in the wiki][dm-wiki-properties].
  1. We define three more attributes: `title` and `synopsis` (which are `String`s) and `edition`, which is an `Integer`.

Now that we have our model written, we have to create the database before creating our first book. Once we enter `rake -T` on our shell to see all our Rake tasks, we find two new tasks:

  - `rake db:automigrate` will **destroy** all our data and update de database to the last version
  - `rake db:autoupdate` will update our database without destroying our data

_So what happens with `rake db:migrate`? I don't want to lose everything every time I migrate!_ Don't worry, it's aliased to `rake db:autoupgrade`, so it won't hurt you if you're too used to `rake db:migrate`.

We can now create our database with `rake db:create db:autoupgrade` and we can create our first book in the database! Let's open our beautiful Rails console and create our first book. `<spam>`_I'm using a real world example: a novel written by Patrick Rothfuss which all you fantasy-lovers should read. Oh, and people who just enjoy reading too :)_`</spam>`

{% codeblock lang:rbcon %}
:001 > book = Book.create(title: "The Name of the Wind", synopsis: "A rare find these days, fit for lovers of fantasy and newcomers to the genre alike.", edition: 20)
 => #<Book @id=1 @title="The Name of the Wind" @synopsis="A rare find these days, fit for lovers of fantasy and newcomers to the genre alike." @edition=20>
{% endcodeblock %}

_Wohoo! OMG it looks so beautiful!_ As I know you liked this way of defining the model's attributes, let's take a look at migrations before we go to the controller.

## Structural migrations and lazy-loading
_Structural migrations?_ I don't know another way to call them, so I call them this way. Structural migrations are those that change a database table structure: the ones that add or remove a field, add or remove primary keys, change a field type or add or create a table. That is, all the migrations that define the current database structure. Those migrations don't exist with DataMapper.

Let's see the usual procedure with ActiveRecord (actually, the list order is not important):

  1. Think about the model's attributes
  1. Add validations to the model
  1. Write a migration to create the model's table and add attributes

So we often create the migration at the last moment to ensure we don't forget any field. And I repeat, you won't need this last step with DataMapper. Let's take the example from above and suppose we want to modify some model attributes, so our Book model will now look like this:

{% codeblock Our DataMapper model (app/models/book.rb) after some changes lang:ruby %}
class Book
  include DataMapper::Resource

  # Attributes
  property :id, Serial
  property :title, String
  property :synopsis, Text
  property :author, String
  property :publication_year, Integer

end
{% endcodeblock %}

Notice we've added two new attributes, `author` and `publication_year`, and we have changed `synopsis`'s type from `String` to `Text`. Let's update our database with `rake db:migrate` and see what has changed:

{% codeblock lang:rbcon %}
 :001 > Book.first
 => #<Book @id=1 @title="The Name of the Wind" @synopsis=<not loaded> @author=nil @publication_year=nil>
 :002 > Book.first.synopsis
 => "A rare find these days, fit for lovers of fantasy and newcomers to the genre alike."
{% endcodeblock %}

So what has happened here? the database has been successfully updated to our last version: the book doesn't have any edition number and now has an author and a publication year. _And what about the synopsis? What does `<not loaded>` mean?_ DataMapper lazy-loads some data types to make database queries faster. This means that fields with a lot of data (like `Text` attributes) won't be loaded until they are required, like in this example.

### Other migrations
_What if I really need a migration?_ Well, that's a good point against DataMapper: this way of migrating and updating the database is really nice, but fails when we really need to change the data. There's a gem out there called [_dm-migrations_][dm-migrations] which could help you on that, but I haven't tried it, so I can't tell you anything about it right now. Sorry about that :(

## Beautiful scoped associations
Let's see another database-related feature from DataMapper: **scoped associations**. To see the point, we have to add two new models and modify our Book model a little bit:

{% codeblock app/models/author.rb lang:ruby %}
class Author
  include DataMapper::Resource

  # Attributes
  property :id, Serial
  property :name, String

  # Relations
  has n, :books

end
{% endcodeblock %}

{% codeblock app/models/book.rb lang:ruby %}
class Book
  include DataMapper::Resource

  # Attributes
  property :id, Serial
  property :title, String
  property :synopsis, Text
  property :publication_year, Integer

  # Relations
  belongs_to :author, key: false
  has n, :topics, through: Resource

end
{% endcodeblock %}

{% codeblock app/models/topic.rb lang:ruby %}
class Topic
  include DataMapper::Resource

  # Attributes
  property :id, Serial
  property :name, String

  # Relations
  has n, :books, through: Resource

end
{% endcodeblock %}

_Woah, not so fast! What does `through: Resource` mean?_ Another beautiful DataMapper feature! Stablishing relations through `Resource` makes DataMapper auto-create the table between `books` and `topics`, but that's not all! You want beautiful things? You'll have them. Open your `rails console` and enter the following after migrating the database:

{% codeblock lang:rbcon %}
 :001 > book = Book.first
 => #<Book @id=1 @title="The Name of the Wind" @synopsis=<not loaded> @publication_year=nil @author_id=0>
 :002 > book.author = Author.create(name: "Patrick Rothfuss")
 => #<Author @id=2 @name="Patrick Rothfuss">
 :003 > book.topics
 => []
 :004 > book.topics << Topic.create(name: "fantasy")
 => [#<Topic @id=1 @name="fantasy">]
 :005 > book.topics
 => [#<Topic @id=1 @name="fantasy">]
 :006 > book.save
 => true
 :007 > BookTopic.all
 => [#<BookTopic @book_id=1 @topic_id=1>]
{% endcodeblock %}

_WAIT AGAIN! What's that??_ Amazing, right? When defining a relation through `Resource`, DataMapper creates a class so that relation becomes an object! Let's try to destroy this one!

{% codeblock lang:rbcon %}
 :008 > BookTopic.first.destroy
 => true
 :009 > Book.first.topics
 => []
 :010 > BookTopic.create(book: Book.first, topic: Topic.first)
 => #<BookTopic @book_id=1 @topic_id=1>
 :011 > Book.first.topics
 => [#<Topic @id=1 @name="fantasy">]
{% endcodeblock %}

So you can play with relations modifying directly the relation object. Let's keep with the scoped associations we were talking about before and try this on our `rails console` again:

{% codeblock lang:rbcon %}
 :012 > Author.first.books.topics
 => [#<Topic @id=1 @name="fantasy">]
{% endcodeblock %}

That's how we get all the topics an author has written about without using dirty Arel Table relations, try this with ActiveRecord ;) Actually, associations are a really powerful resource with DataMapper. We could keep talking about them forever, but you can also take a look at the [DataMapper associations docs page][dm-associations-docs], which is pretty neat and clear about that. You can also take a look at the [Finding docs page][dm-finding-docs], where you'll find another amazing feature with associations: **queries combination**.

## Rails controllers with DataMapper
Just to end the post, I wanted to show you how a Rails controler looks with Datamapper. I've uploaded a gist with my BooksController so you can have a look at it:

{% gist 1268166 %}

As you see, it's very similar to an ActiveRecord one, except for one thing: **DataMapper doesn't have a `find` method**. So use the `get` method, which works pretty similar to the first one :)

## Final thoughts
I must say I love the DataMapper approach about the model, this way of clearly showing what attributes a model has and the clean way DataMapper migrates the database. I've been looking for a good way to have my models information properly sorted and easy to access, and I think I can achieve it with DataMapper and some design pattern like [Single Responsibility Principle][srp-pdf]. I just wanted to introduce you to DataMapper, an ORM not as used as ActiveRecord in Rails world, but I really think you should give it a try.

As a final note, you can find a debate on using DataMapper or ActiveRecord [here][dm-ar-debate]. You'll find some of the points explained in this post and some more to help you decide :)

[codegram-blog-post]: http://blog.codegram.com/2011/11/datamapper-an-alternative-to-activerecord
[data_mapper-github]: https://github.com/datamapper/data_mapper
[dm-website]: http://datamapper.org
[dm-wiki-properties]: http://datamapper.org/docs/properties.html
[dm-resource-module]: https://github.com/datamapper/dm-core/blob/master/lib/dm-core/resource.rb
[dm-migrations]: https://github.com/datamapper/dm-migrations
[dm-associations-docs]: http://datamapper.org/docs/associations.html
[dm-finding-docs]: http://datamapper.org/docs/find.html
[srp-pdf]: http://www.objectmentor.com/resources/articles/srp.pdf
[dm-ar-debate]: http://wrangl.com/datamapper-activerecord
