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

I've written a lot of code like this.  In fact, I wrote code like this exclusively for years.  Patterns and practices behind database access has evolved a lot since then.  It's hard for me to look at other people's code like this and not say something about it.  When considering the security aspects, it is dangerous.  There are a lot of libraries created to address these problems, and to effectively use those, I think it's important to understand why they exist.

Do Not Trust User Input
=====
If you have any kind of application and you are doing any kind of logic based upon strings that are non-constants, you are going to have a bad time.  If you have a logic branch based on user input, that input needs to be sanitized before doing anything with it.  I'll explain what i mean by sanitized by showing an example of what can happen.  Let's say somehow the contents of the variable become "0; DROP TABLE USERS", the executed query is now:

	query = "SELECT * FROM USERS WHERE UserId = 0; DROP TABLE USERS"
	db.Execute(query)

Not good.  This is now a data security risk.  This is an example of [SQL Injection](http://en.wikipedia.org/wiki/SQL_injection), and it is still one of the main vulnerabilities discovered in web apps.  The safe way to execute SQL statements is to parameterize them.  When using parameters, if unsafe information is passed into it, the statement will not execute at all. Paramaterization looks different depending on the language, but it usually resembles something like this:

	query = "SELECT * FROM USERS WHERE UserId = ?"
	db.Execute(query, 1) # userID would be 1 in this case

This may address some of the security concerns, but this code still smells.  Database code is notoriously hard to test and this isn't doing us any favors.  If you get data back theres no way to know if it's correct, or really, if it is what you even asked for without instrumentation on the database side.  We should strive to isolate our [responsibilites](http://autoincomplete.com/2013/05/29/SOLID-SRP-2-of-6/) within our objects.  The responsibilies in this case are displaying data and modifying/persisting that data.

Object-Relational Mapping
---
One approach to trying to ease this pain of using data from a relational database into the type system of your choice is to use an [Object-Relational Mapper](http://en.wikipedia.org/wiki/Object-relational_mapping).  (For more word buzzword bingo points, the technical term for this problem is [The Object-Relational Impedence Mismatch](http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)).  Some popular versions of Object-Relational Mappers (ORMs) are:

* Hibernate/NHibernate (Java/.NET)
* ADO .NET Entity Framework (.NET)
* ActiveRecord (Ruby on Rails)
* Core Data (Objective-C)

All of these have different ways of behaving.  But the common goal is to get your plain old objects into relational stores.  Using these frameworks, most of the SQL is abstracted from the developer.  

Here's a naive implementation for base ORM functionality (reading in and mapping data) in pseudo-Ruby (illustrative code only!)

	class User
	  attr_accessor :id, :first_name, :last_name, :isAdmin
      
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

The database operations are encapsulated within an object which represent a user. Database operations are much simpler now: 

	user = User.find(3)
	user.first_name = "new"
	user.update

The plumbing on the aforementioned frameworks is vastly more sophisticated then the snippet above.  They offer a host of other features including change tracking, batch updates, and the ability to navigate your data using the representative object model.  There is usually a bit of a performance penalty to using ORMS, but they can speed up development time.  They do come with their own learning curve. The decision to use one, and which one to use, can get as heated as a religious discussion.  

As far as .NET goes, I've seen some positive changes after ORMs have become more widely used.  I see a lot more use of just plain old objects to transmit and bind data.  Which is really helpful, because you can just create collections of your domain objects and test functionality without needing a database at all.  It is good practice to wait till the last possible second before you even need to concern yourself with database implementation info.  The best code is the code that doesn't need to be written, so don't write it until you need it.

Pitfalls with Database Abstraction
---
ORMs can sometimes make a black box out of the database.  I can't stress enough how important it is to still be aware of what is actually being executed against the database.  You need to be mindful of what each of your objects expose and how/when they access the database for reasons of performance and data security.

Last year, GitHub was [compromised](http://www.infoq.com/news/2012/03/GitHub-Compromised) by a Mass Assignment vulnerability.  This specific instance involved Ruby on Rails, but similar technologies are in place with other ORM-backed MVC frameworks, so I wouldn't say the "failure" is Rails specific (although the wording might be).

A site can become vulnerable by having classes that blindly just accept any user input.  It's very similiar to the SQL injection discussed earlier, in that if you know the right characters to put in the right boxes, you can do some real harm.

If we use are User class above in a real page, it is susceptible to this. We would normally use a class like this in a create user page.  This page would have text inputs for first and last name ONLY.  The assumption is that is all the person who is viewing the webpage can modify.  There is a submit button on the page, and that submit sends a post to another URI (say /user/create) with this:

>	RequestBody: first_name=Bob&last_name=Dobalina

On the URI that boby is sent to there is some automatic wireup that tries to match up the parameters given to the user model  we created(In ASP.NET MVC this is called Model Binding) and calls the "create" method.  This is great because it's doing what we expect it to do, and it took minimal code because it combined the magic of our ORM with our MVC framework.  The problem manifests itself when I craft my own post request and send this instead:

>	RequestBody: first_name=david&last_name=lightman&isAdmin=true

This will create that user and give them admin access.  Because that attribute was exposed on the model, I just had to make a guess at what it was.  The root cause of this mistake was exposing that "isAdmin" property externally.  Removing isAdmin from the attr_accessor list will fix the problem.
