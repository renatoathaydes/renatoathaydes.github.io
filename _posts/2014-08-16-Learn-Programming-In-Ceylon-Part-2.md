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

## Custom types with classes

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
quickly become un-manageable and is the source of countless bugs in real software.

## Interfaces

Quite often, we come across concepts which have the same kind of properties, but differ only in the details of how they behave.
Continuing with our game, we may think of one example of this as being a dealer. Whatever the game we decide to create
may be, we will probably need a dealer. There are many things a dealer can do, and most of them do not depend on which
game is being played, such as dealing the cards, shuffling and perhaps comparing players' hands.

This is a good example of where we could use an `interface` to model a fairly abstract concept.

> An interface is an abstract definition of a particular concept which we want to model. In other words, it can be seen
  as a contract which may be satisfied by concrete or abstract classes to provide the actual behavior.

Before we define our first interface, we must formally define what a `method` is:

> A method is simply a shared function that is declared inside a class or interface. It is useful to make that distinction
  because methods can have access to internals of a class which are not available to other functions, so methods, unlike
  functions, can be seen as being an integral part of the class or interface in which they are defined.

Ok, now that we know the theory, let's give that a go:

{% highlight ceylon %}
// first, some definitions
alias Hand => {Card*};
alias Pack => {Card+};

interface Dealer {    
    shared formal Pack shuffle(Pack pack);
    
    shared formal Hand dealHand(Pack cards);
    
    shared formal Comparison compare(Hand hand1, Hand hand2);
    
    shared default {Hand*} deal(Pack cards, Integer playersCount) =>
        (1..playersCount).map((Integer i) => dealHand(cards));
        
}
{% endhighlight %}

> Notice that each method of an interface which does not have a default implementation must be declared `formal`.
  Default implementations can be provided and must be declared `default`.

`Dealer` is just a concept, it does not define how to do anything (notice how the formal methods have no body)
except for the trivial case of defining the `deal` method, as that can be easily done in terms of the formal method
`dealHand`, even if we do not yet have a concrete implementation of that method. The actual behavior of a `Dealer` must
be provided by implementation classes.

This is very useful as it allows us to write other parts of the game which do not depend upon the specific rules of our
game before we even know what our game rules are going to look like. And notice that there could even be more than one
game (and therefore different implementations of `Dealer`), but the parts of the game which will not be impacted by the
specifics of each game can still be written only once.

Notice that, because interfaces do not implement all the behavior that would be necessary to actually use an instance of
it, you cannot instantiate an interface directly!

The following results in a compiler error (so you can't even run this program):

{% highlight ceylon %}
// does not compile!
value dealer = Dealer();

// what would this return if this compiled?
value hand = dealer.dealHand(pack);
{% endhighlight %}

For this reason, before we can run our program, we will have to provide at least one implementation for `Dealer`.

### Benefiting from syntax-sugar for operators

Ceylon provides some *syntax sugar* to make code more readable through the use of certain interfaces.

For example, in Part 1, we saw how the `Integer`'s methods `plus`, `minus`, `divided`, `times` and
`remainder` are equivalent to using the operators `+`, `-`, `/`, `*` and `%` respectively.

So, instead of writing `2.plus(2) == 4` we can write `2 + 2 == 4`.

The only reason why that is possible is because `Integer` satisfies the interfaces `Summable` (which
defines `plus`, or `+`), `Numeric` (which defines `minus`, `divided` and `times`, or `-`, `/` and `*`) and `Integral`
(which defines `remainder`, or `%`).

Anyone could write a type that satisfies `Summable` to be able to use the `+` operator to sum two instances of that type.

As an example, we can make a Summable Iterable so that we can *add* two Iterables:
 
> Notice that in Ceylon, you cannot add two Iterables with the `+` operator. The reason for that is that doing that would
  likely break the contract of `Summable` that the `+` operation should be associative.
  For example, { 1, 2 } + { 3 } would have to be the same as { 3 } + { 1, 2 }. In the definition of `MyList` below, we
  get around that fact by adding the constraints that `MyList` contains only `Integer`s and is always sorted, so that 
  `MyList { 1, 2 } + MyList { 3 } == MyList { 3 } + MyList { 1, 2 }`.

{% highlight ceylon %}
// necessary renaming to avoid clash with Iterable.sort in MyList
import ceylon.language { doSort = sort }

"An always sorted Iterable containing Integers that can be summed."
class MyList({Integer*} integers) satisfies Summable<MyList> & Iterable<Integer> {
    
    {Integer*} _integers = doSort(integers);
    
    shared actual MyList plus(MyList other) =>
            MyList(this._integers.chain(other._integers));
    
    shared actual Iterator<Integer> iterator() => _integers.iterator();
    
}

MyList result = MyList { 1, 2 } + MyList { 3 };
assert(result.sequence == [1, 2, 3]);
{% endhighlight %}

The example above, incidentally, shows how a class can declare that it satisfies an interface, or, in this case, two
interfaces.

> A class may declare that it satisfies more than one interface by using the `&` symbol between each interface.
  What this symbol does is similar to what `|` does for union types, but results in an intersection type. Intersection
  types and union types are analogous to sets in set theory. If you look at a type as a set of properties and methods,
  then an union of the two types is the sum of all properties and methods of the two types. The intersection
  of two types is the set of properties and methods which are common to both types. That is why, in the example above,
  an instance of `MyList`, which satisfies types `Summable<MyList>` and `Iterable<Integer>` can be assigned to either type.

It also shows how you can import an element and rename it to avoid name clashes (if we didn't rename `sort` to `doSort`,
it would have clashed with `Iterable`'s own `sort` method inside the definition of `MyList`, causing a compiling error
because we are not allowed to use an inherited member in the class initializer).

The `in` operator maps to method `contains` of `Iterable`, so because `MyList` satisfies `Iterable`, we can use the `in`
operator to test if a `MyList` contains a certain element:

{% highlight ceylon %}
assert(1 in MyList { 1, 2, 3 });
assert(2 in MyList { 1, 2, 3 });
assert(3 in MyList { 1, 2, 3 });
{% endhighlight %}

> Notice that when a class satisfies an interface, it gets all of its default methods for free! That's why we can use the
  `in` operator with `MyList`, as shown above. We could also use many more methods and properties from `Iterable`:
  `first`, `last`, `find`, `sort`, `any`, `every`...

If we try to compare two instances of `MyList`, we will notice that the results will not be as expected:

{% highlight ceylon %}
// this assertion fails
assert(MyList { 1, 2 } == MyList { 1, 2 });
{% endhighlight %}

That's because of the way the `==` operator works. It maps to the `equals` method of the `Object` abstract class.

Before we explain that in more detail, we must understand what abstract classes are.

## Abstract classes

Finally, the last way in which we can create a type in Ceylon is by declaring an abstract class.

> An abstract class is similar to an interface, but besides having formal and default methods and properties, it can also
have a parameter list and hold state, like concrete classes. Unlike concrete classes, but like interfaces, abstract 
classes may not be instantiated.

Let's consider `MyList` again. It explicitly declares only two methods, `MyList plus(MyList other)` and
`Iterator<Integer> iterator()`. However, because it satisfies interface `Iterable<Integer`, it **inherits** all of its
methods, so although we can call a lot of methods on `MyList` instances:
{% highlight ceylon %}

{% endhighlight %}

This operator is equivalent to the `equals` method of the
`Object` abstract class. All custom types, by default, extend the `Basic` type, which in turn, extends `Object`.


TODO


# Objects and Enumerated types

In the previous section, we declared a lot of classes which do not have any property, they simply exist to enumerate possible
values of a suit or a rank.

There is a better way to enumerate values like this in Ceylon using objects.

> An object is an instance








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


