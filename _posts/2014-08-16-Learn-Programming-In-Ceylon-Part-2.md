## Under construction!

Will update this soon! Currently this is not done at all!

## Introduction

In [Part 1](http://renatoathaydes.github.io/Learn-Programming-In-Ceylon-Part-1) of this series, we had a look at quite a
lot of programming concepts! We saw how to get started with Ceylon and its IDE, what types (`String`, `Boolean` etc) and values
(`"Hello"`, `true`) are, functions (`function add(Float x, Float y) => x * y`), lists (Sequences and Tuples: `[1, 2, 3]`,
Iterables: `{'a', 'b', 'c'}`), loops (`for` and `while`), conditionals (`if` `then` `else`), comprehensions
(`{ for (a in as) a.something }`), and finally, variadic parameters (`function takeManyStrings(String* strings) => ...`).

We also created a first, more or less complete program, the *Maths Helper*, that allows users to do some simple maths.

In this part of the series, we will start looking at some more advanced concepts, such as union types, switches,
custom types, Maps, Trees and polymorphism. These concepts will allow you to tackle problems with a high level of
complexity in a manageable manner.

To start with, let's have a look at one of the most innovative features of Ceylon: union types.

## Union types

You might remember the `askUserForNumber` function, from Part 1, which asked the user for a Number and returned either
the number the user entered or `null`, ie. no value, if the user did not enter a valid number:

{% highlight ceylon %}
Float? askUserForNumber(String question) { ... }
{% endhighlight %}

We already mentioned then that `Float?` is just short notation for the type `Float | Null`, which reads as `Float or Null`.

This is called an **union type**.

> An union type is a type formed by two or more different types. A value always has a single type, but a function may accept
or return values of different types. That's the main reason union types exist.

To try this out, you can use the following code (don't forget, this must be inside the `run` function):

{% highlight ceylon %}
Float? askUserForNumber(String question) {
	process.write(question);
	value userX = process.readLine();
	return parseFloat(userX);
}

value x = askUserForNumber("Enter a number (x): ");
print(x exists then "Thank you!" else "Sorry, that's not a number.");
{% endhighlight %}

> We show above how you can use `then` ... `else` directly in a single expression... you just need something that evaluates
to a Boolean before `then`, and an expression that evaluates to some value in case that's true and another in case that's not.

Any number of types can be part of an union. Here are some examples:

{% highlight ceylon %}
void show(String|Boolean|Integer input) {
    String type;
    if (is String input) {
        type = "String";
    } else if (is Boolean input) {
        type = "Boolean";
    } else {
        type = "Integer";
    }
    print("``type``: ``input``");
}

show("hello"); // prints 'String: hello'
show(true);    // prints 'Boolean: true'
show(400);     // prints 'Integer: 400'

// convert a String into an Integer, Float or Boolean, whatever possible,
// or just return the String itself if none was possible.
Integer|Float|Boolean|String convert(String string) {
    return parseInteger(string) 
    else parseFloat(string)
    else parseBoolean(string)
    else string;
}

assert(convert("true") is Boolean);
assert(convert("1") is Integer);
assert(convert("1.0") is Float);
assert(convert("Just String") is String);
{% endhighlight %}

These examples are a little bit artificial but should already show the possibilities union types bring to the table.
To truly understand how union types can be extremely useful, however, we must first learn how to define our own types.

> A particularly useful construct to deal with union types is the `switch`, which we will meet soon.

## Defining our own types

All types we have talked about so far (`String`, `Boolean`, `Float`, `Integer`, `Character`) are defined by Ceylon itself
and available to any Ceylon program. We can achieve a lot using only these types, but to model the real world, it is very
convenient to be able to define our own types so that it becomes more natural to reason about the problems we are trying
to solve.

In Ceylon, there are four different ways one can define a type:

* `alias`
* `class`
* `interface`
* `abstract class`

The easiest way to define a type is to just borrow an existing definition and call it our own! That's what `alias` does.

## Alias

Sometimes, we want to declare a type which is very simple, or similar to another existing type, but we still believe it
would be useful to create that type to make code easier to understand.

A common use-case for this is to keep people's names. A `String` might seem adequate enough for a simple name, but that
would allow empty names to exist. So we may want to use `[Character+]` instead. And perhaps a sequence of `[Character+]`
for the full name. The problem is that we would end up with types like this:

{% highlight ceylon %}
// parse a name, returning either a full name or null if the given input was empty
[[Character+]+]? parseName(String input) { ... }
{% endhighlight %}

But we can make code like this much more readable by defining some aliases, as in this example:

{% highlight ceylon %}
alias Name => [Character+];
alias FullName => [Name+];

// parse a name, returning either a full name or null if the given input was empty
FullName? parseName(String input) {
    function stringAsSequence(String s) => s.trimmed.sequence;
    value parts = [ for (part in input.split().map(stringAsSequence))
                    if (nonempty part) part ];
    if (nonempty parts) {
        return parts;
    } else {
        return null;
    }
}

FullName? myName = parseName("Renato Athaydes");
assert(is FullName myName, myName == ["Renato", "Athaydes"]);

FullName? noname = parseName("");
assert(noname is Null);

FullName? emptyName = parseName("    ");
assert(emptyName is Null);
{% endhighlight %}

Aliases can also be useful to make large union types nicer on the eye. For example, we could improve the declaration of
`convert` in the previous section by defining an alias for `Integer|Float|Boolean|String`:

{% highlight ceylon %}
alias Convertable => Integer|Float|Boolean|String;

Convertable convert(String string) {
    return parseInteger(string) 
    else parseFloat(string)
    else parseBoolean(string)
    else string;
}
{% endhighlight %}

This is especially useful when such kind of type appears in a lot of places, so you don't need to re-write (or read) the
large type name again and again.

> Aliases do not actually create new types, they simply 'rename' an existing type or a simple type expression.

# Custom types with classes

Imagine we want to model playing cards to create a cards game. With what we know so far, we would have to improvise and use
types like `String` and `Integer`, or some aliases, to represent the cards. That might actually work, but there's a much
nicer way to model things in Ceylon: to create a `class` for the object you want to model.

A simple class declaration may look like the following:

{% highlight ceylon %}
class Card(shared Integer rank, shared Integer suit) {}

value card = Card(5, 3);
assert(card.rank == 5);
assert(card.suit == 3);
{% endhighlight %}

The above declaration defines a class called `Card` which contains the fields `rank` and `suit`. Because the fields are
shared, they can be accessed as *properties* of a card instance.

Using `Integer` as the type of both rank and suit seems misguided. What would the Integer corresponding to *hearts*, or
*Queen*, be? We can do much better than that by defining more new types for these as well:

{% highlight ceylon %}
// suites
class Spades() {}
class Diamonds() {}
class Hearts() {}
class Clubs() {}

// ranks
class Ace() {}
class Jack() {}
class Queen() {}
class King() {}

alias Rank => Integer|Ace|Jack|Queen|King;
alias Suit => Spades|Diamonds|Hearts|Clubs;

// definition of a playing card
class Card(shared Rank rank, shared Suit suit) {}

// example of a card
Card aceOfSpades = Card(Ace(), Spades());
{% endhighlight %}

This is looking good, but there's a problem with our definition of `Rank`: it still allows any `Integer` to be a valid
rank, but as we know, in any game of cards, the rank it limited to just a few numbers, besides *Ace*, *Jack*, *Queen*
and *King*.

The first solution that might come to mind might be to modify the `Card` class declaration to check that the value provided
for the rank is valid in case it is an `Integer` (in the other cases, no check is needed, of course):

{% highlight ceylon %}
class Card(shared Rank rank, shared Suit suit) {
    if (is Integer rank) {
        assert(rank in 2..7);
    }
}
{% endhighlight %}

That is a valid solution, it makes it impossible for anyone to instantiate an invalid card. But it makes it possible to
*try* to instantiate an invalid card, which would result in an Error!

{% highlight ceylon %}
// this will throw an Error
value invalidCard = Card(300, Hearts());
print("Will never get here");
{% endhighlight %}

An arguably better solution would be to enumerate each valid value, just like we did for `Suit`:

{% highlight ceylon %}
// ranks
class Ace() {}
class Two() {}
class Three() {}
class Four() {}
class Five() {}
class Six() {}
class Seven() {}
class Jack() {}
class Queen() {}
class King() {}

alias Rank => Ace|Two|Three|Four|Five|Six|Seven|Jack|Queen|King;
{% endhighlight %}

Now it is just impossible to even try to instantiate an invalid card! The problem is that enumerating all values might not
be always possible. In that case, the previous solution might be the only viable alternative. But keep in mind that every
time you run into this problem, you should try to minimize the possibility of a run-time error being possible, as that can
get quickly un-manageable and is the source of countless bugs in real software.

# Methods

We have met quite a few *methods* already. A method is simply a function that belongs to a class.












Let's imagine we are trying to write a program to store information about cities of the world. We know that every city has
a name. Every city has a population. Every city is located in some country. And so on. Let's pretend we only care about these
features of cities. How can we model this in our programs?

One way would be to define a type for `City`. In Ceylon, we can define a type with a `class`:

{% highlight ceylon %}
class City(shared String name,
           shared Integer population,
           shared String country) {}

// let's create a City
value newYork = City("New York", 8_405_837, "USA");

// we can make it clearer what each value means by using the named-argument syntax
value berlin = City {
    name = "Berlin";
    population = 4_000_000;
    country = "Germany";
};

assert(newYork.population > 8M);
assert(berlin.country == "Germany");
{% endhighlight %}

## Polymorphism - extending existing types

Imagine that we have class `City` as defined above, but that now we decide that for certain important cities, we want to
store much more information. Things like *tallest building*, *best restaurant*, ...
things that just don't make sense for most cities except mega cities. Well, why not call our type `MegaCity`!? We can `extend`
our existing type to allow that to exist in our program:

{% highlight ceylon %}

{% endhighlight %}


## Enumerated types

In the previous example, `alias` was a really nice way to solve the problem. But it may not work in some occasions where
the types we want to alias are not so simple.

As an example, let's suppose that in our game, we need to be able show some information about the cards to help beginners.

This wouldn't work so well with the code shown above. If we try to print a card, say `aceOfSpades`, this is what we would get:

<code>helloCeylon.Card@2111fce</code>

Not nice! This is the default implementation of the `string` property of every user-defined type 

{% highlight ceylon %}

{% endhighlight %}


