## Tail Calls, Default Arguments, and Excessive Recycling {#tail}

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

Let's step through its execution. First, `mapWith((x) => x * x, [1, 2, 3, 4, 5])` is invoked. `first` is not `undefined`, so it evaluates [fn(first), ...mapWith(fn, rest)]. To do that, it has to evaluate `fn(first)` and `mapWith(fn, rest)`, then evaluate [fn(first), ...mapWith(fn, rest)].

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

Is there a better way? Several, in fact, fast algorithms is a very highly studied field of computer science. The one we're going to look at here is called *tail-call optimization*, or "TCO."

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
      for (let arg in args) {
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

## Garbage, Garbage Everywhere

![Garbage Day](images/garbage.jpg)
    
We have now seen how to use [Tail Calls](#tail) to execute `mapWith` in constant space:

{:lang="javascript"}
~~~~~~~~
const mapWith = (fn, [first, ...rest], prepend = []) =>
  first === undefined
    ? prepend
    : mapWith(fn, rest, [...prepend, fn(first)]);
                                                  
mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
~~~~~~~~

But when we try it on very large arrays, we discover that it is *still* very slow. Much slower than the built-in `.map` method for arrays. The right tool to discover why it's still slow is a memory profiler, but a simple inspection of the program will reveal the following:

Every time we call `mapWith`, we're calling `[...prepend, fn(first)]`. To do that, we take the array in `prepend` and push `fn(first)` onto the end, creating a new array that will be passed to the next invocation of `mapWith`.

Worse, the JavaScript Engine actually copies the elements from `prepend` into the new array one at a time. That is very laborious.[^cow]

[^cow]: It needn't always be so: Programmers have developed specialized data structures that make operations like this cheap, often by arranging for structures to share common elements by default, and only making copies when changes are made. But this is not how JavaScript's built-in arrays work.

The array we had in `prepend` is no longer used. In GC environments, it is marked as no longer being used, and eventually the garbage collector recycles the memory it is using. Lather, rinse, repeat: Ever time we call `mapWith`, we're creating a new array, copying all the elements from `prepend` into the new array, and then we no longer use `prepend`.

We may not be creating 3,000 stack frames, but we are creating three thousand new arrays and copying elements into each and every one of them. Although the maximum amount of memory does not grow, the thrashing as we create short-lived arrays is very bad, and we do a lot of work copying elements from one array to another.

> **Key Point**: Our `[first, ...rest]` approach to recursion is slow because that it creates a lot of temporary arrays, and it spends an enormous amount of time copying elements into arrays that end up being discarded. 

So here's a question: If this is such a slow approach, why do some examples of "functional" algorithms work this exact way?

![The IBM 704](images/IBM704.jpg)

### some history

Once upon a time, there was a programming language called [Lisp], an acronym for LISt Processing.[^lisp] Lisp was one of the very first high-level languages, the very first implementation was written for the [IBM 704] computer. (The very first FORTRAN implementation was also written for the 704).

[Lisp]: https://en.wikipedia.org/wiki/Lisp_(programming_language)
[IBM 704]: https://en.wikipedia.org/wiki/IBM_704
[^lisp]: Lisp is still very much alive, and one of the most interesting and exciting programming languages in use today is [Clojure](http://clojure.org/), a Lisp dialect that runs on the JVM, along with its sibling [ClojureScript](https://github.com/clojure/clojurescript), Clojure that transpiles to JavaScript.

The 704 had a 36-bit word, meaning that it was very fast to store and retrieve 36-bit values. The CPU's instruction set featured two important macros: `CAR` would fetch 15 bits representing the Contents of the Address part of the Register, while `CDR` would fetch the Contents of the Decrement part of the Register.

In broad terms, this means that a single 36-bit word could store two separate 15-bit values and it was very fast to save and retrieve pairs of values. If you had two 15-bit values and wished to write them to the register, the `CONS` macro would take the values and write them to a 36-bit word.

Thus, `CONS` put two values together, `CAR` extracted one, and `CDR` extracted the other. Lisp's basic data type is often said to be the list, but in actuality it was the "cons cell," the term used to describe two 15-bit values stored in one word. The 15-bit values were used as pointers that could refer to a location in memory, so in effect, a cons cell was a little data structure with two pointers to other cons cells.

Lists were represented as linked lists of cons cells, with each cell's head pointing to an element and the tail pointing to another cons cell.

> Having these instructions be very fast was important to those early designers: They were working on one of the first high-level languages (COBOL and FORTRAN being the others), and computers in the late 1950s were extremely small and slow by today's standards. Although the 704 used core memory, it still used vacuum tubes for its logic. Thus, the design of programming languages and algorithms was driven by what could be accomplished with limited memory and performance.

Here's the scheme in JavaScript, using two-element arrays to represent cons cells:

{:lang="javascript"}
~~~~~~~~
const cons = (a, d) => [a, d],
      car  = ([a, d]) => a,
      cdr  = ([a, d]) => d;
~~~~~~~~
      
We can make a list by calling `cons` repeatedly, and terminating it with `null`:

{:lang="javascript"}
~~~~~~~~
const oneToFive = cons(1, cons(2, cons(3, cons(4, cons(5, null)))));

oneToFive
  //=> [1,[2,[3,[4,[5,null]]]]]
~~~~~~~~

Notice that though JavaScript displays our list as if it is composed of arrays nested within each other like Russian Dolls, in reality the arrays refer to each other with references, so `[1,[2,[3,[4,[5,null]]]]]` is actually more like:

{:lang="javascript"}
~~~~~~~~
const node5 = [5,null],
      node4 = [4, node5],
      node3 = [3, node4],
      node2 = [2, node3],
      node1 = [1, node2];
    
const oneToFive = node1;
~~~~~~~~

This is a [Linked List](https://en.wikipedia.org/wiki/Linked_list), it's just that those early Lispers used the names `car` and `cdr` after the hardware instructions, whereas today we use words like `data` and `reference`. But it works the same way: If we want the head of a list, we call `car` on it:

{:lang="javascript"}
~~~~~~~~
car(oneToFive)
  //=> 1
~~~~~~~~
      
`car` is very fast, it simply extracts the first element of the cons cell.

But what about the rest of the list? `cdr` does the trick:

{:lang="javascript"}
~~~~~~~~
cdr(oneToFive)
  //=> [2,[3,[4,[5,null]]]]
~~~~~~~~
      
Again, it's just extracting a reference from a cons cell, it's very fast. In Lisp, it's blazingly fast because it happens in hardware. There's no making copies of arrays, the time to `cdr` a list with five elements is the same as the time to `cdr` a list with 5,000 elements, and no temporary arrays are needed. In JavaScript, it's still much, much, much faster to get all the elements except the head from a linked list than from an array. Getting one reference to a structure that already exists is faster than copying a bunch of elements.

So now we understand that in Lisp, a lot of things use linked lists, and they do that in part because it was what the hardware made possible.

Getting back to JavaScript now, when we write `[first, ...rest]` to gather or spread arrays, we're emulating the semantics of `car` and `cdr`, but not the implementation. We're doing something laborious and memory-inefficient compared to using a linked list as Lisp did and as we can still do if we choose.

That being said, it is easy to understand and helps us grasp how literals and destructuring works, and how recursive algorithms ought to mirror the self-similarity of the data structures they manipulate. And so it is today that languages like JavaScript have arrays that are slow to split into the equivalent of a `car`/`cdr` pair, but instructional examples of recursive programs still have echoes of their Lisp origins.

### so why arrays

If `[first, ...rest]` is so slow, why does JavaScript use arrays instead of making everything a linked list?

Well, linked lists are fast for a few things, like taking the front element off a list, and taking the remainder of a list. But not for iterating over a list: Pointer chasing through memory is quite a bit slower than incrementing an index. In addition to the extra fetches to dereference pointers, pointer chasing suffers from cache misses. And if you want an arbitrary item from a list, you have to iterate through the list element by element, whereas with the indexed array you just fetch it.

We have avoided discussing rebinding and mutating values, but if we want to change elements of our lists, the naïve linked list implementation suffers as well: When we take the `cdr` of a linked list, we are sharing the elements. If we make any change other than cons-ing a new element to the front, we are changing both the new list and the old list.

Arrays avoid this problem by pessimistically copying all the references whenever we extract an element or sequence of elements from them.

For these and other reasons, almost all languages today make it possible to use a fast array or vector type that is optimized for iteration, and even Lisp now has a variety of data structures that are optimized for specific use cases.

### summary

Although we showed how to use tail calls to map and fold over arrays with `[first, ...rest]`, in reality this is not how it ought to be done. But it is an extremely simple illustration of how recursion works when you have a self-similar means of constructing a data structure.