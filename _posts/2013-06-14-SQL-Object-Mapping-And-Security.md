---
layout: post
title: "SQL, Object Mapping, and Security"
description: ""
category: development
tags: [Database, Security]
---
While looking at [StackOverflow](http://www.stackoverflow.com/) for questions to answer.  I still run across a lot of people using inline SQL and kludging string variables into their statements. By that I mean something like this:

    query = "SELECT * FROM USERS WHERE UserId = " + userId 
	db.Execute(query)

I've written a lot of code like this.  In fact, I wrote code like this exclusively for years.  Patterns and practices behind database access has evolved a lot since then.  Now, when I see it, I feel like i've stepped into a time machine.  When considering the security aspects, it is dangerous.  Many libraries have been created to manage database connectivity and query logic in the past 10 years, and to effectively use those, I think it's important to understand why they exist.

The Motivating Example
-----
If you have any kind of application and you are doing any kind of logic based upon strings that are non-constants, you are going to have a bad time.  When you have a logic branch based on user input, that input needs to be verified that it is safe (sanitized) before doing anything with it.  Let's say somehow the contents of the variable `userId` became "0; DROP TABLE USERS".  This can happen because the variable is just blindly created from a user entered textbox or form request. the executed query is now:

	query = "SELECT * FROM USERS WHERE UserId = 0; DROP TABLE USERS"
	db.Execute(query)

Not good.  This is now a huge risk.  This is an example of [SQL Injection](http://en.wikipedia.org/wiki/SQL_injection), and it is still one of the main vulnerabilities discovered in web applications (but it's certainly not limited to the web).  The safe way to execute SQL statements is to parameterize them.  If unsafe information is passed into the parameterized SQL statement, the statement will not execute at all. Paramaterization looks different depending on the language, but it usually resembles something like this:

	query = "SELECT * FROM USERS WHERE UserId = ?"
	db.Execute(query, userId) 

The SQL statement is defined with placeholders, and before the query is run you assign values to those placeholders.  Depending on the language you are using, it can also check the types for you to make sure you are passing the correct form of data. This may address some of the security concerns, but this code still smells.  Database code is notoriously hard to test and just adding parameters doesn't make it any easier.  When you get data back theres no way to know if it's what you asked for or if that's what's in the data store.  We should strive to isolate our [responsibilites](http://autoincomplete.com/2013/05/29/SOLID-SRP-2-of-6/) within our objects.  The responsibilites in this case are displaying data and modifying/persisting that data.

Object-Relational Mapping
---
One approach to trying to ease this pain of using data from a relational database into the type system of your choice is to use an [Object-Relational Mapper](http://en.wikipedia.org/wiki/Object-relational_mapping).  (For more word buzzword bingo points, the technical term for this problem is [The Object-Relational Impedence Mismatch](http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)).  Some popular versions of Object-Relational Mappers (ORMs) are:

* Hibernate/NHibernate (Java/.NET)
* ADO .NET Entity Framework (.NET)
* ActiveRecord (Ruby on Rails)
* Core Data (Objective-C)

All of these have different ways of behaving.  But the common goal is to get your plain old objects into relational stores.  Using these frameworks, most of the SQL is abstracted from the developer.  

Here's a naive implementation for basic ORM functionality (reading in and mapping data) in pseudo-Ruby (illustrative code only!)

	class User
	  attr_accessor :id, :first_name, :last_name, :isAdmin
      
	  def initialize(id, first_name, last_name, isAdmin)
	    # setting of attributes here
	  end
      
	  def self.find(id)
	    row = db.get_first_row "SELECT id, first_name, last_name, isAdmin FROM Users WHERE id = ?", id
	    User.new( row['id'], row['first_name'], row['last_name'], row['isAdmin'] )  
	  end
      
      def self.create(first_name, last_name)
        db.execute("INSERT INTO Users (first_name, last_name) VALUES(? ?)", first_name, last_name)
        User.new(db.last_id, first_name, last_name, false)
      end
      
	  def update
	    db.execute("UPDATE Users Set first_name = ?, last_name = ?, isAdmin = ? WHERE id = ?",
	      @first_name, @last_name, @isAdmin, @id)
	  end
	end

The database operations are encapsulated within an object which represents a user. Database operations are much simpler now: 

	user = User.find(3)
	user.first_name = "new"
	user.update

The plumbing on the aforementioned frameworks is vastly more sophisticated then the snippet above.  They offer a host of other features including change tracking, batch updates, and the ability to navigate your data using the representative object model.  So you may be able to access some normalized data through dot operators. There is a performance penalty to using ORMs, but they can speed up development time.  They all come with their own learning curve. The decision to use one, and which one to use, is highly subjective and varies from person to person.

As far as .NET goes, I've seen some positive changes after ORMs have become more widely used.  I see a lot more use of just plain old objects to transmit and bind data.  Which is really helpful, because you can just create collections of your domain objects and test functionality without needing a database at all.  It is good practice to write to an abstraction of a database anyway.  The [Repository Pattern](http://martinfowler.com/eaaCatalog/repository.html) is an example of this. 

There be Dragons
---
ORMs can sometimes make a black box out of the database.  I can't stress enough how important it is to still be aware of what is actually being executed against the database.  You need to be mindful of what each of your objects expose and how/when they access the database for reasons of performance and data security.

In 2012, GitHub was [compromised](http://www.infoq.com/news/2012/03/GitHub-Compromised) by a mass assignment vulnerability.  This specific instance involved Ruby on Rails, but similar technologies are in place with other ORM-backed MVC frameworks, so the same thing is possible elsewhere.

A site can become vulnerable by having classes that blindly just accept any user input.  It's very similiar to the SQL injection, in that if you know the right characters to put in the right boxes, you can do some real harm.

If we use the User class above in a real page, it is susceptible to this. We would normally use a class like this in a create user page.  This page would have text inputs for first and last name ONLY.  The assumption is that is all the person who is viewing the webpage can modify.  There is a submit button on the page, and that submit sends a post to another URI (say /user/create) with this:

	RequestBody: user[first_name]=Bob&user[last_name]=Dobalina

On the endpoint where the body is sent, automatic wireup tries to match up the parameters given to the user model we created (In ASP.NET MVC this is called Model Binding) and calls the "create" method.  This is great because it's doing what we expect it to do, and it took minimal code because it combined the magic of our ORM with our MVC framework.  It will look like this in Rails:

    @user = User.create params[:user]

The problem manifests itself when I craft my own post request and send this instead:

	RequestBody: user[first_name]=David&user[last_name]=Lightman&user[isAdmin]=true

This will create that user and give them admin access.  Because that attribute was exposed on the model, I (as the attacker) just had to make a guess at what the name of the attribute was.  There are a couple solutions to this problem.  From an architecture side, It may be wiser to pull all the administrative tasks in another controller that isn't so tied to creating and listing users.  From the front end client point of view, you could just assign all the attributes manually.

I find issues like this interesting because they illustrate what can happen without an understanding/respect for your frameworks.  