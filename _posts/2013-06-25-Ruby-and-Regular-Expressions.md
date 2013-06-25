---
layout: post
title: "Ruby and Regular Expressions"
description: ""
category: development
tags: [Ruby, Regular Expressions]
---
I'll start this with a confession.  I never bothered to learn Regular Expressions.  The most effective tool in the programmer's toolbox for string searching and manipulation, yeah, I didn't want any part of it.  Granted, I never really had a text project come across my desk that involved an excessive amount of string parsing.  I saw the Q-Bert-ish syntax and was put off by it.  During my experiments [learning Ruby](http://autoincomplete.com/2013/06/18/Dynamic-Eye-for-the-Static-Guy/), I've discovered how straight foward Regular Expressions can be if you have a good knowledge of the building blocks and not being intimidated by it.  I will show The Simplest Thing(TM) to do as a gentle introduction.

Regular Expressions in Ruby
----
A Regular Expression is a set of characters used to identify an instance within a string.  In Ruby a regular expression is noted just by enclosing text in a `/` character.

    this_is_a_regex = /regex/
    this_is_a_regex.is_a?(Regexp) # returns true

There are a couple operators that are used in Ruby specifically for Regular Expressions.  `=~` Will return the index of the string where the pattern is found, `nil` if nothing is fund.  `!~` will return `false` if the string matches, and `true` otherwise.

    this_is_a_regex =~ "This string contains regex" # evals to 21, as thats 
                                                    # where the match starts
    this_is_a_regex !~ "This is a string" # evals to true

That's pretty much all there is to testing a string for an expression in Ruby.  Now for the expressions themselves.

Basic Matching
----

The `.`(period) represents a wildcard for any character, therefore:

    any_single_character = /./
    any_single_character =~ "hello" # returns 0

    two_adjacent_characters = /../
    two_adjacent_characters =~ "a" # will be nil

The `\` (backslash) is the escape character, if you actually meant to find the `.`:

    the_dot_character = /\./
    the_dot_character =~ "autoincomplete.com" # evals to 14

Combining what we know thus far, we can combine all these things,

    g__gle_com = /g..gle\.com/
    g__gle_com =~ "google.com" # this will eval to 0, as it matches
    g__gle_com =~ "gaagle.com" # so will this

In addition to finding individual characters, you can specifiy a set of characters, and will match whenever one of those characters are found.  The syntax of this is to encluse the set in `[]` (brackets).  For example `[abc]` will match on `a`,`b`, OR `c`

    even_number_set = [2468]
    even_number_set =~ "1024" # will be 2, (the first occurance)

You can use the `-` in the set to define a range

    three_to_seven = /[3-7]/
    three_to_seven =~ "1024" #evals to 3

Now with this information we can start checking for some standard formatting data, let's say we are looking for a 12H time value:

    timeformat = /[01][0123456789]:[012345][0123456789] [aApP][mM]/
    timeformat =~ "03:49 pM" # evals to 0

This is a bit longer then it needs to be, there are ways we can shorten this up

Abbreviations in Regular Expressions
---
Here are some common abbreviations used

* \d - `[0123456789]`
* \s - any whitespace characters (spaces, new line, tabs)
* \w - numbers and letters `[0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ]`

We can abbreviate the above to this:

    timeformat = /\d\d:\d\d [aApP][mM]/
    timeformat =~ "03:49 pM" # evals to 0 (this is mostly the same, 
                             # its shorter but it will also take 99:99)

Now we can start looking for actual data, lets say we are looking for a zip code, state combination:

    state_zip = /\w\w, \d\d\d\d\d/
    state_zip =~ "IL, 60666" # this is a match, two letters, a comma, space, followed by 5 numbers

More Basic Symbols
---
You can use a combination of the `|` (pipe) symbol and `(``)` (parenthesis) to provide OR functionality

    l_or_t = /(l|t)/
    l_or_t =~ "pat" # this will eval to 2
    l_or_t =~ "pal" # this will also eval to 2

There is also the `*` (asterisk) character, which means "matches the previous element zero or more times", so both of these will match

    number_1__ = /#1\d*/
    number_1__ =~ "You are #1" # evals to 8
    number_1__ =~ "You are #1000" # this also evals to 8

    number_1_place = /#1\d* place/
    number_1_place =~ "You are #123432 place" # this is also 8

Final Thoughts
---
This is just scratching the surface of Regular Expressions.  As another resource, I would recommend [RegexOne](http://regexone.com/).  It has interactive tutorials that walk you through finding an expression to find (and exclude) strings. You can also use [Rubular](http://rubular.com/) as a sandbox to test expressions and strings to see if they match.

I guess i can't have a post of simple stuff without an arbitrary code example doing something impractical. The `scan` method on a `String` object pulls out each element that matches a given pattern on a string and returns an array, and can be passed a code block.  So we can do this to count up all the values of the characters that are numbers in a string:

    string_with_numbers = '3 bears 52 cards 5 senses 9 times 0 crates 3 sacks'
    puts with_numbers.scan(/\s*\d*\s/).inject(0) { |sum, curr| sum += curr.to_i }

This evals to `72`.  Note: This pattern is looking for (`\d*`) which will take any group of numbers that are surrounded by whitespace.  It doesn't take in account `.` or `,`.