## Tail Calls,  Excessive Recycling, and the 704 {#tail}

The `mapWith` and `foldWith` functions we wrote are useful for illustrating the basic principles behind using recursion to work with self-similar data structures, but they are not "production-ready" implementations. One of the reasons they are not production-ready is that they consume memory proportional to the size of the array being folded.

Let's look at how. Here's our extremely simple `mapWith` function again:

    const mapWith = (fn, [first, ...rest]) =>
      first === undefined
        ? []
        : [fn(first), ...mapWith(fn, rest)];
                                                  
    mapWith((x) => x * x, [1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]

Let's step through its execution. First, `mapWith((x) => x * x, [1, 2, 3, 4, 5])` is invoked. `first` is not `undefined`, so it evaluates [fn(first), ...mapWith(fn, rest)]. To do that, it has to evaluate `fn(first)` and `mapWith(fn, rest)`, then evaluate [fn(first), ...mapWith(fn, rest)].

This is roughly equivalent to writing:

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
    
Note that while evaluating `mapWith(fn, rest)`, JavaScript must retain the value `first` or `fn(first)`, plus some housekeeping information so it remembers what to do with `mapWith(fn, rest)` when it has a result. JavaScript cannot throw `first` away. So we know that JavaScript is going to hang on to `1`.

Next, JavaScript invokes `mapWith(fn, rest)`, which is semantically equivalent to `mapWith((x) => x * x, [2, 3, 4, 5])`. And the same thing happens: JavaScript has to hang on to `2` (or `4`, or both, depending o the implementation), plus some hosuekeeping information so it remembers what to do with that value, while it calls the equivalent of `mapWith((x) => x * x, [3, 4, 5])`.

This keeps on happening, so that JavaScript collects the values `1`, `2`, `3`, `4`, and `5` plus housekeeping information by the time it calls `mapWith((x) => x * x, [])`. It can start assembing the resulting array and start discarding the information it is saving.

That information is saved on a *call stack*, and it is quite expensive. Furthermore, doubling the length of an array will double the amount of space we need on the stack, plus double all the work required to set up and tear down the housekeeping data for each call (these are called *call frames*, and they include the place where the function was called, an environment, and so on).

In practice, using a method like this with more than about 50 items in an array may cause some implementations to run very slow, run out of memory and freeze, or cause an error.
                                                  
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

Is there a better way? Several, in fact, fast algorithms is a very highly studied field of computer science. The one we're going to look at here is called *tail-call optimization*, or "TCO."

### tco

A "tail-call" occurs when a function's last act is to invoke another function, and then return whatever the other function returns. For example, consider the `maybe` function decorator we saw earlier:

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

There are three places it returns. The first two don't return anything, they don't matter. But the third is `fn.apply(this, args)`. This is a tail-call, because it invokes another function and returns its result. This is interesting, because after sorting out what to supply as arguments (`this`, `args`), JavaScript can throw away everything in its current stack frame. It isn't going to do any more work, so it can throw its existing stack frame away.

And in fact, it does exactly that: It throws the stack frame away, and does not consume extra memory when making a `maybe`-wrapped call. That being said, one wrapping is not a big deal. But consider this:

    const length = ([first, ...rest]) =>
      first === undefined
        ? 0
        : 1 + length(rest);
        
The `length` function calls itself, but it is not a tail-call, because it returns `1 + length(rest)`, not `length(rest)`.

The problem can be stated in such a way that the answer is obvious: `length` does not call itself in tail position, because it has to do two pieces of work, and while one of them is in the recursive call to `length`, the other happens after the recursive call.

The obvious solution?

### converting non-tail-calls to tail-calls

The obvious solution is push the `1 +` work into the call to `length`. Here's our first cut:

    const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
      first === undefined
        ? 0 + numberToBeAdded
        : lengthDelaysWork(rest, 1 + numberToBeAdded)
    
    lengthDelaysWork(["foo", "bar", "baz"], 0)
      //=> 3
      
This `lengthDelaysWork` function calls itself in tail position. The `1 +` work is done before calling itself, and by the time it reaches the terminal position, it has the answer. Now that we've seen how it works, we can clean up the `0 + numberToBeAdded` business. But while we're doing that, it's annoying to remember to call it with a zero. Let's fix that with `callLast`:

    const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
      first === undefined
        ? numberToBeAdded
        : lengthDelaysWork(rest, 1 + numberToBeAdded)
  
    const length = callLast(lengthDelaysWork, 0);
    
    length(["foo", "bar", "baz"])
      //=> 3
      
This version of `length` calls uses `lengthDelaysWork`, and JavaScript optimizes that not to take up memory proportional to the length of the string. We can use this technique with `mapWith`:

    const mapWithDelaysWork = (fn, [first, ...rest], prepend) =>
      first === undefined
        ? prepend
        : mapWith(fn, rest, [...prepend, fn(first)]);
        
    const mapWith = callLast(mapWithDelaysWork, []);
                                                  
    mapWith((x) => x * x, [1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
      
We can use it with ridiculusly large arrays:

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
    
Brilliant! We can map over large arrays without incurring all the memory and performance overhead of non-tail-calls. And this basic trasnformation from a recursive function that does not make a tail call into a tail call is 

### recycling
    
Now that being said, if we profile our new version of `mapWith`, we'll discover that it is still very slow. Much slower than the built-in `.map` method for arrays. The right tool to discover *why* we're still slow is a memory profiler, but a simple inspection of the program will reveal the following:

Ever time we call `mapWithDelaysWork`, we're calling `[...prepend, fn(first)]`. This creates a new array in memory, populates it with the contents of the `prepend` array, and then copies the value of `fn(first)` into the last position.

We never use `prepend` again, so that array is discarded. We may not be creating 3,000 stack frames, but we are creating an empty array, an array with one element, and array with two elements, an array with three elements, and so on upt to an array with three thousand elements that we eventually use. Although the maximum amount of memory does not grow, the thrashing as we create short-lived arrays is very bad, and we do a lot of work copying elements from one array to another.

If this is so bad, why do many examples of "functional" algrithms work this exact way?

![The IBM 704](images/IBM704.jpg)

### some history

Once upon a time, there was a programming language called [Lisp], an acronym for LISt Processing. Lisp was one of the very first high-level languages. As mentioned previously, the very first implementation was written for the [IBM 704] computer. (The very first FORTRAN implementationw as also written for the 704).

[Lisp]: https://en.wikipedia.org/wiki/Lisp_(programming_language)
[IBM 704]: https://en.wikipedia.org/wiki/IBM_704

The 704 had a 36-bit word, meaning that it was very fast to store and retrive 36-bit values. The CPU's instruction set featured two important macros: `CAR` would fetch 15 bits representing the Contents of the Address part of the Register, while `CDR` would fetch the Contents of the Decrement part of the register. In broad terms, this means that a single 36-bit word could store two separate 15-bit values and it was very fast to save and retrieve pairs of values. If you had two 15-bit values and wished to write them to the register, the `CONS` macro would take the values and write them to a 36-bit word.

Thus, `CONS` put two values together, `CAR` extracted one, and `CDR` extracted the other. Lisp's basic data type is often said to be the list, but in actuality it was the "cons cell," the term used to describe two 15-bit values stored in one word. The 15-bit values were used as pointers that could refer to a location in memory, so in effect, a Cons Cell was a little data strcuture with two pointers to other Cons Cells.

Lists were represented as linked lists of Cons Cells, with each cell's head pointing to an element and the tail pointing to another Cons Cell.

Here's the scheme in JavaScript, using two-element arrays to represent cons cells:

    const cons = (a, d) => [a, d],
          car  = ([a, d]) => a,
          cdr  = ([a, d]) => d;
      
We can make a list by calling `cons` repeatedly, and terminating it with `null`:

    const oneToFive = cons(1, cons(2, cons(3, cons(4, cons(5, null)))));
    
    oneToFive
      //=> [1,[2,[3,[4,[5,null]]]]]

Here's where things get interesting. If we want the head of a list, we call `car` on it:

    car(oneToFive)
      //=> 1
      
`car` is very fast, it simply extracts the first element of the cons cell.

But what about the rest of the list? `cdr` does the trick:

    cdr(oneToFive)
      //=> [2,[3,[4,[5,null]]]]
      
Again, it's just extracting a reference from a cons cell, it's very fast. In Lisp, it's blazingly fast because it happens in hardware. There's no making copies of arrays, the time to `cdr` a list with five elements is the same as the time to `cdr` a list with 5,000 elements, and no temporary arrays are needed.

Thus, an operation like `[first, ...rest] = someArray`  that can be very slow with JavaScript is lightning-fast in Lisp, because `[first, ...rest]` is really a `car` and a `cdr` if we're making lists our of cons cells. Likewise, an operation like `someArray = [something, ...moreElements]` is lightning-fast in Lisp because we're only creating one new cons cell, we aren't duplicating the entire array.

Alas, although lists made out of cons cells are very fast for prepending elements and "shifting" elements off the front, they are slow for iterating over elements because the computer has to "pointer chase" through memory, it's much faster to increment a register and fetch the next item. And it's excruciating to attempt to access an arbitrary item, you have to iterate from the beginning, every time.

So FORTRAN used arrays, and in time Lisp added vectors that work like arrays, and with a new data structure came new algorithms. And so it is today. Although we showed how to map and fold over arrays with `[first, ...rest]`, in reality this is not how it ought to be done. But it is an extremely simple illustration of how recursion works when you have a self-similar means of constructing a data structure.