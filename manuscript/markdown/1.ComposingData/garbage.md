## Garbage, Garbage Everywhere {#garbage}

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

![The IBM 704](images/IBM704b.jpg)

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

We'll look at linked lists again when we look at [Plain Old JavaScript Objects](#pojos).

### so why arrays

If `[first, ...rest]` is so slow, why does JavaScript use arrays instead of making everything a linked list?

Well, linked lists are fast for a few things, like taking the front element off a list, and taking the remainder of a list. But not for iterating over a list: Pointer chasing through memory is quite a bit slower than incrementing an index. In addition to the extra fetches to dereference pointers, pointer chasing suffers from cache misses. And if you want an arbitrary item from a list, you have to iterate through the list element by element, whereas with the indexed array you just fetch it.

We have avoided discussing rebinding and mutating values, but if we want to change elements of our lists, the naïve linked list implementation suffers as well: When we take the `cdr` of a linked list, we are sharing the elements. If we make any change other than cons-ing a new element to the front, we are changing both the new list and the old list.

Arrays avoid this problem by pessimistically copying all the references whenever we extract an element or sequence of elements from them (We'll see this explained later in [Mutation](#mutation)).

For these and other reasons, almost all languages today make it possible to use a fast array or vector type that is optimized for iteration, and even Lisp now has a variety of data structures that are optimized for specific use cases.

### summary

Although we showed how to use tail calls to map and fold over arrays with `[first, ...rest]`, in reality this is not how it ought to be done. But it is an extremely simple illustration of how recursion works when you have a self-similar means of constructing a data structure.