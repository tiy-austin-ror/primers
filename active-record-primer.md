# Active Record: A Primer
##Description
To paraphrase the Rails guide on Active Record a bit, Active Record is the M in MVC. It is the model layer of the system, which is responsible for representing business rules and logic. Active Record is also an ORM in addition to an implementation of the Active Record structural design pattern. Active Record allows us to define models in our program by writing Ruby classes that inherit from `ActiveRecord::Base`. This gives our models all of the methods needed to persist the data they hold in the database.

We use Active Record models to 'wrap' the raw data found in our database tables in a Ruby class that we can fill with our own methods and custom behaviors.

## Terms
- **A Model** A Ruby class that inherits from `ActiveRecord::Base` and represents a single row (or record) in a database table.
- **SQL** Structured Query Language. The language spoken by almost all (relational) databases.
- **Migration** A process that takes your database structure from Point A to Point B by following a set of steps.
- **Record** A row in the database, normally used in reference to individual model instances. "Get the first record from the users table by using `User.first`" In that sentence, the result of `User.first` will be an instance of the model `User`, but it is common to hear that referred to as a 'record'.

##Intro
Active Record models correspond to tables in your database. If your program has a class called `User` that subclasses `ActiveRecord::Base` then there is an implicit assumption that you also have a table in your database called `users`. Note the plurality and casing of both of those as they are important. With Active Record, naming is extremely important.

A file called `user.rb` is *expected* to contain a class called `User` and that there will be a table in the database called `users`.

There are ways to overrule this - for instance, when you're working with a database that wasn't created with Rails in mind. However, these are the defaults and they'll serve you well when starting from scratch.

Active Record maintains a connection to your database and is responsible for taking your Ruby objects and converting them to a format your database can understand. This is primarily done by taking your requests to an Active Record object (and instance of any model you've defined) and converting that to SQL.

Active Record will look at each of your models when it is loaded and inspect the corresponding table in the database to give your model all the additional behavior that it needs.

For example, if you have a `User` model that looks like this:

```
class User < ActiveRecord::Base
end
```
and in your database you have a table named `users` with a structure like this:

| ID | first_name | last_name  | age|
|----|----        |----        |----|
| 1  | 'joe'      | 'albright' | 30 |
| 2  | 'sue'      | 'kim'      | 28 |

Then Active Record will create **attributes** for each column it finds in your database. So your `User` model, while still empty in `user.rb`, will contain methods for `id`, `first_name`, `last_name`, and `age`.

Active Record also gives you a pair of very important methods on each instance of your model: `.save` and `.save!`. These are the methods used to take the *current state* of your model and persist (or, save) it to the database.

Demonstration with PRY

```
[2] pry(main)> User.count # Get the current amount of users
=> 50
[3] pry(main)> me = User.new # Create a new user
=> #<User:0x007fda43d2aaa8 id: nil, first_name: nil, last_name: nil, email: nil>
[4] pry(main)> me.first_name = "Justin" # assign values to the attributes
=> "Justin"
[5] pry(main)> me.last_name = "Herrick"
=> "Herrick"
[6] pry(main)> me.email = "justin@theironyard.com"
=> "justin@theironyard.com"
[7] pry(main)> User.count # See if the user count has changed
=> 50 # It has not changed. Our database has the same number of rows
[8] pry(main)> me.save # Let's save our object
=> true
[9] pry(main)> User.count # Now let's check our row count
=> 51 # One more user now
```

## Most Used Methods
I'm going to be using my `User` model for all these examples, but bear in mind these will work with almost any models you have. In no particular order:
###1. `User.new`
 `User.new` is a way of creating a new model object the same way we create an instance of **any** Ruby class. So this is no surprise. What Active Record allows you to do is pass in a `Hash` of keys that correspond to the attributes on your model. So we could create a new user with:

```
User.new({ first_name: "justin",
            last_name: "herrick",
                email: 'justin@theironyard.com' })
```
_(Note: the above model object is not persisted until saved)_

###2. `User.create`
If you need to instantiate a model object and immediately save it to the database, you can do that in one line using `.create`. This is very similar to calling `new` on an object; we can pass create a `Hash` of attributes and it will assign those for you. However, unlike `.new`, after calling `.create` that record is saved directly to the database.

```
User.create({ first_name: "justin",
               last_name: "herrick",
                   email: 'justin@theironyard.com' })
```
###3. `User.find`
If you already have the id of the object you are looking for, you can use that id to quickly and easily retrieve that record from the database. Let's say I know that my internal user id is `14`. If I want to retrieve my user information from the `users` table, I need only say `User.find(14)`. Pretty straight forward. It is worth nothing that if I type an ID that does not exist, I will get an error. This error:

```
> User.find(78704)
ActiveRecord::RecordNotFound: Couldn't find User with 'id'=78704
```
###4. `User.where`
`.where` is the powerhouse of Active Record query methods. I would venture to guess the majority of the queries in your Active Record based projects will come down to using `.where`. You can think of `.where` as a filter for your entire table. If you want to find every user who has the same first name as you, you can ask for that `User.where(first_name: 'justin')`. You don't have to stop at just one attribute, either. You can match against as many attributes as you want.

###5. `User.order`
The sorting of a set of records is normally pretty important. Sorting in SQL is fast -  really fast compared to Ruby - so it's worth sorting when you get the records from the database when you can. `User.order('last_name ASC')` will return all the records sorted by `last_name` ascending (as in: from a to z). `.order` is also chainable with `.where`. So I can look for all people with a certain name and also make sure they are ordered. `User.where(first_name: 'Finn').order('age DESC')`

###6. `User.count`
Knowing how many records we have is important. I need to know how many tweets I've made at all times. I'm glad that `.count` is there to help me out. `User.count` will return the number of rows in the `users` table. `.count` is also chainable with `.where` so you can get the count of any filtered set from the database table you are looking at.

###7. `User.average`
One of the more esoteric calculations you can run that Active Record provides is the `.average` method. Calling `User.average(:age)` will give me the average age of all the rows in my `users` table. `.average` is also chainable with `.where` so you can get the average of any filtered set from the database table you are looking at.


###8. `User.pluck`
If you only need a single field (column) from your database, just pluck it out. `User.pluck(:first_name)` will return all of the `first_names` from the `users` table. `.pluck` is also chainable with `.where` so you can get the field you want of any filtered set from the database table you are looking at.

###9. `User.limit`
Just like in SQL you can put a hard limit on the amount of records that get returned from any Active Record query. As expected `.limit` is also chainable with `.where` so you can get the right amount of records from any filtered set from the database table you are looking at.

###10. The rest
- `.offset`
- `.max`
- `.sum`
- `.not` _< Used with where >_ `User.where.not(age: 21)`
- `.take`
- `.first`
- `.joins`
- `.select`
- And more!

## Parting Thoughts
I hope this primer gives you some idea of the concepts, systems, and methods used in Active Record and can help you become productive quickly. That being said there is **no** replacement for reading the official guides. I have linked them below, and the first two are what the majority of the info in this primer is from. I *strongly* encourage you read them in their entirely sooner than later.

##Resources
- [The Active Record Intro Guide](http://guides.rubyonrails.org/active_record_basics.html)
- [The Active Record Query Guide](http://guides.rubyonrails.org/active_record_querying.html)
- [The Active Record Migrations Guide](http://edgeguides.rubyonrails.org/active_record_migrations.html)
- [Wikipedia: Active Record Pattern](http://en.wikipedia.org/wiki/Active_record_pattern)
