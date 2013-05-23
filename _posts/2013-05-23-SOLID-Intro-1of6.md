---
layout: post
title: SOLID Principles of Object-Oriented Design (Part 1 of 6)
description: ""
tags: [programming]
---
The SOLID principles of Object-Oriented design are guidelines to writing objects that can easily be extended and be maintained.  When I first learned about them, a couple of them were kind of hard for me to grasp at first.  The principles are:

* Single responsibility Principle
* Open/Closed Principle
* Liskov Substitution Principle
* Interface Segregation Principle
* Dependency Inversion Principle


I find it hard to illustrate these principles without going back to the "Interviewish" OO comparisons of cars and animals.  Though the next few posts, I will take a peice of code (It's a contrived example, but much similiar to code "in the wild") and apply these principles to create something that is more resiliant to change, and less error prone.  

Business Rules
-----------
This small application runs each morining.  It's purpose is to email a forecasted price of an item so that a user can make a budget decision.  The source of the prices is an HTTP endpoint.  The naive algorithm gets prices from the last 7 days and computes the average.  It then emails that value to the specified user

The Code (C#) 
----------
	using System;
	using System.Net.Mail;
	using System.Net.Http;
	 
	namespace SOLID_workflow
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            var beginDate = DateTime.Now.Subtract(new TimeSpan(7,0,0,0));
	            decimal total = 0;
	 
	            // get a total of the prices from the last 7 days
	            while (beginDate < DateTime.Now)
	            {
	                var response = new HttpClient()
	                	.GetAsync("http://www.null.com/" + beginDate.ToShortDateString()).Result;
	 
	                int price = int.Parse(response.Content.ReadAsStringAsync().Result);
	 
	                total += price;
	                beginDate = beginDate.Add(new TimeSpan(1, 0, 0, 0));
	            }
	 
	            // calculate the average from the total
	            var average = total / 7;
	 
	            var emailMessage = new MailMessage("from@null.com", "to@null.com")
	                                   {
	                                       Body = "The forcasted price is " + average.ToString()
	                                   };
	 
	            new SmtpClient("smtp.null.com").Send(emailMessage);
	 
	        }
	    }
	}
[Repo Link](http://github.com/tcshao/SOLID-RefactoringExample/blob/master/src/Initial/Program.cs)

This code has various issues, but for the purpose of the exercise, the issues we will tackle concern the following questions

* Can this code be tested, if so, how?
* If you need to change any of the steps in this process, how easy is it? what will be affected by these changes?
* How can you seperate the configuration from the application logic?

In the next post I will refactor this code according to the Single Responsibility Principle.

