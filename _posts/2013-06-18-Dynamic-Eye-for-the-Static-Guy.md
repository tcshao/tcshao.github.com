---
layout: post
title: "Dynamic Eye for the Static Guy"
description: ""
category: development
tags: [Ruby, C#]
---
I have been playing around with Ruby quite of bit in my spare time, I'm really starting to like it. I'm going to use this post to share with you what I find cool about it, and what i've learned from playing around with it.  If you are a developer in a staticly-typed language and was interested in looking over the fence, this post is for you.

Why bother learning another language?
---
The more languages you learn and the more experience you have, the cost of learning another language gets higher. Sherlock Holmes proposed that the brain worked like an attic.  It had a finite size, so what knowledge you decided to commit to memory had an opportunity cost associated with it.  You bring all your baggage and expectations along with the learning process, so it's hard to start "fresh".  On the flip side, learning a new language (and the conventions that go with it) can help you look at the other languages you use in a new light.  I touched on that a little bit in this [post](http://autoincomplete.com/2012/11/20/imperative-vs-declarative-programming/).

Why Ruby?
---
If you have a history of working with static languages, the dynamic switch is quite jarring.  You lose the warm-fuzzy autocompletion, and even more importantly, compile-time checking.  As a benefit, you get more concise code, and the need to write a lot less of it. There is a strong underpinning in [Ruby](http://en.wikipedia.org/wiki/Ruby_(programming_language) for productivity and developer happiness.  

As a C# developer, I have been finding a lot of things I like in C# in the past few framework versions (and subsequent ASP.NET updates).  I knew some of this was in Ruby, but it wasn't until I started playing with Ruby that I realized how much was there.  I am a big fan of [LINQ](http://msdn.microsoft.com/en-us/library/bb308959.aspx) in .NET, which is really an analog to the Ruby [Enumerable Module](http://ruby-doc.org/core-2.0/Enumerable.html).  I also started messing with Ruby on Rails using Michael Hartl's [Ruby on Rails Tutorial](http://ruby.railstutorial.org/).  Both Rails and ASP.NET MVC are implementations of a web framework using the MVC pattern.  Given that, there is a lot of similiarities between them.  ASP.NET uses a some dynamic typing features in it's Razor view engine.

I was seeing all these things in C# have obviously been influenced by Ruby (and relatives of), I was curious to learn more about it, and possibly look for a hint of where C# is heading.

How do I get started?
---
The aforementioned [Ruby on Rails Tutorial](http://ruby.railstutorial.org/) has a great [Up and Running](http://ruby.railstutorial.org/ruby-on-rails-tutorial-book#sec-up_and_running) section that gets you from 0 to Ruby fairly quickly (It includes directions for multiple platforms).  If you are interested in learning, I would recommend going through the whole online book. You will more about the Rails framework than Ruby itself, but It gives a lot info on code conventions, testing, and deployment which are universal concepts.  Be warned though, it is a bit like drinking from a firehouse, there is a LOT of information.  In the end, you end up with a little-more-than-trivial application. I really enjoy the style of that tutorial because it centers on actual workflow and best practicies, rather then 'here's how you create a webpage' with no real direction.  I'm also a fan of the [Sublime Text](http://www.sublimetext.com/) editor, which will give you a lot of color coding assistance and some word completion features.

Code example and comparison
---
Ruby is [interpreted](http://en.wikipedia.org/wiki/Interpreted_language) so there is no compile-time checking because there is no such thing as 'compile time'. To get back some of the errors the compiler could have caught for us, the solution is to write tests. I'm going to code a [Template Class](http://en.wikipedia.org/wiki/Template_method_pattern) in Ruby and C# to show what i'm talking about.

The template pattern codifies a sequence of events to happen.  in C# it enforces this through typing and inheritance:

    public abstract class Workflow
    {
        public abstract void YouShouldDoThisFirst();
        public abstract void ThenYouShouldDoThis();
        public abstract void ThisWillBeDoneLast();

        public void Process()
        {
            YouShouldDoThisFirst();
            ThenYouShouldDoThis();
            ThisWillBeDoneLast();
        }
    }

Now if i want to make a class that will follow this template, I have to create one.  The methods listed will becalled as specified in the `Process` method.

    public class Headhunter : Workflow
    {
        public override void YouShouldDoThisFirst()
        {
            Console.WriteLine("Lock the target");
        }

        public override void ThenYouShouldDoThis()
        {
            Console.WriteLine("Bait the line");
        }

        public override void ThisWillBeDoneLast()
        {
            Console.WriteLine("Spread the net");
        }
    }

Note: if i don't create those methods with those specific names, the program won't even compile, so it has no option to behave other than specified.

Here's the same thing in Ruby:

    class Workflow
	  def process
	    you_should_do_this_first
	    then_you_should_do_this
	    this_will_be_done_last
	  end
	end
	  
	class Headhunter < Workflow
	  def you_should_do_this_first
	    puts("Lock the target")
	  end
        
	  def then_you_should_do_this
	    puts("Bait the line")
	  end
	      
	  def this_will_be_done_last
	    puts("Spread the net")
	  end
	end

This is a little more compact.  There's a big difference in the way this code is run.  You could  erase the `this_will_be_done_last` function and the program would run fine UNTIL you call that method.  Then it will crash and burn. To protect ourselves from this, tests must be written.  Here is an [RSpec](http://rspec.info/) test for this class. (The rails tutorial goes through RSpec and using it)

	describe Headhunter do 
	 
	  before do
	    @headhunter = Headhunter.new()
	  end

	  subject { @headhunter }
	 
	  it { should respond_to(:you_should_do_this_first) }
	  it { should respond_to(:then_you_should_do_this) }
	  it { should respond_to(:this_will_be_done_last) }
	 
	  it "should call methods in order" do
	    @headhunter.should_receive(:you_should_do_this_first).ordered
	    @headhunter.should_receive(:then_you_should_do_this).ordered
	    @headhunter.should_receive(:this_will_be_done_last).ordered

	    @headhunter.process
	  end
	 
	end

The `before` and `subject` tell the rest of the tests what to do and what object we will have under test.  It then tests that each of the methods are actually present (`should respond_to`) and called in order (`should_recieve`) indicates that this method is called when `process` is called).  Note that now you have something that describes how the class actually behaves.

We definitely made up for all that lost typing.  However, the argument could be made that tests should be written for the C# anyway.  Although there's not much to test right now that won't pass Unit Test #0 (The compiler).  I know dynamic languages can be considered more error-prone due to that missing compiler step, which is why testing is even more important.

In later posts, I will work through some more patterns and show how you the change to dynamic really cuts down on lines of code you have to write.