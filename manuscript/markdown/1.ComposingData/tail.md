## Tail Calls (and Default Arguments) {#tail}

The `mapWith` and `foldWith` functions we wrote in [Self-Similarity](#linear-recursion) are useful for illustrating the basic principles behind using recursion to work with self-similar data structures, but they are not "production-ready" implementations. One of the reasons they are not production-ready is that they consume memory proportional to the size of the array being folded.

Let's look at how. Here's our extremely simple `mapWith` function again:

{:lang="javascript"}
~~~~~~~~
const mapWith = (fn, [first, ...rest]) =>
  first === undefined
    ? []
    : [fn(first), ...mapWith(fn, rest)];

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
~~~~~~~~

Let's step through its execution. First, `mapWith((x) => x * x, [1, 2, 3, 4, 5])` is invoked. `first` is not `undefined`, so it evaluates [fn(first), ...mapWith(fn, rest)]. To do that, it has to evaluate `fn(first)` and `mapWith(fn, rest)`, then evaluate `[fn(first), ...mapWith(fn, rest)]`.

This is roughly equivalent to writing:

{:lang="javascript"}
~~~~~~~~
const mapWith = function (fn, [first, ...rest]) {
  if (first === undefined) {
    return [];
  }
  else {
    const _temp1 = fn(first),
          _temp2 = mapWith(fn, rest),
          _temp3 = [_temp1, ..._temp2];

    return _temp3;
  }
}
~~~~~~~~

Note that while evaluating `mapWith(fn, rest)`, JavaScript must retain the value `first` or `fn(first)`, plus some housekeeping information so it remembers what to do with `mapWith(fn, rest)` when it has a result. JavaScript cannot throw `first` away. So we know that JavaScript is going to hang on to `1`.

Next, JavaScript invokes `mapWith(fn, rest)`, which is semantically equivalent to `mapWith((x) => x * x, [2, 3, 4, 5])`. And the same thing happens: JavaScript has to hang on to `2` (or `4`, or both, depending on the implementation), plus some housekeeping information so it remembers what to do with that value, while it calls the equivalent of `mapWith((x) => x * x, [3, 4, 5])`.

This keeps on happening, so that JavaScript collects the values `1`, `2`, `3`, `4`, and `5` plus housekeeping information by the time it calls `mapWith((x) => x * x, [])`. It can start assembling the resulting array and start discarding the information it is saving.

That information is saved on a *call stack*, and it is quite expensive. Furthermore, doubling the length of an array will double the amount of space we need on the stack, plus double all the work required to set up and tear down the housekeeping data for each call (these are called *call frames*, and they include the place where the function was called, an environment, and so on).

In practice, using a method like this with more than about 50 items in an array may cause some implementations to run very slow, run out of memory and freeze, or cause an error.

{:lang="javascript"}
~~~~~~~~
mapWith((x) => x * x, [
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
  10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
  20, 21, 22, 23, 24, 25, 26, 27, 28, 29,
  30, 31, 32, 33, 34, 35, 36, 37, 38, 39,
  40, 41, 42, 43, 44, 45, 46, 47, 48, 49,
  50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69,
  70, 71, 72, 73, 74, 75, 76, 77, 78, 79,
  80, 81, 82, 83, 84, 85, 86, 87, 88, 89,
  90, 91, 92, 93, 94, 95, 96, 97, 98, 99,
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
  10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
  20, 21, 22, 23, 24, 25, 26, 27, 28, 29,
  30, 31, 32, 33, 34, 35, 36, 37, 38, 39,
  40, 41, 42, 43, 44, 45, 46, 47, 48, 49,
  50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69,
  70, 71, 72, 73, 74, 75, 76, 77, 78, 79,
  80, 81, 82, 83, 84, 85, 86, 87, 88, 89,
  90, 91, 92, 93, 94, 95, 96, 97, 98, 99
])
  //=> ???
~~~~~~~~

Is there a better way? Yes. In fact, there are several better ways. Making algorithms faster is a very highly studied field of computer science. The one we're going to look at here is called *tail-call optimization*, or "TCO."

### tail-call optimization

A "tail-call" occurs when a function's last act is to invoke another function, and then return whatever the other function returns. For example, consider the `maybe` function decorator:

{:lang="javascript"}
~~~~~~~~
const maybe = (fn) =>
  function (...args) {
    if (args.length === 0) {
      return;
    }
    else {
      for (let arg of args) {
        if (arg == null) return;
      }
      return fn.apply(this, args);
    }
  }
~~~~~~~~

There are three places it returns. The first two don't return anything, they don't matter. But the third is `fn.apply(this, args)`. This is a tail-call, because it invokes another function and returns its result. This is interesting, because after sorting out what to supply as arguments (`this`, `args`), JavaScript can throw away everything in its current stack frame. It isn't going to do any more work, so it can throw its existing stack frame away.

And in fact, it does exactly that: It throws the stack frame away, and does not consume extra memory when making a `maybe`-wrapped call. This is a very important characteristic of JavaScript: **If a function makes a call in tail position, JavaScript optimizes away the function call overhead and stack space.**

That is excellent, but one wrapping is not a big deal. When would we really care? Consider this implementation of `length`:

{:lang="javascript"}
~~~~~~~~
const length = ([first, ...rest]) =>
  first === undefined
    ? 0
    : 1 + length(rest);
~~~~~~~~

The `length` function calls itself, but it is not a tail-call, because it returns `1 + length(rest)`, not `length(rest)`.

The problem can be stated in such a way that the answer is obvious: `length` does not call itself in tail position, because it has to do two pieces of work, and while one of them is in the recursive call to `length`, the other happens after the recursive call.

The obvious solution?

### converting non-tail-calls to tail-calls

The obvious solution is push the `1 +` work into the call to `length`. Here's our first cut:

{:lang="javascript"}
~~~~~~~~
const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
  first === undefined
    ? 0 + numberToBeAdded
    : lengthDelaysWork(rest, 1 + numberToBeAdded)

lengthDelaysWork(["foo", "bar", "baz"], 0)
  //=> 3
~~~~~~~~

This `lengthDelaysWork` function calls itself in tail position. The `1 +` work is done before calling itself, and by the time it reaches the terminal position, it has the answer. Now that we've seen how it works, we can clean up the `0 + numberToBeAdded` business. But while we're doing that, it's annoying to remember to call it with a zero. Let's fix that:

{:lang="javascript"}
~~~~~~~~
const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
  first === undefined
    ? numberToBeAdded
    : lengthDelaysWork(rest, 1 + numberToBeAdded)

const length = (n) =>
  lengthDelaysWork(n, 0);
~~~~~~~~

Or we could use partial application:

{:lang="javascript"}
~~~~~~~~
const callLast = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const length = callLast(lengthDelaysWork, 0);

length(["foo", "bar", "baz"])
  //=> 3
~~~~~~~~

This version of `length` calls uses `lengthDelaysWork`, and JavaScript optimizes that not to take up memory proportional to the length of the string. We can use this technique with `mapWith`:

{:lang="javascript"}
~~~~~~~~
const mapWithDelaysWork = (fn, [first, ...rest], prepend) =>
  first === undefined
    ? prepend
    : mapWithDelaysWork(fn, rest, [...prepend, fn(first)]);

const mapWith = callLast(mapWithDelaysWork, []);

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
~~~~~~~~

We can use it with ridiculously large arrays:

{:lang="javascript"}
~~~~~~~~
mapWith((x) => x * x, [
     0,    1,    2,    3,    4,    5,    6,    7,    8,    9,
    10,   11,   12,   13,   14,   15,   16,   17,   18,   19,
    20,   21,   22,   23,   24,   25,   26,   27,   28,   29,
    30,   31,   32,   33,   34,   35,   36,   37,   38,   39,
    40,   41,   42,   43,   44,   45,   46,   47,   48,   49,
    50,   51,   52,   53,   54,   55,   56,   57,   58,   59,
    60,   61,   62,   63,   64,   65,   66,   67,   68,   69,
    70,   71,   72,   73,   74,   75,   76,   77,   78,   79,
    80,   81,   82,   83,   84,   85,   86,   87,   88,   89,
    90,   91,   92,   93,   94,   95,   96,   97,   98,   99,

  // ...

  2980, 2981, 2982, 2983, 2984, 2985, 2986, 2987, 2988, 2989,
  2990, 2991, 2992, 2993, 2994, 2995, 2996, 2997, 2998, 2999 ])

  //=> [0,1,4,9,16,25,36,49,64,81,100,121,144,169,196, ...
~~~~~~~~

Brilliant! We can map over large arrays without incurring all the memory and performance overhead of non-tail-calls. And this basic transformation from a recursive function that does not make a tail call, into a recursive function that calls itself in tail position, is a bread-and-butter pattern for programmers using a language that incorporates tail-call optimization.

### factorials

Introductions to recursion often mention calculating factorials:

> In mathematics, the factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`. For example:

{:lang="javascript"}
~~~~~~~~
5! = 5  x  4  x  3  x  2  x  1 = 120.
~~~~~~~~

The naïve function for calcuating the factorial of a positive integer follows directly from the definition:

{:lang="javascript"}
~~~~~~~~
const factorial = (n) =>
  n == 1
  ? n
  : n * factorial(n - 1);

factorial(1)
  //=> 1

factorial(5)
  //=> 120
~~~~~~~~

While this is mathematically elegant, it is computational [filigree].

[filigree]: https://en.wikipedia.org/wiki/Filigree

Once again, it is not tail-recursive, it needs to save the stack with each invocation so that it can take the result returned and compute `n * factorial(n - 1)`. We can do the same conversion, pass in the work to be done:

{:lang="javascript"}
~~~~~~~~
const factorialWithDelayedWork = (n, work) =>
  n === 1
  ? work
  : factorialWithDelayedWork(n - 1, n * work);

const factorial = (n) =>
  factorialWithDelayedWork(n, 1);
~~~~~~~~

Or we could use partial application:

{:lang="javascript"}
~~~~~~~~
const callLast = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const factorial = callLast(factorialWithDelayedWork, 1);

factorial(1)
  //=> 1

factorial(5)
  //=> 120
~~~~~~~~

As before, we wrote a `factorialWithDelayedWork` function, then used partial application (`callLast`) to make a `factorial` function that took just the one argument and supplied the initial work value.

### default arguments

Our problem is that we can directly write:

{:lang="javascript"}
~~~~~~~~
const factorial = (n, work) =>
  n === 1
  ? work
  : factorial(n - 1, n * work);

factorial(1, 1)
  //=> 1

factorial(5, 1)
  //=> 120
~~~~~~~~

But it is hideous to have to always add a `1` parameter, we'd be demanding that everyone using the `factorial` function know that we are using a tail-recursive implementation.

What we really want is this: We want to write something like `factorial(6)`, and have JavaScript automatically know that we really mean `factorial(6, 1)`. But when it calls itself, it will call `factorial(5, 6)` and that will not mean `factorial(5, 1)`.

JavaScript provides this exact syntax, it's called a *default argument*, and it looks like this:

{:lang="javascript"}
~~~~~~~~
const factorial = (n, work = 1) =>
  n === 1
  ? work
  : factorial(n - 1, n * work);

factorial(1)
  //=> 1

factorial(6)
  //=> 720
~~~~~~~~

By writing our parameter list as `(n, work = 1) =>`, we're stating that if a second parameter is not provided, `work` is to be bound to `1`. We can do similar things with our other tail-recursive functions:

{:lang="javascript"}
~~~~~~~~
const length = ([first, ...rest], numberToBeAdded = 0) =>
  first === undefined
    ? numberToBeAdded
    : length(rest, 1 + numberToBeAdded)

length(["foo", "bar", "baz"])
  //=> 3

const mapWith = (fn, [first, ...rest], prepend = []) =>
  first === undefined
    ? prepend
    : mapWith(fn, rest, [...prepend, fn(first)]);

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
~~~~~~~~

Now we don't need to use two functions. A default argument is concise and readable.

### defaults and destructuring

We saw earlier that destructuring parameters works the same way as destructuring assignment. Now we learn that we can create a default parameter argument. Can we create a default destructuring assignment?

{:lang="javascript"}
~~~~~~~~

const [first, second = "two"] = ["one"];

`${first} . ${second}`
  //=> "one . two"

const [first, second = "two"] = ["primus", "secundus"];

`${first} . ${second}`
  //=> "primus . secundus"
~~~~~~~~

How very useful: defaults can be supplied for destructuring assignments, just like defaults for parameters.
