---
layout: post
title: "Imperative vs Declarative Programming"
description: ""
tags: [programming]
---
I've always considered a key measure of code quality is is how easily it can be read. If it can be easily understood by looking at it, than it can easily be extended or refactored (or at least can be done with less fear).  

This is a key reason I am starting to gravitate towards declarative style and functional techniques. Using those, code reads more like the actual problem at hand rather than the steps-oriented approach produced by imperative programming.

[Imperative Programs](http://en.wikipedia.org/wiki/Imperative_programming) defines explicit sequences of commands the computer will perform to get the desired result.  In contrast, [Declarative Programs](http://en.wikipedia.org/wiki/Declarative_programming) describes what the program should perform.  One of the most common declarative languages is SQL.  When you run a SELECT statement, you do not think about the steps the computer is actually taking, you are telling it what you would like it to do.

I will show an example using the controversial FizzBuzz problem, here is an answer in imperative style (C#)

	for (int i = 1; i <= 100; i++)
	{
		string output = string.Empty;
		if (i % 3 == 0) output += "Fizz";
		if (i % 5 == 0) output += "Buzz";
		if (String.IsNullOrEmpty(output)) 
			output = i.ToString();
		Console.WriteLine(output);
	}

This is fairly straight-foward as C# is concerned. One thing I would point out though. If you handed this code to someone who wasn't quite sure what it did, it wouldn't be readily apparent. You would have to reason through each step of your code to determine what was going on. For example, the requirement where it prints "FizzBuzz" isn't clear from looking at the code.


Compare this with a functional approach. (I will show using F#)

First, we need to get a base algorithm to solve the problem:

    let fizzBuzz x =
        match x with
        | _ when (x % 15) = 0 -> "FizzBuzz"
        | _ when (x % 3) = 0 -> "Fizz"
        | _ when (x % 5) = 0 -> "Buzz"
        | _ -> x.ToString()

This is a function that takes in "x" (which is inferred as an integer) and returns a string. It uses pattern matching as a control structure. It matches the argument it is given with the options listed.  The part to the right of the -> is what the function will return.  The last case is what it will do if it doesn't find a match. 

I feel this is much easier to read. All of the cases can be determined upon visual inspection. (As an aside you can paste this function right into F# Interactive and test it ([Online Version Here](http://www.tryfsharp.org/Tutorials.aspx)). This process lets you test code as you build, facilitating iterative development)

All that is left to do is set up a sequence for how many numbers we want to go through:

    [1..100] |> Seq.iter (fun number -> printfn "%s" (fizzBuzz number))

The |> operator is the pipeline operator. It takes what's on the left and applies it to what is on the right. So, it takes in the numbers 1-100 as the source of the sequence, then iterates through the sequence and applies the function listed between the () to each item.

Side Note, it seems to be an unwritten rule that FizzBuzz is always processed from 1 to 100.