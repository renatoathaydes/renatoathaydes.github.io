---
layout: post
title: Learn Programming in Ceylon - for beginners
---


## Introduction

Programming is one of the most rewarding activities there is. Once you master, you will be able to create all sorts of
things. Every single app, or program, you ever used on a computer was created mostly by computer programmers.

Every game you've ever played, every animation you've watched on the Internet, every website you've visited, required
someone (or many people) to write computer instructions that specify in minute detail how everything should look, behave,
change, react. It may look like an incredibly daunting task! But also one of the most fun tasks you can imagine!

These days, very few people work on the processor-instructions level, which is done in a kind of language called [Assembly]
(http://en.wikipedia.org/wiki/Assembly_language).

That's because to achieve anything much useful in Assembly would require enormous amounts of effort. Assembly was designed
to map directly to machine instructions. Basically, it is a machine-friendly language, not a human-friendly one.

Luckily, people have created many higher level languages overtime that made it easier for humans to write code that can
later be translated to Assembly by other programs, and then executed by computers. Programs that do the translation are called
**compilers** or **interpreters**.

One of the first such higher level languages was [Fortran](http://en.wikipedia.org/wiki/Fortran), created in the early 1950's.
Many other languages followed, a few noticeable examples being LISP (1958), COBOL (1959), C (1969) and Prolog (1972).

In recent years, there has been an explosion on the number of programming languages freely available to anyone willing to
learn them, some of which even "compile" to other languages! The number is almost certainly in the thousands, though it is
hard to know how many languages are actually used by more than just a few people.

The big, dominant languages, on the other hand, are currently just a few: Java, C++, C#, Python, JavaScript, besides LISP
(under many varieties) are probably the main ones.

Newer languages incorporate lessons learned in previous generations of languages to make programming easier and more efficient.
And a lot of lessons have been learned in the half-century people have been using higher level languages.

The newest generation of languages, which includes beautiful languages such as Haskell and D, are still coming up with innovations
to attempt to improve on their predecessors and find the perfect balance between simplicity and expressivity, speed and
conciseness.

To get the balance right is very difficult. But out of the newest wave of languages, one stands out for having an incredibly
powerful and disciplined, yet simple and accessible character: the [Ceylon programming language](http://ceylon-lang.org/).

## What is Ceylon

Ceylon is a direct [descendant of Java](http://ceylon-lang.org/documentation/1.0/faq/#what_is_ceylon),
which is a language that has been battle-tested in all sorts of conditions (in small and very large applications) and despite
its shortcomings, proven to be a great tool for developing almost any sort of application. Ceylon improves on Java's foundations
but also takes lessons from many other languages, packaging everything into a highly concise unit which really makes programming a
pleasurable experience.

Amongst the [many goals](http://ceylon-lang.org/blog/2012/01/10/goals/) of Ceylon, to achieve high readability and
predictability are the top ones. These are extremely important in programming. When you read some code, you should be able
to immediately know what it is trying to achieve, even if you are not familiar with all the details of the software.

Ceylon was designed from scratch to work in at least 2 different platforms:

* JVM (Java Virtual Machine) - the Java runtime.
* JavaScript engines - which includes almost every Internet browser.

This is really cool because it means you can write code that runs wherever you need it to run without changes!

## Getting started with Ceylon

To get started with Ceylon, the easiest way is to use the [Ceylon IDE](http://ceylon-lang.org/documentation/1.0/ide/install/)
(Integrated Development Environment, or in plain English, the software you use to easily write  and run Ceylon programs,
which lots of nice features like auto-completion - it suggests to you what kinds of things you can do while you write something -,
syntax highlighting and many other things).

## Installing the Ceylon IDE

The first step to get the Ceylon IDE is to download Eclipse (Standard), which is the general system the Ceylon IDE is based on. Go to [this page](http://www.eclipse.org/downloads/), choose the correct version for your system, then unzip the file you've downloaded into an easy to find directory, such as `/programming/eclipse/`.

Inside the `eclipse` folder, you will find the Eclipse executable. Double-click on it to start Eclipse (and add a shortcut to it on your desktop so you can find it easily next time!). Eclipse will suggest a workspace location the first time you open it. An Eclipse workspace is just a place for Eclipse to save your projects and other information it needs to work. Just accept the default (or choose any other location you want) and click OK.

Finally, you should see the Eclipse Welcome Page:

![Eclipse Welcome Page](https://www.sugarsync.com/pf/D6255314_638_6794803818?directDownload=true)

The overview and tutorials Eclipse comes with are targeted mostly at Java developers, but feel free to have a look at some of it if you have some time... but if you just want to get your hands dirty with Ceylon, just ignore those and keep reading!

To install the Ceylon plugin into Eclipse, follow these steps:

1. In the top menu bar, click on the `Help` menu item then select `Install new software...`.
2. In the `Work with` field, enter this URL:  `http://ceylon-lang.org/eclipse/updatesite/`
3. Click on the `Add...` button. A popup will come up, but you can just hit OK again (you don't need to give the site a name, just leave the Name field blank).
4. You should now select all options, as shown below, and click on OK.

![Install Ceylon in Eclipse](http://ceylon-lang.org/images/eclipseupdatesite.png)

5. Click on `Next` a few times, accept the license agreement and finally hit `Finish`!

> At this stage you might see a Warning message saying that this software contains unsigned content. That's fine, you've just downloaded the files from ceylon-lang.org, so there's nothing to fear, you can trust the good [Ceylon people](http://ceylon-lang.org/community/team/)! Click on OK and continue with the installation.

6. Click on OK to restart Eclipse.

If everything went ok, you should now see a nice elephant (the Ceylon mascot) next to a `Ceylon IDE` button. Click on this button and then on the `New Project` link. Give your project any name you want and follow the prompts.

And that's it! We're ready to start coding in Ceylon!

## Getting started with Ceylon

To get started with Ceylon, we must create a module, which is the top-level unit of distribution of any Ceylon library or application.

Click on `File`, select `New` and then `Ceylon Module`. Enter a module name, such as `helloCeylon`, and click on `Finish`.

> Notice that Ceylon Module names can only contain alpha-numeric characters (and `_`) and must not start with a number or capital-letter.
 The same limitation applies to function and variable names (soon you will see what these words mean).
 Also notice that every Ceylon module has a version! This is an awesome feature that Ceylon, unlike most languages,
 has built into the language itself and which makes Ceylon code highly modular, but that's a little bit of an advanced topic for now!

You should now have a source file called `run.ceylon` with the following contents:

{% highlight ceylon %}
"Run the module `helloCeylon`."
shared void run() {
    
}
{% endhighlight %}

The name of the file does not matter much, as long as it has the `.ceylon` extension.
The function name, which for the above code is `run`, also does not matter, but it's a Ceylon
convention to call the main function, ie. the function that starts your program, `run`.

A few words have now been introduced and deserve an explanation! First of all, a `function` is
just a piece of code which can be executed by either other functions or by the Ceylon runtime.
`function`s will be one of the most important blocks from which you create whole applications
(but not the only one, we will learn about many others later, such as `class`, `interface` etc).

As already mentioned, `run` is the name of the function... `shared` just means that this function
should be "visible" to functions that are located in different locations (`package`s as we will see later),
and `void` means that the function does not return anything. Other functions might return things, like a
`Number`, but let's keep it simple at first!

We can make Ceylon write something out (`print` in programmer's jargon) by changing the contents of `run.ceylon` to this:

{% highlight ceylon %}
"Run the module `helloCeylon`."
shared void run() {
    print("Hello World!");    
}
{% endhighlight %}

This is the famous Hello World program, which is almost always the first program you write when learning a programming language!
As you can see, it's pretty simple in Ceylon! We invoke a another function called `print`, which Ceylon provides to us, with
an argument `"Hello World!"`. We must use quotes so that Ceylon understands that we want it to treat as plain text, not some
kind of function call or anything else. We need to add a `;` at the end to indicate that our statement is complete (like a `.` in
the end of this sentence). You could write more than one statement in one line:

{% highlight ceylon %}
print("Hello"); print("Hi there"); print("I said hi!");
{% endhighlight %}

But this makes it harder to understand the code, so it is almost always better to have one statement only per line.

But now, let's get to the real fun and run our first program! 

Click on the green play button on the top of the screen. In the bottom pane called `Console` (should be near the bottom-right
 of the screen), you should see the message we asked to be printed, `Hello World!`.

You can change the text in the `String` (between quotes) to anything you like! Well, almost anything...

Ceylon allows you to include `value`s, in a String. But to do that, we must first understand what a value is.

## Types and values

Plain text like `"Hello World!"` is referred to as a `String` in the programming world (not only in Ceylon).
There are many other **types**, as these things are called, that you can use. For example:

* `Integer`: integers such as `-2, -1, 0, 1, 2, 3 ...`. Hexa-decimal numbers can also be represented: `#FF05DA`,
which is `16713178` in decimal.
* `Float`: floating-point numbers like `-2.0, -1.9, ..., 12_345.678 ...`
* `Boolean`: either the value `true` or `false`, one of the most useful types in any program!
* `Char`: characters (Unicode) like ` 'a', 'b', 'A', 'z', '\{#03C0}' ` (the last of which is the beloved constant `pi`!)
* `Null`: a type with only one possible value, `null` (meaning the absence of any value, something that comes in handy quite often)

We can also define our own types, but we will have to wait a little bit before we do that!

The following code illustrates how we can declare a value called `name` with contents `"John"`
(in other words, a `String` of value `"John"`):

{% highlight ceylon %}
value name = "John";
{% endhighlight %}

This is equivalent to writing `String name = "John"`, but Ceylon already knows that `"John"` must be a String, so you
 usually don't need to tell it that (sometimes you need to so that code in other parts of your program can also know
 the type of a certain value)!

`value` is a Ceylon keyword, and as such, you must not call something in your program `value`. There's quite a few reserved
[keywords](http://ceylon-lang.org/documentation/1.0/spec/html/lexical.html#identifiersandkeywords) in Ceylon
(which are necessary for it to be able to interpret your programs)... for example, a few of them are `void`,
`if`, `then`, `else`, `for`, `while` and `in`, which are instructions we will get back to shortly...

Now, instead of just printing a fixed String, let's make Ceylon print a String containing our value (paste this inside
 the `run` function):

{% highlight ceylon %}
value name = "John";
print("Hello ``name``!");
{% endhighlight %}

If you run this program, you should see the message `Hello John!`! That's because, by wrapping `name` between back-quotes ` `` `,
we made Ceylon interpret `name` as not plain text, but as an arbitrary Ceylon expression.

You may be asking yourself: but why not just write `print("Hello John!");`? and that would actually be exactly the same thing
for this basic example... but let's now change our program slightly to ask the person running the program to enter a value,
then print *Hello* to the name given:

{% highlight ceylon %}
value name = process.readLine();
print("Hello ``name``!");
{% endhighlight %}

> `process` is an `object` that is always available in Ceylon... to see what it can do, just type `process.` in the IDE
 and wait for the IDE to show you (hitting Ctrl+Space will bring a popup with suggestions from anywhere!).

Now, when you run this program, you will notice it will not print anything until you actually type something into the console
 window and hit Enter. But after you do that, you should see our greeting message printed with whatever value you entered.

This is really awesome, isn't it?

## A little maths fun

How about a program that does maths for us?

That's pretty simple, here's some examples:

{% highlight ceylon %}
value x = 2 + 2;
value y = 123 * 456;
print("x is ``x`` and y is ``y``");
{% endhighlight %}

Which prints:

```
x is 4 and y is 56088
```

You could even ask the user to enter numbers to use in the calculation:

{% highlight ceylon %}
process.write("Enter a number (x): ");
value userX = process.readLine();
value x = parseFloat(userX);
process.write("Enter a number (y): ");
value userY = process.readLine();
value y = parseFloat(userY);

if (exists x, exists y) {
    print("``x`` * ``y`` = ``x * y``");
} else {
    print("You must enter numbers!");
}
{% endhighlight %}

> Most of the syntax used above will be explained in the next section

The `write` method of `process` prints something to the console, just like `print`, but it does not create a new line
after printing (unlike `print`, which is really just short for `process.writeLine`).

If you run this program, then enter `3` for `x` and `5` for `y`, the program should print:

```
3.0 * 5.0 = 15.0
```

Neat! Now you will never have to do multiplication in your calculator again!

Actually you could have used many other mathematical operators: `+` (plus), `-` (minus), `/` (division),
`%` (remainder), `^` (exponential)... you can use Ceylon as a powerful calculator:

{% highlight ceylon %}
print(((2.5 * 3.0) ^ 2) + ((10_000 / 7)  % 37));
{% endhighlight %}

## Functions and the 'if' statement

In the previous section, we made use of another Ceylon function, `parseFloat`, to turn the `String` the user entered with
 the keyboard into a `Float` (ie. a floating-point number) so that we can actually multiply the numbers.

But notice that after we get values for `x` and `y`, there is an interesting new construct:

{% highlight ceylon %}
if (exists x, exists y) { ... } else { ... }
{% endhighlight %}

What this does is: first it checks if `x` exists... that's necessary because the user might have entered something that's
not a number for `x`, in which case the function `parseFloat` would not have been able to return a `Float` number at all!
But if `x` exists, it then checks if `y` also exists... only if that's the case, the code between the braces will be executed.

If either `x` or `y` did not exist (ie. could not be understood as a number), Ceylon will execute the code in the `else`
clause and print `You must enter numbers!`.

> You might be thinking, if the function `parseFloat` does not return a `Float` when we type something like `NOT A NUMBER!!!`
as a value for `x`, what does it return??? The answer is the object called `null`, of which you will hear a lot later! For
now, it's enough to say that `parseFloat` gives us either a `Float` or `null`, and because we cannot multiply something with
`null`, we must make sure that what we have is not `null` (which is what `exists` does), which means it must be a `Float`.

You can try using `if` with any *boolean* expression (something that evaluates to either `true` or `false`).

For example:

{% highlight ceylon %}
if (5 > 0) {
    print("Ceylon knows that 5 is greater than 0!");
}

value x = 10;

if (x <= 0) {
    print("This will be printed if x is less than, or equal to 0");
} else if (x <= 5) {
    print("This is printed if x is between 1 and 5");
} else {
    print("This is printed if x is greater than 5");
}

if (4 == 4) {
    print("The '==' operator checks for equality... 4 is equal to 4!");
}
{% endhighlight %}

One thing to notice is that functions can be values as well. As such, we can pass functions as arguments to other functions.

Check this out!

{% highlight ceylon %}
Float? askUserForNumber(String question) {
	process.write(question);
	value userX = process.readLine();
	return parseFloat(userX);
}

shared void run() {
	function multiply(Float a, Float b) => a * b;
	function add(Float a, Float b) => a + b;
	
	void printResult(Float(Float, Float) operation, Float x, Float y) {
		print("Result: ``operation(x, y)``");
	}
	
	value multiplication = 0;
	value addition = 1;
	value option = askUserForNumber("Enter ``multiplication`` for multiplication, ``addition`` for addition: ");
	value x = askUserForNumber("Enter x: ");
	value y = askUserForNumber("Enter y: ");
	
	assert(is Float option, is Float x, is Float y,
	       option == multiplication || option == addition);
	
	value operation = option == multiplication then multiply else add;
	printResult(operation, x, y);
}
{% endhighlight %}

First, we define a simple function that just asks the user for a number. As it is a top-level function (ie. not nested
inside another function), we must declare its return type (which is the same as `parseFloat`'s return type).

Now, inside the `run` function, we defined two new functions, `multiply` and `add`, using the short notation for functions
with the fat arrow `=>`.
This kind of function can only contain one expression. This is a very common style in Ceylon because usually functions
are broken up into many smaller functions until they become a simple expression. The long notation for functions is to
use `{ ... }` to wrap all statements the function should execute (like we did for the `askUserForNumber` and `run` functions).

We then defined a function called `printResult` which takes another function, called `operation`, with a type
`Float(Float, Float)`. You might have guessed that this type corresponds to any function that takes two `Float`s as arguments
and returns another `Float`. Not coincidentally, both `add` and `multiply` are examples of just such functions.

A function that returns a `String` and takes only one `Float` would look like this: `String(Float)`.

The program asks the user to enter either `0` for multiplication or `1` for addition, then asks for the values of `x` and `y`.

In the next step, it `assert`s that the user entered valid values!

`assert` can be used for 2 reasons:

* to verify a condition is true and in case it's not, throw an Error. When an Error is thrown, the program does not continue
execution (but there is a mechanism called `try`/`catch` which allows us to handle Errors such as this).
* to narrow the type of a value in conjunction with the `is` operator. After line `assert(is Float x);`, it is guaranteed
that `x` is a `Float`.

Finally, we call `printResult` using the appropriate operation.

Below are some sample runs of the above program:

{% highlight javascript %}
Enter 0 for multiplication, 1 for addition: 0
Enter x: 10
Enter y: 3
Result: 30.0

Enter 0 for multiplication, 1 for addition: 1
Enter x: 4
Enter y: 7
Result: 11.0

Enter 0 for multiplication, 1 for addition: 2
Enter x: 10
Enter y: 5
ceylon run: Assertion failed
  unviolated is Float option
  unviolated is Float x
  unviolated is Float y
  violated option == multiplication || option == addition
  ...
{% endhighlight %}

The error message in the last run states that the option assertion was violated because option was not multiplication nor addition.

It is considered good practice to keep each function you write (including the `run` function) at most a dozen or so lines long.

The resulting program may be a little longer than otherwise in the end, but writing code like this has several benefits:

* every function you write can be re-used in many places (make it `shared` if you want to maximize re-usability - at the cost
  of flexibility, as it gets harder and harder to change functions once they are used in more and more places).
* you only need to keep a small amount of information in your head when you are trying to understand what the code does
  because each function is small.
* each functionality is isolated from others and explicitly declares what is relevant information (its arguments) for it
  to do its job, and what is gives back (the return type).

If the name of the function is not enough for us to understand what it does, well, it probably has a bad name in the first place...
but we should also look at the type signature of the function, which is basically the types of the arguments (what we pass in)
and the type of the return value (what we get out). This is valuable information!

## Union types

Look at the type signature of `askUserForNumber`:

{% highlight ceylon %}
Float? askUserForNumber(String question) { ... }
{% endhighlight %}

We can see that we must provide a `String` with a question (likely a question to be shown to the user) and we get back a
`Float?`. Here we have a very interesting new concept to learn, which is called union types.

> An union type is a type formed by two or more different types. A value always has a single type, but a function may accept
or return values of different types. That's why union types exist.

`Float?` is synonym with `Float | Null`, which reads as *float or null*. This means that a value of this type must either
be a `Float` like `3.1415` or a `null`.

This makes sense, because when we ask the user to enter a number, we cannot be sure that the user is always going to give us
something that is a valid number.

To try this out, you can do something like this:

{% highlight ceylon %}
value x = askUserForNumber("Enter a number (x): ");
print(x exists then "Thank you!" else "Sorry, that's not a number.");
{% endhighlight %}

> We show above how you can use `then` ... `else` directly in a single expression... you just need something that evaluates
to a Boolean before `then`, and an expression that evaluates to some value in case that's true and another in case that's not.

Any number of types can be part of an union. As an example:

{% highlight ceylon %}
// we must import the 'type' function to find the type of a value
import ceylon.language.meta { type }

shared void run() {
	void show(String|Boolean|Integer input) {
		print("``input`` is of type ``type(input)``");
	}
	show("hello");
	show(true);
	show(400);
}
{% endhighlight %}

This is example is really artificial, but serves to demonstrate a few new concepts:

* the first line is a comment (ie. it is not treated as code) because it starts with `//`. Comments can also be wrapped
between `/*` and `*/` to span several lines.
* only a few functions and objects are available to Ceylon code without having to be explicitly imported. Most functions
from the Ceylon SDK (Software Development Kit) need to be explicitly imported, as shown above for the `type` function.
* You can define functions inside other functions.
* Union types can be formed by many different types.



For more of the basics, have a look at the [Ceylon basics](http://ceylon-lang.org/documentation/1.0/tour/basics/) in the Tour of Ceylon.


