## Making Data Out Of Functions

![Coffee served at the CERN particle accelerator](images/cern-coffee.jpg)

In our code so far, we have used arrays and objects to represent the structure of data, and we have extensively used the ternary operator to write algorithms that terminate when we reach a base case.

For example, this `length` function uses a functions to bind values to names, POJOs to structure nodes, and the ternary function to detect the base case, the empty list.

{:lang="javascript"}
~~~~~~~~
const length = (node, delayed = 0) =>
  node === EMPTY
    ? delayed
    : length(node.rest, delayed + 1);

length(OneTwoThree)
  //=> 3
~~~~~~~~

A very long time ago, mathematicians like Alonzo Church, Moses Schönfinkel, and Alan Turning, and Haskell Curry and asked themselves if we really needed all these features to perform computations. They searched for a radically simpler set of tools that could accomplish all of the same things.

They established that arbitrary computations could be represented with radically simple sets of tools. For example, we don't need arrays to represent lists, or even POJOs to represent nodes in a linked list. We can model lists just using functions.

> [To Mock a Mockingbird](http://www.amazon.com/gp/product/0192801422/ref=as_li_ss_tl?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0192801422) established the metaphor of songbirds for the combinators, and ever since then logicians have called the K combinator a "kestrel," the B combinator a "bluebird," and so forth. The [osin.es] library contains code for all of teh standard combinators and for experimenting using the standard notation.

[To Mock a Mockingbird]: http://www.amazon.com/gp/product/0192801422/ref=as_li_ss_tl?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0192801422
[osin.es]: http://oscin.es

Let's start with some of the building blocks of combinatory logic, the K, I, and V combinators, nicknamed the "Kestrel", "Idiot Bird", and the "Vireo:"

{:lang="javascript"}
~~~~~~~~
const K = (x) => (y) => x;
const I = (x) => (x);
const V = (x) => (y) => (z) => z(x)(y);
~~~~~~~~

### the kestrel and the idiot

A *constant function* is a function that always returns the same thing, no matter what you give it. For example, `(x) => 42` is a constant function that always evaluates to 42. The kestrel, or `K`, is a function that makes constant functions. You give it a value, and it returns a constant function that gives that value.

For example:

{:lang="javascript"}
~~~~~~~~
const K = (x) => (y) => x;

const fortyTwo = K(42);

fortyTwo(6)
  //=> 42

fortyTwo("Hello")
  //=> 42
~~~~~~~~

The *identity function* is a function that evaluates to whatever parameter you pass it. So `I(42) => 42`. Very simple, but useful. Now we'll take it one more step forward: Passing a value to `K` gets a function back, and passing a value to that function gets us a value.

Like so:

{:lang="javascript"}
~~~~~~~~
K(6)(7)
  //=> 6
  
K(12)(24)
  //=> 6
~~~~~~~~

This is very interesting. Given two values, we can say that `K` always returns the *first* value: `K(x)(y) => x` (that's not valid JavaScript, but it's essentially how it works).

Now, an interesting thing happens when we pass functions to each other. Consider `K(I)`. From what we just wrote, `K(x)(y) => x` So `K(I)(x) => I`. Makes sense. Now let's tack one more invocation on: What is `K(I)(x)(y)`? If `K(I)(x) => I`, then `K(I)(x)(y) === I(y)` which is `y`.

Therefore, `K(I)(x)(y) => y`:

{:lang="javascript"}
~~~~~~~~
K(I)(6)(7)
  //=> 7
  
K(I)(12)(24)
  //=> 24
~~~~~~~~

Aha! Given two values, `K(I)` always returns the *second* value.

{:lang="javascript"}
~~~~~~~~
K("primus")("secundus")
  //=> "primus"
  
K(I)("primus")("secundus")
  //=> "secundus"
~~~~~~~~

If we are not feeling particularly academic, we can name our functions:

{:lang="javascript"}
~~~~~~~~
const first = K,
      second = K(I);
      
first("primus")("secundus")
  //=> "primus"
  
second("primus")("secundus")
  //=> "secundus"
~~~~~~~~

### backwardness

Our `first` and `second` functions are a little different than what most people are used to when we talk about functions that access data. If we represented a pair of values as an array, we'd write them like this:

{:lang="javascript"}
~~~~~~~~
const first = ([first, second]) => first,
      second = ([first, second]) => second;
      
const latin = ["primus", "secundus"];
      
first(latin)
  //=> "primus"
  
second(latin)
  //=> "secundus"
~~~~~~~~

Or if we were usinga POJO, we'd write them like this:

{:lang="javascript"}
~~~~~~~~
const first = ({first, second}) => first,
      second = ({first, second}) => second;
      
const latin = {first: "primus", second: "secundus"};
      
first(latin)
  //=> "primus"
  
second(latin)
  //=> "secundus"
~~~~~~~~

In both cases, the functions `first` and `second` know how the data is represented, whether it be an array or an object. You pass the data to these functions, and they extract it.

But the `first` and `second` we built out of `K` and `I` don't work that way. You call them and pass them the bits, and they choose what to return. So if we wanted to use them with a two-element array, we'd need to have a piece of code that calls some code.

Here's the first cut:

{:lang="javascript"}
~~~~~~~~
const first = K,
      second = K(I);
      
const latin = (selector) => selector("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
~~~~~~~~

Our `latin` data structure is no longer a dumb data structure, its a function. And instead of passing `latin` to `first` or `second`, we pass `first` or `second` to `latin`. It's *exactly backwards* of the way we write functions that operate on data.

### the vireo

Given that our `latin` data is represented as the function `(selector) => selector("primus")("secundus")`, our obvious next step is to make a function that makes data. For arrays, we'd write `cons = (first, second) => [first, second]`. For objects we'd write: `cons = (first, second) => {first, second}`. In both cases, we take two parameters, and return the form of the data.

For "data" we access with `K` and `K(I)`, our "structure" is the function `(selector) => selector("primus")("secundus")`. Let's extract those into parameters:

{:lang="javascript"}
~~~~~~~~
(first, second) => (selector) => selector(first)(second)
~~~~~~~~

For consistency with the way combinators are written as functions taking just one parameter, we'll [curry] the function:

{:lang="javascript"}
~~~~~~~~
(first) => (second) => (selector) => selector(first)(second)
~~~~~~~~

[curry]: https://en.wikipedia.org/wiki/Currying

Let's try it, we'll use the word `tuple` for the function that makes data (When we need to refer to a specific tuple, we'll use the name `aTuple` by default):

{:lang="javascript"}
~~~~~~~~
const first = K,
      second = K(I),
      tuple = (first) => (second) => (selector) => selector(first)(second);

const latin = tuple("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
~~~~~~~~

It works! Now what is this `node` function? If we change the names to `x`, `y`, and `z`, we get: `(x) => (y) => (z) => z(x)(y)`. That's the V combinator, the Vireo! So we can write:

{:lang="javascript"}
~~~~~~~~
const first = K,
      second = K(I),
      tuple = V;

const latin = tuple("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
~~~~~~~~

Armed with nothing more than `K`, `I`, and `V`, we can make a little data structure that holds two values, the `cons` cell of Lisp and the node of a linked list. Without arrays, and without objects, just with functions. We'd better try it out to check.

### lists with functions as data

Here's another look at linked lists using POJOs. We use the term `rest` instead of `second`, but it's otherwise identical to what we have above:

{:lang="javascript"}
~~~~~~~~
const first = ({first, rest}) => first,
      rest  = ({first, rest}) => rest,
      tuple = (first, rest) => ({first, rest}),
      EMPTY = ({});
      
const l123 = tuple(1, tuple(2, tuple(3, EMPTY)));

first(l123)
  //=> 1

first(rest(l123))
  //=> 2

first(rest(rest(l123)))
  //=3
~~~~~~~~

We can write `length` and `mapWith` functions over it:

{:lang="javascript"}
~~~~~~~~
const length = (aTuple) =>
  aTuple === EMPTY
    ? delayed
    : 1 + length(rest(aTuple));

length(l123)
  //=> 3

const reverse = (aTuple, delayed = EMPTY) =>
  aTuple === EMPTY
    ? delayed
    : reverse(rest(aTuple), tuple(first(aTuple), delayed));

const mapWith = (fn, aTuple, delayed = EMPTY) =>
  aTuple === EMPTY
    ? reverse(delayed)
    : mapWith(fn, rest(aTuple), tuple(fn(first(aTuple)), delayed));
    
const doubled = mapWith((x) => x * 2, l123);

first(doubled)
  //=> 2

first(rest(doubled))
  //=> 4

first(rest(rest(doubled)))
  //=> 6
~~~~~~~~

Can we do the same with the linked lists we build out of functions? Yes:

{:lang="javascript"}
~~~~~~~~
const first = K,
      rest  = K(I),
      tuple = V,
      EMPTY = (() => {});
      
const l123 = tuple(1)(tuple(2)(tuple(3)(EMPTY)));

l123(first)
  //=> 1

l123(rest)(first)
  //=> 2

return l123(rest)(rest)(first)
  //=> 3
~~~~~~~~

We write them in a backwards way, but they seem to work. How about `length`?

{:lang="javascript"}
~~~~~~~~
const length = (aTuple) =>
  aTuple === EMPTY
    ? 0
    : 1 + length(aTuple(rest));
    
length(l123)
  //=> 3
~~~~~~~~

And `mapWith`?

{:lang="javascript"}
~~~~~~~~
const reverse = (aTuple, delayed = EMPTY) =>
  aTuple === EMPTY
    ? delayed
    : reverse(aTuple(rest), tuple(aTuple(first))(delayed));

const mapWith = (fn, aTuple, delayed = EMPTY) =>
  aTuple === EMPTY
    ? reverse(delayed)
    : mapWith(fn, aTuple(rest), tuple(fn(aTuple(first)))(delayed));
    
const doubled = mapWith((x) => x * 2, l123)

doubled(first)
  //=> 2

doubled(rest)(first)
  //=> 4

doubled(rest)(rest)(first)
  //=> 6
~~~~~~~~

Presto, **we can use pure functions to represent a linked list**. And with care, we can do amazing things like use functions to represent numbers, build more complex data structures like trees, and in fact, anything that can be computed can be computed using just functions and nothing else.

But without building our way up to something insane like writing a JavaScript interpreter using JavaScript functions and no other data structures, let's take things another step in a slightly different direction.

We used functions to replace arrays and POJOs, but we still use JavaScript's built-in operators to test for equality (`===`) and to branch `?:`.

We'll see how to do that, next.

### say "please"

We keep using the same pattern in our functions: `aTuple === EMPTY ? doSomething : doSomethingElse`. This follows the philosophy we used with data structures: The function doing the work inspects the data structure.

We can reverse this: Instead of asking a tuple if it is empty and then deciding what to do, we can ask the tuple to do it for us. Here's `length` again:

{:lang="javascript"}
~~~~~~~~
const length = (aTuple) =>
  aTuple === EMPTY
    ? delayed
    : 1 + length(rest(aTuple));
~~~~~~~~

Let's presume we are working with a slightly higher abstraction, we'll call it a `list`. Instead of writing `length(list)` and examining a list, we'll write something like:

{:lang="javascript"}
~~~~~~~~
const length = (list) => list(
  () => 0,
  (aTuple) => 1 + length(aTuple(rest)))
);
~~~~~~~~

Now we'll need to write `first` and `rest` functions for a list, and those names will collide with the `first` and `rest` we wrote for tuples. So let's disambinuage our names:

{:lang="javascript"}
~~~~~~~~
const tupleFirst = K,
      tupleRest  = K(I),
      tuple = V;
      
const first = (list) => list(
    () => "ERROR: Can't take first of an empty list",
    (aTuple) => aTuple(tupleFirst)
  );
      
const rest = (list) => list(
    () => "ERROR: Can't take first of an empty list",
    (aTuple) => aTuple(tupleRest)
  );

const length = (list) => list(
    () => 0,
    (aTuple) => 1 + length(aTuple(tupleRest)))
  );
~~~~~~~~

We'll also write a handy list printer:

{:lang="javascript"}
~~~~~~~~
const print = (list) => list(
    () => "",
    (aTuple) => `${aTuple(tupleFirst)} ${print(aTuple(tupleRest))}`
  );
~~~~~~~~

How would all this work? Let's start with the obvious. What is an empty list?

{:lang="javascript"}
~~~~~~~~
const EMPTYLIST = (whenEmpty, unlessEmpty) => whenEmpty()
~~~~~~~~

And what is a node of a list?

{:lang="javascript"}
~~~~~~~~
const node = (x) => (y) =>
  (whenEmpty, unlessEmpty) => unlessEmpty(tuple(x)(y));
~~~~~~~~

Let's try it:

{:lang="javascript"}
~~~~~~~~
const l123 = node(1)(node(2)(node(3)(EMPTYLIST)));

print(l123)
  //=> 1 2 3
~~~~~~~~

We can write `reverse` and `mapWith` as well. We aren't being super-strict about emulating combinatory logic, we'll use default parameters:

{:lang="javascript"}
~~~~~~~~
const reverse = (list, delayed = EMPTYLIST) => list(
  () => delayed,
  (aTuple) => reverse(aTuple(tupleRest), node(aTuple(tupleFirst))(delayed))
);

print(reverse(l123));
  //=> 3 2 1
  
const mapWith = (fn, list, delayed = EMPTYLIST) =>
  list(
    () => reverse(delayed),
    (aTuple) => mapWith(fn, aTuple(tupleRest), node(fn(aTuple(tupleFirst)))(delayed))
  );
  
print(mapWith(x => x * x, reverse(l123)))
  //=> 941
~~~~~~~~

We have managed to provide the exact same functionality that `===` and `?:` provided, but using  functions only.

### summary

Functions are a fundamental building block of computation. They are "axioms" of combinatory logic, and can be used to comput anything that JavaScript can compute. We've taken a cusory look at how this can be done by using functions to represent linked lists and some of the operations on them.