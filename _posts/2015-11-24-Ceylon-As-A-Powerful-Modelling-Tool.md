# Using Ceylon's type system as a powerful modelling tool

Ceylon has a really powerful type system that allows programmers to express concepts that just are not possible with
most current type systems out there.

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

You can say that a function may succeed, fail or just not be able to run:

```ceylon
Success|Failure|NotRun myFunction(...) { ... }
```

And the type checker will enforce at compile time that the user of the function checks the result before using it:

```ceylon
void consumeSuccess(Success success) {}
void reportFailure(Failure failure) {}
void maybeRunAgain() {}

Success|Failure|NotRun result = myFunction();

// OK
switch(result)
case (is Success) { consumeSuccess(result); }
case (is Failure) { reportFailure(result); }
case (is NotRun) { maybeRunAgain(); }

// DOES NOT COMPILE
consumeSuccess(result);
```

But let's suppose that `Success`, `Failure` and `NotRun` are all subtypes of a type that has a property
`displayMessage`. Then, you could call it directly on any instance of `Success|Failure|NotRun` without checking,
as that would be safe:

```ceylon
// no problem!
print(result.displayMessage);
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

## Designing an application using the Ceylon type system

Suppose that our goal is to create a web application to present health reports about a population.
Starting from the viewpoint of what we want users to see, the first thing we need to worry about is, basically,
how the data will be presented.

The obvious choice is to present the data in HTML so that users can see it right in their browsers, so let's start by
implementing a renderer that does that.

We also need the concrete representation of the data, of course, which will need to be translated into whatever the
renderer takes.

> The renderer could of course *know* about the actual implementation of the data, but if we did that our code
  would become much more brittle, interconnected and resistant to change!

Once we know how to represent the data and present it to the user, we must put the two together in a *glue* module which
converts the data into the generic form the renderer can recognize.

We could also add modules to add data persistence, search functionality, data entry and editing etc. but after we've
implemented the initial modules, it should be clear how the implementation of further functionality should be easy to
integrate into the system.

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

One problem with the current use of `render` is that the header is expected to have the same type as the data rows. This
is obviously wrong... to fix that is a matter of partially separating the types of the items of headers and cells, whilst
still keeping the length of the types bound together. This can be done as shown below:

```ceylon
// explicitly give the type of the data
[[Name, Weight, Height]+] data = [
    ["John", 72, 1.79],
    ["Mary", 61, 1.76],
    ["Ana", 77, 1.89]
];

// convert the data tuples to string tuples of the same length
value stringData = data.map((row)
    => [row[0].string, row[1].string, row[2].string]);

// render the converted tuples... the header length must match the rows' lengths
render(process.write, 
    header(["Name", "Weight", "Height"]), 
    stringData.map(row));
```

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

As a final improvement, we should stop using String concatenation to create the HTML page and start using the more
proper, type-safe tool for the job, which is the `ceylon.html` module!

We also give the type parameter more appropriate names than A, B, C.

```ceylon
shared void render<Element, First, Rest>(
    Anything(String) write,
    InvariantTuple<Element, First, Rest> headers,
    {InvariantTuple<Element, First, Rest>+} cells) {
    
    function asString(Anything item)
        => item?.string else "?";
    
    Html html = Html {
        doctype = html5;
        Head {
            title = "Medical Web";
        };
        Body {
            H1("Medical Web App"),
            Table {
                header = headers.tuple.map(asString).map(Th);
                rows = cells.map((row) => Tr {
                    row.tuple.map(asString).map(Td)
                });
            }
        };
    };
    
    write(html.string);
}
```

This is still a little bit naive... we cannot, for example, include page snippets from different sources... or provide
a custom CSS stylesheet... but to keep this blog post within a reasonable size, let's move on and see how we can
represent the data.

### Representing the data

Health information is something we, programmers, probably don't know much about... you might be able to invite some medical
practitioners to give advice, but in the end we're the ones who need to come up with a, hopefully, good model to store
the data and allow for possible changes in the future.

In our trivial examples, we had only height and weight represented, but I guess a person's data should contain a lot
more than that... DOB, gender,  known conditions, current medication, allergies... and I'm sure the list goes on and on.

As it's probably impossible to know all of these beforehand anyway, let's try to be very generic in defining the data
model.

I like the idea of having a `Person` class as we can use that to define some mandatory fields which we know must be
present, like `name` and `dob`, just like we would do in the old-fashioned way, at least to start with...
we enforce a few constraints on the name field so that patently wrong input is not accepted by our program
(maybe the constraints I define are not ideal, but that's beyond the point of this blog post). We do that with a named
constructor `createName` which verifies programmatically that the given String meets the constraints. It is a shortcoming
of the Ceylon type system that it is still not possible to specify a String with a certain length range... that would only
be possible with [dependent types](https://en.wikipedia.org/wiki/Dependent_type), to my knowledge, but this is still a
subject of research that no practical language has been able to incorporate (undecidability being a major issue). 

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
class Person<out Attribute>(
        shared FullName fullName, 
        shared Date dob,
        shared {Attribute*} attributes = {})
            given Attribute satisfies Object {}
```

The generic type parameters ensure that under a module which has certain requirements on the `Person` object, we can
get a type-safe representation of its attributes.

We limit actual `Attribute` implementation to sub-types of `Object` because we do not want to allow `Null` as a valid
attribute (which would be the case if we did not bound the type parameter)! We could go a little further and bound
 `Attribute` to a new interface, say `PersonAttribute`, with `given Attribute satisfies PersonAttribute`, but that
 seems unnecessary for now.

Let's have a look how at how we can use attributes.

For example, in the `legal` module, we will be interested only in the *legal* attributes of a `Person`, so when we want
to access that, we must `narrow` the attributes and see if its there:

```ceylon
// define the legal module's attributes
shared class SocialSecurityNumber(shared actual String string) {}
shared class Nationality(shared actual String string) {}
shared alias LegalAttributes => [SocialSecurityNumber, Nationality];

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
takes no arguments, there's no information to infer the type from. Of course, the reason this works in Ceylon is that
Ceylon has reified generics (besides powerful type inference).

We could now define *medical* attributes to a person, so that the `medical.details` module also has its own
module-specific attributes:

```ceylon
shared class Allergies(shared {String*} substances) {
    string = substances.string;
}
shared class KnownConditions(shared {String*} conditions) {
    string = conditions.string;
}
shared class CurrentMedication(shared {String*} medication) {
    string = medication.string;
}

shared alias MedicalAttributes => [Allergies, KnownConditions, CurrentMedication];

value person4 = Person(FullName(name("Smith")),
    date(1984, 12, 21), {
        [SocialSecurityNumber("555-555-xxx"), Nationality("New Zealand")],
        [Allergies {"peanuts"}, KnownConditions {}, CurrentMedication {}]
});

// using medical attributes
value medicalAttributes = person4.attributes.narrow<MedicalAttributes>();

if (medicalAttributes.size == 1, exists attributes = medicalAttributes.first) {
    value allergies = attributes[0];
    value currentMedication = attributes[2];
    print("``person4.fullName`` has allergy to ``allergies.substances``
           and is currently taking ``currentMedication.medication``");
}
```

Notice that in the previous examples, two different methods were used to extract information from the
attributes tuple: in the first, we deconstructed the tuple directly, in the second, we accessed the
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

If you attempt to access an index that does not exist, the returned element will be `null`:

```ceylon
// DOES NOT COMPILE
CurrentMedication currentMedication = attributes[3];

// OK, but useless
Null doesNotExist = attributes[3];
```

This should be enough for us to represent a `Person` quite well, with module-specific attributes making our code
really modular, while at the same time allowing us to evolve the model in the future much more easily. Just imagine
that we wanted to change the legal attributes. Not a single module would require changes except for the legal module
itself, and perhaps any other module that depends on it (legal module extensions?).

Storage of a `Person` instance should be easy with either Ceylon
[serialization](https://gist.github.com/gavinking/ca2fba39c73d9dc376ee) or the use of a modern document database
(I would recommend an ACID, scalable document/graph database called [OrientDB](http://orientdb.com/) for that),
for example.

### Putting everything together

Now that we have a data representation and a renderer function, we can write the glue code necessary to make the two
work together.

That's quite easy!

First, we need some functions that turn each type we're interested in adding to the report we will show to the user
into a `String`:

```ceylon
String renderedName(FullName fullName)
    => "``fullName.lastName``\
        ``if (exists fn = fullName.firstName)
            then (", " + fn.string)
            else ""``";

String renderedDate(Date date)
    => date.string; // maybe we should format it on the client's Locale

String renderedSSN(SocialSecurityNumber? ssn)
    => ssn?.string else "UNKNOWN";

String renderedAllergies(Allergies? allergies)
    => if (exists a = allergies) then
            (if (a.substances.empty) then "None"
             else a.substances.string)
       else "UNKNOWN";

String renderedMedication(CurrentMedication? medication)
    => if (exists m = medication) then
            (if (m.medication.empty) then "None"
             else m.medication.string)
       else "UNKNOWN";
```

In which module each function should be located is left as an exercise to the reader ;)

Now we can define the headers of the table we want to display, as well as the type of the cells which will form the
rows of the table:

```ceylon
value headerNames = ["Full name", "Date of birth", "SSN",
                     "Allergies", "Current Medication"];

alias RenderableRow => [FullName, Date, SocialSecurityNumber?, 
                        Allergies?, CurrentMedication?];
```

Each `Person<Anything>` instance needs to be turned into a `RenderableRow`, which our renderer knows how to render:

```ceylon
RenderableRow personRow(Person<Anything> person) {
    value legalAttributes = person.attributes.narrow<LegalAttributes>();
    value medicalAttributes = person.attributes.narrow<MedicalAttributes>();
    
    SocialSecurityNumber? ssn = legalAttributes.first?.first else null;
    
    [Allergies?, CurrentMedication?] medicalColumns;
    if (exists attributes = medicalAttributes.first) {
        medicalColumns = [attributes[0], attributes[2]];
    } else {
        medicalColumns = [null, null];
    }
    
    return [person.fullName, person.dob, ssn, *medicalColumns];
}
```

Next, we define a function that takes a `{Person<Anything>*}` stream, which would probably come from whatever data store
we had at our disposal, turn that into a `{RenderableRow*}` stream and, finally, call the `render` function with the
headers and this stream of rows (if there's at least one row):

```ceylon
void renderPeople({Person<Anything>*} people) {
    value data = people.map(personRow)
            .map((pr) => [renderedName(pr[0]), renderedDate(pr[1]),
                          renderedSSN(pr[2]), renderedAllergies(pr[3]),
                          renderedMedication(pr[4])])
            .map(row);

    if (exists firstRow = data.first) {
        // idiom to turn a {A*} into a {A+}
        value nonEmptyData = { firstRow }.chain(data.rest);
        render(process.write, header(headerNames), nonEmptyData);
    } else {
        print("No people were found");
    }
}
```

That's it... we can test this with a small example:

```ceylon
value person1 = Person {
    fullName = FullName(name("Smith"));
    dob = date(1984, 12, 21);
    attributes = [
        [SocialSecurityNumber("555-555-xxx"), Nationality("New Zealand")],
        [Allergies {"peanuts"}, KnownConditions {}, CurrentMedication {}]
    ];
};

value person2 = Person {
    fullName = FullName {
        firstName = name("Michael");
        lastName = name("Jordan");
    };
    dob = date(1963, 2, 17);
    attributes = [
        [SocialSecurityNumber("555-555-yyy"), Nationality("USA")],
        [Allergies {}, KnownConditions {}, CurrentMedication {"Naproxen"}]
    ];
};

renderPeople { person1, person2 };
```

### Extending the system

Now that we've got a working system, just as in real life, we would probably want to extend the system pretty much
as soon as we were done with the initial design. That's when you'll know whether you succeeded in making your
system change-friendly.

Let's see how the system we have designed so far accommodates change.

Our next objective is to add computed values to the report. Computed values are calculated based on existing attributes
of a person.

The `bmi` and `overweight` functions we saw in the beginning of this post are examples of computed values:

```ceylon
function bmi(Kilogram weight, Meter height)
        => weight.float / (height ^ 2.0);

function overweight(Kilogram weight, Meter height)
        => bmi(weight, height) > 25.0;
```

To add support for these values, first we need to add a new module called `medical.measurement`, which will contain
the functions above (with the return type explicitly declared, as that's mandatory in Ceylon for `shared` functions)
as well as a new attribute to be added to `Person`:

```ceylon
shared alias Kilogram => Integer;
shared alias Meter => Float;

shared class Measurements(
    shared Kilogram weight,
    shared Meter height) {

    shared [Kilogram, Meter] properties = [weight, height];

}

shared Float bmi(Kilogram weight, Meter height)
        => weight.float / (height ^ 2.0);

shared Boolean overweight(Kilogram weight, Meter height)
        => bmi(weight, height) > 25.0;
```

Now we can use this in the web module:

```ceylon

```
