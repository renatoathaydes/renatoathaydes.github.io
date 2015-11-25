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

## ??

Suppose that our goal is to create a web application to present health reports about a population.
The first thing we need to worry about is, basically, how the data will be presented.
So we write a little module that creates HTML based on some input. Creating HTML nowadays is easy.
The real problem is how to represent the input so that our renderer module is able to represent it, while being
flexible enough to accept changes in the input without requiring modifications.

One way to solve our problem is to decide on just how general we can make our input look like while still being able
to render it in HTML in an appropriate manner. We could attempt to model the data as a table, for example.
So our requirements from the data would be just that it should have some headers, with the actual data following
below them, much like a CSV file or a HTML table.

Notice that we don't need to care about what those headers will be, or even how many they are. We do care, however,
that **each row of the table has the same number of elements as the header**! This is nearly impossible to guarantee at
the type-system level in almost any language out there... however, in Ceylon we actually can!

The first thing we need is a little `Tuple` wrapper to makes the type parameters of `Tuple` *invariant*.
This is necessary to force the arguments of the `render` function that we will define below to acquire the
exact same type, not a union between the types of each argument!

```ceylon
class InvariantTuple<Element, First, Rest>(
    shared Tuple<Element, First, Rest> tuple) {}
```

And a trivial implementation of a `render` function that takes a single header and a non-empty stream of rows:

```ceylon
void render<A, B, C>(InvariantTuple<A, B, C> headers, 
                     {InvariantTuple<A, B, C>*} cells) {
    process.write("<table><thead>");
    headers.tuple.each((item)
        => process.write("<th>``item else "?"``</th>"));
    process.write("</thead><tbody>");
    cells.each((cell) => cell.tuple.each((item)
        => process.write("<td>``item else "?"``</td>")));
    process.write("</tbody></table>");
}
```

`Tuple`'s types parameters are covariant, while `InvariantTuple`'s are invariant. This means that when we call the
`render` function, the type parameters `<Element, First, Rest>` must  be exactly the same for headers and cells.

So if we try to call `render` with Tuples of different length, the compiler will not accept our code.
Even if one of the columns has a value of wrong type, the compiler will stop us on our tracks!

```ceylon
// for conciseness
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

Going back to our problem now... we now know how to write an extremely generic renderer. But we just rendered the HTML
to `sysout`, which is not very useful for a web app... let's fix that by taking as an argument a function which has the
same signature as `process.write`, which we had used: `Anything(String)`
(in other words, a function that takes a `String` and returns whatever):

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




