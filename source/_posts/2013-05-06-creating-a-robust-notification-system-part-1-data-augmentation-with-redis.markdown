---
layout: post
title: "Creating a Robust Notification System - Part 1: Data Augmentation with Redis"
date: 2013-05-06 18:03
comments: true
categories: [Code, Ruby, Rails, Programming, Redis]
---

Over the last few months I've been working on a personal project, and in the next three blog posts I will be talking a bit about one portion of the overall system. I have created what I think it a pretty cool notification and messaging system, during which time I explored several design patterns that were new to me. I would like to share my experience and newfound knowledge. Thanks for reading!


Introduction
------------

The basic goal of this series is to discuss how to create a flexible notification system. This system is not designed to be a personal messaging system; its primary purpose is not to allow users to send messages to each other, but rather to alert a user about some change of state in the overall application. These notifications may be messages sent only to an admin user, alerting them to a new user registration, or perhaps to notify a user about a new comment in a thread they are a part of.

In this post we will be discussing how the notifications and messages themselves are persisted. Later posts will focus on how we create and display the notifications, but for now we will only concern ourselves with how to store the data.

<!-- more -->

The Basics
----------

As I've mentioned in a [previous blog post](http://blog.eebsy.com/blog/2013/01/21/nginx-and-unsupported-http-headers/), I play an MMO called [EvE Online](http://www.eveonline.com/), where players fly spaceships around in a gigantic virtual world. There are [many things](http://i.imgur.com/Kom642W.jpg) a player can do within the EvE world, but the part I'm involved in is the creation of some of the largest ships in the game. The application I created helps me keep track of my entire production chain, allowing me to see when I need to log in to the game to update a task, or start a new build process. Because there are so many things going on in real time, I quickly found I needed a central place I could see important changes to the overall application. I needed to be able to create notifications when specific events happened, and then be able to review those notifications.

At a very basic level, I needed to be able to send a user a notification containing some message about the state change. I also wanted to be able to send the same message to multiple users, in the event that I needed to send out a site wide announcement to all users, or a notification to all admins. I started by creating a Message model for holding the content of the message in addition to the User model I already had. The Message model simply has **title** and **body** fields, of type string and text respectively. There is a many to many relationship between Users and Messages. If you have the simplest many to many relationship, Rails provides the [*has_and_belongs_to_many* association](http://guides.rubyonrails.org/association_basics.html#the-has_and_belongs_to_many-association), which simply creates a join table with two columns for the foreign keys. However, I needed to store a little bit more information, such when the message was read by the user, so I created a Notification join model. The Notification model has **user_id** and **message_id** fields, along with a **read_at** timestamp. I connected these three models with Rails' [*has_many_through* association](http://guides.rubyonrails.org/association_basics.html#the-has_many-through-association).

{% img /images/notification_association.png 750 360 Notification Association %}

Here's what the code for my models ended up looking like:

{% codeblock User Model - user.rb %}
class User < ActiveRecord::Base

  # Other properties, validation, etc

  has_many :notifications
  has_many :messages, :through => :notifications

  def notify(message)
    messages << message
  end

  def unread_notifications
    notifications.where(:read_at => nil)
  end
end
{% endcodeblock %}

{% codeblock Message Model - message.rb %}
class Message < ActiveRecord::Base

  attr_accessible :body, :title

  has_many :notifications
  has_many :users, :through => :notifications
end
{% endcodeblock %}

{% codeblock Notification Model - notification.rb %}
class Notification < ActiveRecord::Base
  attr_accessible :message_id, :read_at, :user_id

  belongs_to :user
  belongs_to :message
end
{% endcodeblock %}

Great! Now at the very least I can create a message with a title and body, and send it to one or more users by using the *notify* method on a User instance. I can also fetch all unread notifications for that user.

Adding Additional Data
----------------------

When I created the initial implementation, I thought I was effectively done at this point. However, as I began to create messages, and add content to the **body** field, I quickly realized I had a problem. Any time I needed to reference another object in the message, I needed to first fetch that object, grab the data I needed, and hard code it into the message.

The other issue is that any formatting I wanted to add to the text in the body had to be hard coded directly into the body of the message. This irked me for a few reasons, the primary one being that I hate putting view information into my models. It also meant that if I ever wanted to change the formatting of a message, it would only change from that point forward, there would be no good way to change the display of previously created messages.

I realized I needed to scrap the concept of the **body** field, but rather associate other models or data with the message, in a less tightly coupled manner. I actually keep the **body** field around so don't be surprised if you see references to it later, I'll explain why in Part 2. Let me give you a concrete example of the data I need to associate. In the application, there is the concept of a Run. A Run essentially represents the entire production process for a single Ship. A User is able to reserve a Run, through a Reservation. When a User reserves a Run, I would like the system to create a notification for all admin Users.

Here are a couple examples of messages the system may need to display.

> [Eebs](#) reserved [Archon 12](#) 4 minutes ago.

or

> Order for Archon has changed:<br/>
> Status changed from Open to Sold<br/>
> Volume Remaining changed from 1 to 0

The message that gets created for a new Reservation needs to tell me which User reserved what Run, with links to each. With the **body** method, I would need to fetch the User and Run objects, grab their names and id's, and hard code some \<a\> tags into the body of the message. Instead, I need to somehow tell the message about the Run and the User that it pertains to, preferably only storing their id's so I can simply fetch the objects when I display the message. I may also want to store additional data that the message may reference, such as a hash of data that gets displayed in a list in the message (as seen in the second example), so my message objects need to support storing somewhat arbitrary data.

Enter Redis
-----------

A couple months ago, I was watching the [videos from RailsConf 2012](http://www.confreaks.com/events/railsconf2012), and a particular one caught my attention. It was [Obie Fernandez's](http://obiefernandez.com/) talk on [Redis Application Patterns in Rails](http://www.youtube.com/watch?v=dH6VYRMRQFw). He talks about how his company uses Redis alongside ActiveRecord to leverage its scalability and performance benefits. I highly recommend watching the video before you continue reading, or at least go back and watch it after. The main point I took away from the talk was that using a second storage system to persist additional data alongside Active Record isn't hard, and often times will offer better performance that just using straight ActiveRecord with a disk based database.

For those of you that haven't used [Redis](http://redis.io/) before, it is a NoSQL, in-memory "advanced key-value store". You can store strings, hashes, lists, sets and sorted sets. This data is kept in-memory, allowing very fast retrieval. With some simple configuration, Redis can achieve PostgreSQL-like persistence, while maintaining its high performance levels.

In the talk, Obie discusses how it can be very advantageous to offload some data onto the Redis store. SQL databases are great for reporting, allowing us to make complex queries to gain insight into out data, but what if we don't need that reporting? What if we simply need to dump some additional attributes that belong to the model into our persistence layer, but don't want to clutter our database with data that will only ever be used in conjunction with the model it belongs to?

If we augment our models to allow them to store some data in the Redis store, we can improve performance, while keeping our database simple and concise. There are several method of doing so, which Obie talks about in the video. One of the methods he mentions is [Redis-Objects](https://github.com/nateware/redis-objects), a gem created by [Nate Wiger](http://www.nateware.com/). Redis-Objects allows you to very easily add values to your models that will automatically be stored in the Redis store when set.

Redis-Objects makes it very easy to add these Redis values to your models. I decided to use [Single Table Inheritance](http://martinfowler.com/eaaCatalog/singleTableInheritance.html) with my messages, allowing me to set different Redis-Objects values on each message model. I'll talk more about why I used STI in Part 2.

Updated Message model including Redis::Objects

{% codeblock Message Model - message.rb %}
class Message < ActiveRecord::Base
  include Redis::Objects

  attr_accessible :body, :title

  has_many :notifications
  has_many :users, :through => :notifications
end
{% endcodeblock %}

Here I add the **order_id** value, as well as the **changes** value, to the ChangedOrderMessage model, which extends the base Message model. **changes** is a hash of data, and using *:marshal => true* allows us to store complex data in the Redis store.

{% codeblock ChangedOrderMessage Model - changed_order_message.rb %}
class ChangedOrderMessage < Message
  value :order_id
  value :changes, :marshal => true
end
{% endcodeblock %}

{% codeblock NewReservationMessage Model - new_reservation_message.rb %}
class NewReservationMessage < Message
  value :reservation_id
end
{% endcodeblock %}

With the NewReservationMessage, I simply store the Reservation's **id**, allowing us to look up the Reservation when we display the message. It's not a good idea to store an entire object in Redis, but rather we simply store the **id** and fetch it at a later time. The User that created the Reservation is available via an association on the Reservation model which is why you don't see me storing the **user_id** on the Message, although I could have done so.

Now that we have our Redis-Objects values set up, creating a new Message and storing the additional data is simple:

{% codeblock %}
message = NewReservationMessage.create! :title => "New Reservation!"
message.reservation_id = @reservation.id
{% endcodeblock %}

{% codeblock %}
message = ChangedOrderMessage.create! :title => "Order Changed!"
message.order_id = @order.id
message.changes = changes # Here changes is a hash of data
{% endcodeblock %}

When we call *message.order_id = @order.id* Redis-Objects creates a key based on the object's **id**, and assigns *@order.id* to that key. Note that Redis-Objects requires the object it's included in have an **id** property. Because the **id** isn't set until the record is saved, I call *create!* to persist the data via ActiveRecord, and then assign the **order_id**.

When we are displaying the message, we can fetch the data like this:

{% codeblock %}
reservation = Reservation.find(message.reservation_id.value)
{% endcodeblock %}

{% codeblock %}
order = Order.find(message.order_id.value)
changes = message.changes.value
{% endcodeblock %}

I'll be discussing message display in Part 2, where you'll see the rest of the message body come into play.

Streamlining Redis Access
-------------------------

So far everything is pretty easy and straightforward. The additional data is saved to and fetched from the Redis store automatically, we just need to call *value* on the result from Redis. I'd like to make this and a few other things a bit easier though. You may have noticed that when I fetch the associated ActiveRecord object, I need to call *find* on that Model. This is a bit cumbersome, and I simplified this by creating a [Concern](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

{% codeblock RedisObjectRelations Module - redis_object_relations.rb %}
module RedisObjectRelations
  extend ActiveSupport::Concern

  module ClassMethods
    # If a message has a redis object in the form <name>_id
    # define a method <name> that loads the appropriate
    # Active Record object from the database.
    def has_redis_object_relations
      redis_objects.each do |object, options|
        if m = /^(?<name>.*)_id$/.match(object)
          name = m[:name]
          class_eval <<-EndMethods
            def #{name}
              @#{name} ||= #{name.camelize}.find(#{name}_id.value)
            end
          EndMethods
        end
      end
    end
  end
end
{% endcodeblock %}

I define a class method, *has_redis_object_relations*, that can be called in the individual message models after setting up the values. The *redis_objects* method returns a hash of the Redis-Object keys and their options, and so I can use that to create a method that performs the *find* call for me. I can then update the Message model to include *RedisObjectRelations*.

{% codeblock Message Model - message.rb %}
class Message < ActiveRecord::Base
  include Redis::Objects
  include RedisObjectRelations

  attr_accessible :body, :title

  has_many :notifications
  has_many :users, :through => :notifications
end
{% endcodeblock %}

{% codeblock ChangedOrderMessage Model - changed_order_message.rb %}
class ChangedOrderMessage < Message
  value :order_id
  value :changes, :marshal => true
  has_redis_object_relations
end
{% endcodeblock %}

Now it's even easier, all the Redis magic happens behind the scenes.

{% codeblock %}
reservation = message.reservation
{% endcodeblock %}

Wrapping Up
-----------

I've now covered how to use Redis to your advantage to store arbitrary attributes on models, allowing some flexibility in the data you associate with each of them. In my application, I've separated the data storage from the message display, enabling me to change the format of a message or add additional data later on.

In the next post, I'll discuss how I display the messages to users using the Presenter pattern, as well as talk about why STI is a good fit in this case.

Please feel free to leave any comments or questions below.
