A DSL that focuses on defining sequences of operations, in the ilk of RxJava and such.

First there are atoms; just single things. They act like single things in pipelines:

```
'a' // a single thing
```

Then there are lists, which act like sequences of things in pipelines:

```
[ 1, 2, 3 ] // three things, in left-to-right order
```

Lists are defined by `[]` and contain other symbols and expressions. The list evaluates shallowly, so whatever it contains remains in its given state.

And then there are tuples, which act like single things in pipelines, but represent more than one thing:

```
('cat', 4, false) // three things, delivered as one
```

And then there are records, which are like tuples whose component values are named:

```
{ name: 'Mark', age: 42 }
```

You can define symbols which act as shortcuts to other entities:

```
A: [ 1, 2, 3 ]
```

Values and symbols are all immutable; note the correspondence between record member assignments and symbol definitions.

Like in normal programming languages, you have operations you can apply to the things defined above, using typical functional notation:

```
print('Hello, world')
```

However, you can also 'stream' arguments into operations. For example, this will print each of the items in the list defined by `A`, in order:

```
A -> print
```

In this stream (which we call a pipeline), `print` is a 'sink', accepting a single argument and yielding none. Since this pipeline starts with a source and ends
with a sink, it constitutes a complete operation. Since it's not assigned to something, it will execute eagerly. If the pipeline were incomplete (requiring input,
or yielding unused results), it would trigger a compile error, just as if you'd tried to execute a function without passing sufficient arguments to it.

A couple definitions now. An 'operation' is something that takes values and does some work on them. Operations may or may not have side effects. Operations that have side
effects, are called 'actions'. Operations that have side effects and also return values are called 'taps'; operations that have side effects and don't return values are called
'sinks'. Operations that don't have side effects, but return values, are called 'functions'. Operations with neither side effects nor return values are called 'useless'. The
different kinds of operation behave differently; the compiler knows how they differ, and will raise errors if they aren't matched up properly (for example if you try to
use a sink as an input, or leave an operation with outputs hanging).

A few examples of operations, for use later in this document:

```
add     // A 'function' that takes two numbers and returns their sum as a single value
log     // A 'tap' that prints a value to the log, and passes it through unmodified
print   // A 'sink' that prints a value to the log
```

What if you have an operation that accepts multiple arguments; can you stream those as well? Yes. You can 'partially apply' the operation by fixing all but one argument,
representing the streamed argument as an underscore:

```
A -> add(3, _) -> print   // if A is [1, 2, 3] then the program will print 4, 5 and 6 in that order.
```

`add(3, _)` can be abbreviated as just `add(3)`, as long as it's the last argument being streamed.

You can also stream a tuple of appropriate size into an operation that takes multiple arguments; the tuple elements will be 'unboxed' appropriately:

```
[(1,1), (2,2), (3,3)] -> add -> print   // prints 2, 4, 6
```

The language is strict about ensuring that the right number of elements make it to the operation, to limit accidents; but see 'pattern matching', below.

The lines above show a pipeline being executed immediately. You can also assign pipelines to symbols. Symbols can receive complete pipelines, like those shown above; this defers
execution of the pipeline until the symbol itself is used in an executing context. Symbols can also receive 'pipeline fragments', which could be pieces of pipelines that are
not bound to inputs, or have unconsumed outputs, or both. Symbols take on the characteristics of the thing they are assigned to; for example, if you assign a pipeline fragment
that needs an input stream, but ends in a sink, the symbol will itself act like a sink:

```
printThreeMore = add(3, _) -> print   // A -> printThreeMore pretty much does what you'd expect
```

All of the operations listed above are 'scalar', meaning that the operation itself acts on one item at a time. There is a whole other category of operations that don't. These
require special treatment, because the operations above are nice and safe: however many inputs you supply, you'll always get the same number of results. We can easily be
sure that you won't overflow or underrun the pipe. Once it's possible for an operation to yield more results than it takes in, or consume more than one input, there's a whole
new category of failure possible.

One deceptively simple nonscalar operation is the 'generator'. A generator takes a fixed number of values (possibly zero) and yields many.
