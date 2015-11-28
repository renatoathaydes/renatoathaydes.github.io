# Ceylon as a powerful data modelling tool

Ceylon has a really powerful type system that allows you to model your code's data and behaviour in a
really advanced way.

For example, you can express that your method takes a non-empty stream of Strings:

```ceylon
void take({String+} s) {}
```

Which you can call like this:

```ceylon
take { "a", "b", "c" };
```

But not like this:

```ceylon
// does not compile!
take {}
```

This is extremely useful!

Here's another example of the power of Ceylon's type system.

Imagine a function `assess` that takes a couple of variables,
say a person's weight and height ( of types `Float` and `Integer`, respectively)
and a predicate function whose parameters are also of type `Float` and `Integer` and that
returns a `Boolean`... examples of such a predicate function may be `overweight`
or `inRiskGroup` (these should probably return a probability or some other graded assessment,
but let's try to keep it simple for now).

Here's the `assess` function written in Ceylon:

```ceylon
Boolean assess([Integer, Float] weightHeight, 
               Boolean(Integer, Float) predicate)
                => predicate(*weightHeight);
```

To avoid making mistakes with measurement units, let's give the types some more specific names:

```ceylon
alias Kilogram => Integer;
alias Meter => Float;

Boolean assess([Kilogram, Meter] weightHeight, 
               Boolean(Kilogram, Meter) predicate)
                => predicate(*weightHeight);
```

We could use this function like this:

```ceylon
function bmi(Kilogram weight, Meter height)
        => weight.float / (height ^ 2.0);

function overweight(Kilogram weight, Meter height)
        => bmi(weight, height) > 25.0;

value needToDiet = assess([82, 1.74], overweight);
```

Well, that's pretty cool already... a function can take any other function as an argument...
you can *spread* the elements of a Tuple (using the `*` spread operator) to call a function which
takes those elements... nice... but we can go even further in Ceylon!

We could make the `assess` function return whatever type the `predicate` function happens to return
(as the function may not be a predicate anymore, we also need to rename it to something more general,
like `fun`):

```ceylon
Result assess<Result>([Integer, Float] weightHeight, 
                      Result(Integer, Float) fun)
                        => fun(*weightHeight);
```

Easy enough with the use of generics. But why stop here? The `assess` function does not really care much
about the type of `fun`'s arguments as long as the types are the same as those of the `weightHeight` Tuple.
So let's remove that unnecessary type restriction (and rename `weightHeight` to a very generic `variables`):

```ceylon
Result assess<Result, Args>(
        Args variables, 
        Callable<Result, Args> fun)
            given Args satisfies Anything[]
            => fun(*variables);
```

Now we can use `assess` with type-safety over functions of absolutely any type:

```ceylon
// some example functions
function hi() => null;
function hello(String name) => print("Hello ``name``");
function bye(String* names) => names.collect(
    (name) => print("Hello ``name``"));

// assess will take any function, as long as the arguments types match
assess([], hi);
assess(["John"], hello);
assess([], bye);
assess(["Mary", "Adam"], bye);
```

Great! But I'm sure many of you are asking yourselves what's the point of writing a function that takes another
function and some arguments just to apply that function to the arguments... why not just apply the function directly?

And you'd be right for this toy example... but the thing is, using the same principles we have used so far, we are
able to create extremely powerful systems that can be extended at will. Don't believe me? Read on!

## Designing a real system just to use the cool stuff

Suppose that our goal is to create a web application to present health reports about a population.
Starting from the viewpoint of what we want users to see, the first thing we need to worry about is, basically,
how the data will be presented.

The obvious choice is to present the data in HTML so that users can see it right in their browsers, so let's start by
implementing a renderer that does that.

We also need the concrete representation of the data, of course, which will need to be translated into whatever the
renderer takes.

> The renderer could of course *know* about the actual implementation of the data, but if we did that our code
  would become much more brittle, interconnected and resistant to change!

Once we know how to represent the data and present it to the user, we must also implement some kind of search
functionality to allow doctors and nurses to see the data in whatever way they want to.

### The renderer

So we write a little module that creates HTML based on some input. Creating HTML nowadays is
[easy](https://modules.ceylon-lang.org/repo/1/ceylon/html/1.2.0/module-doc/api/index.html).
The real problem to be solved is how to represent the input so that our renderer module is able to have an abstract
representation of it, while being flexible enough to accept changes in the input without requiring modifications.

One way to solve our problem is to decide on just how general we can make our input look like while still being able
to render it in HTML in an appropriate manner. We could attempt to model the data as a table, for example.
So our requirements from the data would be just that it should have some headers, with the actual data following
below them, much like a CSV file or a HTML table.

Notice that we don't need to care about what those headers will be, or even how many they are. We do care, however,
that **each row of the table has the same number of elements as the header**! This is nearly impossible to guarantee at
the type-system level in almost any language out there... however, in Ceylon we actually can!

To achieve that, the first thing we need is a little `Tuple` wrapper to makes the type parameters of `Tuple` *invariant*.
This is necessary to force the arguments of the `render` function that we will define below to acquire the
exact same type, not a union between the types of each argument!

```ceylon
class InvariantTuple<Element, First, Rest>(
    shared Tuple<Element, First, Rest> tuple) {}
```

And a trivial implementation of a `render` function that takes a single header and a non-empty stream of rows:

```ceylon
void render<A, B, C>(InvariantTuple<A, B, C> headers, 
                     {InvariantTuple<A, B, C>+} cells) {
    process.write("<table><thead>");
    headers.tuple.each((item)
        => process.write("<th>``item else "?"``</th>"));
    process.write("</thead><tbody>");
    cells.each((cell) => cell.tuple.each((item)
        => process.write("<td>``item else "?"``</td>")));
    process.write("</tbody></table>");
}
```

> We want a non-empty stream of rows so that we don't need to worry about rendering empty tables... a different renderer
  should be used just to render empty tables so that we can have fine control over an empty table representation.

`Tuple`'s types parameters are covariant, while `InvariantTuple`'s are invariant. This means that when we call the
`render` function, the type parameters `<Element, First, Rest>` must  be exactly the same for headers and cells.

So if we try to call `render` with Tuples of different length, the compiler will not accept our code.
Even if one of the columns has a value of wrong type, the compiler will stop us on our tracks!

```ceylon
// (unfortunately, this definition of row and header only works with the JavaScript back-end
// because Ceylon 1.2 does not support function types in the JVM).
// In the JVM, row and header must be normal functions that create an InvariantTuple explicitly.
value row = InvariantTuple;
value header = row;

// a well-formed table
render(header(["weight", "height"]), {
     row(["82", "1.82"]),
     row(["63", "1.74"])
});

// this WILL NOT COMPILE
render(header(["weight", "height"]), {
     row(["82", "1.82"]),
     row(["63", "1.74", "bad"]) // wrong number of parameters
});

// this WILL NOT COMPILE
render(header(["weight", "height"]), {
     row(["82", "1.82"]),
     row(["63", 1.74]) // cell has wrong type
});
```

Notice that this generalizes to any number of elements, of any type:

```ceylon
render(header([10, 0.1, true, "hello", [4, 0.4]]), {
    row([2, 0.2, false, "bye", [3, 0.3]])
});
```

Even the types of the sub-tuples in the example above must match exactly. This is really impressive for a type-system
to achieve, so please take a moment to appreciate the power of this construct.

Going back to our problem... we now know how to write an extremely generic renderer. But we just rendered the HTML
to `sysout`, which is not very useful for a web app... let's fix that by taking as an argument a function which has the
same signature as `process.write`, which we had used: `Anything(String)`
(in other words, a function that takes a `String` and returns whatever, including void):

```ceylon
void render<A, B, C>(Anything(String) write,
                     InvariantTuple<A, B, C> headers, 
                     {InvariantTuple<A, B, C>*} cells) {
    write("<table><thead>");
    headers.tuple.each((item)
        => write("<th>``item else "?"``</th>"));
    write("</thead><tbody>");
    cells.each((cell) => cell.tuple.each((item)
        => write("<td>``item else "?"``</td>")));
    write("</tbody></table>");
}
```

Now, the caller decides whether to print the HTML to `sysout` or stream it down a `socket` or `file`.

```ceylon
// just render to sysout (console in JS)
render(process.write, header(["weight", "height"]), {
     row(["82", "1.82"]),
     row(["63", "1.74"])
});

// write to a file (requires the ceylon.file module, which is JVM-only)
value fileHandle = let (res = filePath.resource)
    if (is Nil res) then res.createFile()
    else if (is File res) then res
    else null;

if (exists fileHandle) {
    render(fileHandle.Overwriter().write, header([10, 0.1, true, "hello", [4, 0.4]]), {
        row([2, 0.2, false, "bye", [3, 0.3]])
    });
}
```

### Representing the data

Health information is something we, programmers, probably don't know much about... you might be able to invite some medical
practitioners to give advice, but in the end we're the ones who need to come up with a, hopefully, good model to store
the data and allow for possible changes in the future.

In our trivial examples, we had only height and weight represented, but I guess a person's data should contain a lot
more than that... DOB, gender,  known conditions, current medication, allergies... and I'm sure the list goes on and on.

As it's probably impossible to know all of these beforehand anyway, let's try to be very generic in defining the data
model.

I like the idea of having a `Person` class as we can use that to define some mandatory fields which we know must be
present, like `name` and `dob`, just like we would do in the old-fashioned way... we enforce a few constraints on the
name field so that patently wrong input is not accepted by our program (maybe the constraints I define are not ideal,
but that's beyond the point of this blog post).

> to use `Date` for the `dob` field, the Ceylon SDK module `ceylon.time` must be imported.

```ceylon
shared class Name {
    shared actual String string;
    
    throws(`class Exception`, "if the given string is not between 1 and 256 characters long")
    shared new createName(String string) {
        if (1 <= string.size <= 256) {
            this.string = string;    
        } else {
            throw Exception("Name must be between 1 and 256 characters long");
        }
    }
}
    
shared class FullName(shared Name lastName,
                      shared Name? firstName = null, 
                      shared {Name*} otherNames = {}) {}

shared class Person(shared FullName fullName,
                    shared Date dob) {}
```

So, currently, a `Person` has a `fullName` and a `dob`. a `FullName` always contains a `lastName`, but may also contain
a `firstName` and a list of `otherNames`.

The following are examples of valid `Person` instances:

```ceylon
value person1 = Person(FullName(name("Jordan")),
                       date(1963, 2, 17));

value person2 = Person(FullName(name("Jordan"), name("Michael")),
                       date(1963, 2, 17));

value person3 = Person(FullName {
        firstName = name("Michael");
        otherNames = { name("Jeffrey") };
        lastName = name("Jordan");
    },
    date(1963, 2, 17));
```

Now, we add a new field to `Person` that will make it possible to add new attributes to it without losing type-safety.

```ceylon
class Person<out Attribute>(shared FullName fullName, 
             shared Date dob,
             shared {Attribute*} attributes = {})
            given Attribute satisfies Object {}
```

The generic type parameters ensure that under a module which has certain requirements on the `Person` object, we can
get a type-safe representation of its attributes.

For example, in the `legal` module, we will be interested only in the *legal* attributes of a `Person`, so when we want
to access that, we must `narrow` the attributes and see if its there:

```ceylon
// define the legal module's attributes
class SocialSecurityNumber(shared actual String string) {}
class Nationality(shared actual String string) {}
alias LegalAttributes => [SocialSecurityNumber, Nationality];

// create a person with legal attributes some other module's attributes
value person4 = Person(FullName(name("Smith")),
    date(1984, 12, 21), {
        [SocialSecurityNumber("555-555-xxx"), Nationality("New Zealand")],
        "Some other attributes from other modules"
});


// using legal attributes
value legalAttributes = person4.attributes.narrow<LegalAttributes>();

if (legalAttributes.size == 1, exists attributes = legalAttributes.first) {
    value [ssn, nationality] = attributes;
    print("``person4.fullName`` has SSN ``ssn``
           and nationality ``nationality``");
}
```

In the example above, `person4` has `LegalAttributes`, but also has a `String`-typed attribute that just represents
here the unknown type of the other attributes of a `Person` which may be kept by other modules.

Even though in Ceylon you almost never need to provide type parameters to functions (because the types are inferred
from the actual arguments), we need to do it sometimes, as we did when using the `narrow` function, because as it
takes no arguments, there's no information to infer the type from. By giving an explicit type to the `legalAttributes`
variable, we allow type inference to do its work so we can avoid declaring the type parameter:

```ceylon
{LegalAttributes*} legalAttributes = person4.attributes.narrow();
```

Of course, the reason this works in Ceylon is that Ceylon has reified generics (besides powerful type inference).

We could now define *medical* attributes to a person, so that the `medical` module also has its own module-specific
attributes to work with:

```ceylon
class Allergies(shared {String*} substances) {}
class KnownConditions(shared {String*} conditions) {}
class CurrentMedication(shared {String*} medication) {}

alias MedicalAttributes => [Allergies, KnownConditions, CurrentMedication];

value person4 = Person(FullName(name("Smith")),
    date(1984, 12, 21), {
        [SocialSecurityNumber("555-555-xxx"), Nationality("New Zealand")],
        [Allergies {"peanuts"}, KnownConditions {}, CurrentMedication {}]
});

// using medical attributes
{MedicalAttributes*} medicalAttributes = person4.attributes.narrow();

if (medicalAttributes.size == 1, exists attributes = medicalAttributes.first) {
    value allergies = attributes[0];
    value currentMedication = attributes[2];
    print("``person4.fullName`` has allergy to ``allergies.substances``
           and is currently taking ``currentMedication.medication``");
}
```

Note that in the previous examples, two different methods were used to extract information from the
attributes `Tuple`: in the first, we deconstructed the `Tuple` directly, in the second, we accessed the
elements we were interested in by index. Unlike in most languages, both are completely type-safe:

```ceylon
// deconstructing the legal attributes Tuple with types declared explicitly
value [SocialSecurityNumber ssn, Nationality nationality] = attributes;
```

```ceylon
// accessing medical attributes using type-safe index syntax
// notice that if the types of the elements didn't match, this just wouldn't compile!
Allergies allergies = attributes[0];
CurrentMedication currentMedication = attributes[2];
```

In both examples, explicitly declaring the types has the exact same effect as letting Ceylon infer the types. Both are
completely type-safe.




