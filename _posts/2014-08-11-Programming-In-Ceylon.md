---
layout: post
title: Programming in Ceylon - for beginners
---

Ceylon is a programming language that was designed from scratch to work in at least 2 different platforms:

* JVM (Java Virtual Machine) - the Java runtime
* JavaScript engines - which includes Internet browsers

This is really cool because it means you can write code that runs wherever you need it to without changes!

To get started with Ceylon, the easiest way is to use the [Ceylon IDE](http://ceylon-lang.org/documentation/1.0/ide/install/) (Integrated Development Environment, or in plain English, the software you use to easily write  and run Ceylon programs, which lots of nice features like auto-completion - it suggests to you what kinds of things you can do while you write something -, syntax highlighting and many other things).

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

> Notice that Ceylon Module names can only contain alpha-numeric characters (and `_`) and must not start with a number or capital-letter. Also notice that every Ceylon module has a version! This is an awesome feature that Ceylon, unlike most languages, has built into the language itself and which makes Ceylon code highly modular, but that's a little bit of an advanced topic for now!

You should now have a source file called `run.ceylon` with the following contents:

```ceylon
"Run the module `helloCeylon`."
shared void run() {
    
}
```

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

```ceylon
"Run the module `helloCeylon`."
shared void run() {
    print("Hello World!");    
}
```

This is the famous Hello World program, which is almost always the first program you write when learning a programming language!
As you can see, it's pretty simple in Ceylon! We invoke a another function called `print`, which Ceylon provides to us, with
an argument `"Hello World!"`. We must use quotes so that Ceylon understands that we want it to treat as plain text, not some
kind of function call or anything else. We need to add a `;` at the end to indicate that our statement is complete (like a `.` in
the end of this sentence).

Plain text like `"Hello World!"` is referred to as a `String` in the programming world (not only in Ceylon).

But now, let's get to the real fun and run our first program! 

Click on the green play button on the top of the screen. In the bottom pane called `Console` (should be near the bottom-right
 of the screen), you should see the message we asked to be printed, `Hello World!`.

You can change the text in the String (between quotes) to anything you like! Well, almost anything...

Ceylon allows you to include variables, or `value`s, in a String. To do that, we must first understand what a variable is.

The following code illustrates how we can declare a variable called `name` with contents `"John"`
(in other words, a `String` of value `"John"`):

```ceylon
value name = "John";
```

This is equivalent to writing `String name = "John"`, but Ceylon already knows that `"John"` must be a String, so you don't
need to tell it that!

`value` is a Ceylon keyword, and as such, you must not call your variables `value`. Other Ceylon keywords are `void`,
`for`, `while` and `in`, which are instructions we will get back to shortly...

Now, instead of just printing a fixed String, let's make Ceylon print a String containing our value (paste this inside
 the `run` function):

```ceylon
value name = "John";
print("Hello ``name``!");
```

If you run this program, you should see the message `Hello John!`! That's because, by wrapping `name` between back-quotes ` `` `,
we made Ceylon interpret `name` as not plain text, but as an arbitrary Ceylon expression.

You may be asking yourself: but why not just write `print("Hello John!");`? and that would actually be exactly the same thing
for this basic example... but let's now change our program slightly to ask the person running the program to enter a value,
then print *Hello* to the name given:

```ceylon
value name = process.readLine();
print("Hello ``name``!");
```

> `process` is an `object` that is always available in Ceylon... to see what it can do, just type `process.` in the IDE
> and wait for the IDE to show you (hitting Ctrl+Space will bring a popup with suggestions from anywhere!).

Now, when you run this program, you will notice it will not print anything until you actually type something into the console
 window and hit Enter. But after you do that, you should see our greeting message printed with whatever value you entered.

This is really awesome, isn't it?

How about a program that does maths for us?

That's pretty simple, here's some examples:

```ceylon
value x = 2 + 2;
value y = 123 * 456;
print("x is ``x`` and y is ``y``");
```

Which prints:

```
x is 4 and y is 56088
```

You could even ask the user to enter numbers to use in the calculation:

```ceylon
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
```

The `write` method of `process` prints something to the console, just like `print`, but it does not create a new line
after printing (unlike `print`, which is really just short for `process.writeLine`).

If you run this program, then enter 3 for x and 5 for y, the program should print:

```
3.0 * 5.0 = 15.0
```

Neat! Now you will never have to do multiplication in your head again!

We made use of another Ceylon function, `parseFloat`, to turn the `String` the user entered into a `Float`
(ie. a floating-point number) so that we can actually multiply the numbers.

But notice that after we get values for `x` and `y`, there is a an interesting new construct:

```ceylon
if (exists x, exists y) { ... } else { ... }
```

What this does is, first it checks if `x` exists... that's necessary because the user might have entered something that's
not a number for `x`, in which case the function `parseFloat` would not have been able to return a `Float` number at all!
But if `x` exists, it then checks if `y` also exists... only if that's the case, the code between the braces will be executed.

If either `x` or `y` did not exist (ie. could not be understood as a number), Ceylon will execute the code in the `else`
clause and print `You must enter numbers!`.

> You might be thinking, if the function `parseFloat` does not return a `Float` when we type something like `NOT A NUMBER!!!`
as a value for `x`, what does it return??? The answer is the object called `null`, of which you will hear a lot later! For
now, it's enough to say that `parseFloat` gives us either a `Float` or `null`, and because we cannot multiply something with
`null`, we must make sure that what we have is not `null` (which is what `exists` does), which means it must be a `Float`.

## Calling your own functions

We have now used a few Ceylon built-in functions: `print` and `parseFloat`, for example. We can also define our own functions
and call them just like we call those functions.

This is very useful to make our code more understandable. It is considered good practice to keep each function you write
(including the `run` function) at most a dozen or so lines long.

As an example, we could take the previous example and, instead of doing everything in one single function, break it apart
into a few functions, as shown below:

```ceylon
shared void run() {
	value x = askUserForNumber("Enter a number (x): ");
	value y = askUserForNumber("Enter a number (y): ");

	printMultiplicationResult(x, y);
}

Float? askUserForNumber(String question) {
	process.write(question);
	value userX = process.readLine();
	return parseFloat(userX);
}

void printMultiplicationResult(Float? x, Float? y) {
	if (exists x, exists y) {
		print("``x`` * ``y`` = ``x * y``");
	} else {
		print("You must enter numbers!");
	}
}
```

This program may be a little longer than our original one, but writing code like this has several benefits:

* every function you write can be re-used in many places (make it `shared` if you want to maximize re-usability).
* you only need to keep a small amount of information in your head when you are trying to understand what the code does
  because each function is small
* each functionality is isolated from others and explicitly declares what is relevant information (its arguments) for it
  to do its job, and what is gives back (the return type).

Let's analyse the above program.

We start with the `run` function. What does it do? Have a look at it...

Well, it asks the user for a number `x`, then it asks the user for a number `y`, then it prints the result of the multiplication.

Notice how describing it in English reads almost exactly as just reading the code itself. That's how you know you wrote good
code! You only need to look at the implementation of `askUserForNumber` if that's the part of the program you are currently
interested in.

If the name of the function is not enough for us to understand what it does, well, it probably has a bad name in the first place...
but we should also look at the type signature of the function, which is basically the types of the arguments (what we pass in)
and the type of the return value (what we get out). This is valuable information!

Look at the type signature of `askUserForNumber`:

```ceylon
Float? askUserForNumber(String question) { ... }
```

We can see that we must provide a `String` with a question (likely a question to be shown to the user) and we get back a
`Float?`. Here we have a very interesting new concept to learn, which is called union types.

> An union type is a type formed by two or more different types. A value always has a single type, but a function may accept
or return values of different types. That's why union types exist.

`Float?` is synonym with `Float | Null`, which reads as *float or null*. This means that a value of this type must either
be a `Float` (`0.0, 1.0, 23.555, -34.1`) or a `null` (the only value of the `Null` type).

This makes sense, because when we ask the user to enter a number, we cannot be sure that the user is always going to give us
something that is a valid number.



For more of the basics, have a look at the [Ceylon basics](http://ceylon-lang.org/documentation/1.0/tour/basics/) in the Tour of Ceylon.


