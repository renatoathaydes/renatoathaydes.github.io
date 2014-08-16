# Learn Programming In Ceylon - Part 2

# Under construction!

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
* Union types can be formed by many different types.

Once we learn to define our own types, union types will become much more interesting. That's the topic of the next section!


## Defining our own types

All types we have talked about so far (`String`, `Boolean`, `Float`, `Integer`, `Character`) are defined by Ceylon itself
and available to any Ceylon programs. We can achieve a lot using only these types, but to model the real world, it is very
convenient to be able to define our own types so that it becomes more natural to reason about our the problems we are trying
to solve.

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


## Using aliases

Sometimes, we want to declare a type which is very simple, or similar to another existing type, but we still believe it is
useful to make that type to make code easier to understand.

For example, in the above example, country is represented as just a `String`. That might make sense if we do not really care
about countries to solve our problem. But we might still want to create a type for `Country` for clarity. We can create an
`alias` as shown below:

{% highlight ceylon %}
alias Country => String;

// and now we can update the definition of City
class City(shared String name,
           shared Integer population,
           shared Country country) {}
{% endhighlight %}

Aliases do not create new types, they simply 'rename' an existing type.

However, sometimes we may face situations where we need a union of several types. In these occasions, alias can be a life-saver!

Imagine we want to model a game of cards to create a game. A simple way would be like this:

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

// definition of a playing card
class Card(shared Integer|Ace|Jack|Queen|King rank,
	       shared Spades|Diamonds|Hearts|Clubs suit) {}

// example of a card
Card aceOfSpades = Card(Ace(), Spades());
{% endhighlight %}

We could make the definition of `Card` much nicer by using aliases:

{% highlight ceylon %}

alias Suit => Spades|Diamonds|Hearts|Club;
alias Rank => Integer|Ace|Jack|Queen|King;

// definition of a playing card
class Card(shared Rank rank,
	       shared Suit suit) {}

// cards can still be created in the exact same way
Card aceOfSpades = Card(Ace(), Spades());
Card twoOfHearts = Card(2, Hearts());
Card queenOfClubs = Card(Queen(), Clubs());
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


For more of the basics, have a look at the [Ceylon basics](http://ceylon-lang.org/documentation/1.0/tour/basics/) in the Tour of Ceylon.


