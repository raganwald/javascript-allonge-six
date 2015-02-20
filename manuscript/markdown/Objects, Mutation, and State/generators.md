## An Introduction to Generators

![Banco do Café](images/banco.jpg)

[Iterables](#iterables) look cool, but then again, everything looks amazing when you’re given cherry-picked examples. What is there they don't do well?

Let's consider how they work. Whether it's a simple functional iterator, or an Iterable exposing a `[Symbol.iterator]` method that in turn exposes an object iterator with a `.next()` method, an iterator is something we call repeatedly until it tells us that it's done.

Iterators are stateful, so the iterator has to arrange its own state such that when you call it, it can compute and return the next item. This seems blindingly obvious and simple. If, for example, you want numbers, you write:

{:lang="js"}
~~~~~~~~
const Numbers = {
  [Symbol.iterator]: () => {
    let n = 0;
    
    return {
      next: () =>
        ({done: false, value: n++})
    }
  }
};
~~~~~~~~

The `Numbers` iterable returns an object that updates a mutable variable, `n`, to deliver number after number. How hard can this be?

Well, we've written our iterator as a *server*. It waits until given a request, and then it returns exactly one item. Then it waits for the next request. There is no concept of pushing numbers out from the iterator, just waiting until a number is pulled out of the iterator by whatever code consumes numbers.

Of course, when we have some code that makes a bunch of something, we don't usually write it like that. We usually just write something like:

{:lang="js"}
~~~~~~~~
let n = 0;

while (true) {
  console.log(n++)
}
~~~~~~~~

And magically, the numbers would pour forth. We would *generate* numbers. Let's put that beside the code for the iterator, minus the iterable scaffolding:

{:lang="js"}
~~~~~~~~
// Iteration
let n = 0;

() =>
  ({done: false, value: n++})
  
// Generation
let n = 0;

while (true) {
  console.log(n++)
}
~~~~~~~~

They're of approximately equal complexity. So why bring up generation? Well, there are some collections that are much easier to generate than to iterate over. Let's look at one:

### recursive iterators

Iterators maintain state, that's what they do. Generators have to manage the exact same amount of state, but sometimes, it's much easier to manage that state in a generator. One of those cases is when we have to recursively enumerate something.

For example, iterating over a tree. Given an array that might contain arrays, let's say we want to generate all the "leaf" elements, i.e. elements that are not, themselves, iterable.

{:lang="js"}
~~~~~~~~
// Generation
const isIterable = (something) =>
  !!something[Symbol.iterator];

const generate = (iterable) => {
  for (let element of iterable) {
    if (isIterable(element)) {
      generate(element)
    }
    else {
      console.log(element)
    }
  }
}

generate([1, [2, [3, 4], 5]])
//=>
  1
  2
  3
  4
  5
~~~~~~~~

Very simple. Now for the iteration version. We'll write a functional iterator to keep things simple, but it's easy to see the shape of the basic problem:

{:lang="js"}
~~~~~~~~
// Iteration
const isIterable = (something) =>
  !!something[Symbol.iterator];

const treeIterator = (iterable) => {
  const iterators = [ iterable[Symbol.iterator]() ];
  
  return () => {
    while (!!iterators[0]) {
      const iterationResult = iterators[0].next();
      
      if (iterationResult.done) {
        iterators.shift();
      }
      else if (isIterable(iterationResult.value)) {
        iterators.unshift(iterationResult.value[Symbol.iterator]());
      }
      else {
        return iterationResult.value;
      }
    }
    return;
  }
}

const i = treeIterator([1, [2, [3, 4], 5]]);
let n;

while (n = i()) {
  console.log(n)
}
//=>
  1
  2
  3
  4
  5
~~~~~~~~

If you peel off `isIterable` and ignore the way that the iteration version uses `[Symbol.iterator]` and `.next`, we're left with the fact that the generator version calls itself recursively, and the iteration version maintains an explicit stack. In essence, both have stacks, but the generator version's stack is *implicit*, while the iterator version's stack is *explicit*.

A less kind way to put it is that the iterator version is greenspunning something built into our programming language: We're reinventing the use of a stack to manage recursion, because writing our code to respond to a function call makes us turn a simple recursive algorithm inside-out.

### state machines

Some iterables can be modelled as state machines. Let's revisit the Fibonacci sequence. Again. One way to define it is:

- The first element of the fibonacci sequence is zero.
- The second element of the fibonacci sequence is one.
- Every subsequent element of the fibonacci sequence is the sum of the previous two elements.

Let's write a generator:

{:lang="js"}
~~~~~~~~
// Generation
const fibonacci = () => {
  let a, b;
  
  console.log(a = 0);
  
  console.log(b = 1);
  
  while (true) {
    [a, b] = [b, a + b];
    console.log(b);
  }
}

fibonacci()
//=>
  0
  1
  1
  2
  3
  5
  8
  13
  21
  34
  55
  89
  144
  ...
~~~~~~~~

The thing to note here is that our `fibonacci` generator has three states: generating `0`, generating `1`, and generating everything after that. This isn't a good fit for an iterator, because iterators have one functional entry point and therefore, we'd have to represent our three states explicitly, perhaps using a [state pattern].

[state pattern]: https://en.wikipedia.org/wiki/State_pattern

We'll keep it simple:

{:lang="js"}
~~~~~~~~
// Iteration
const Numbers = () {
  let a, b, state = 0;
  
  return () => {
    switch (state) {
      case 0:
        state = 1;
        return a = 0;
      case 1:
        state = 2;
        return b = 1;
      case 2:
        [a, b] = [b, a + b];
        return b
    }
  }
}

const i = fibonacci();
let n;

while ((n = i()) < 200) {
  console.log(n);
}
//=>
  0
  1
  1
  2
  3
  5
  8
  13
  21
  34
  55
  89
  144
  ...
~~~~~~~~

Again, this is not particularly horrendous, but like the recursive example, we're explicitly greenspunning the natural linear state. In a generator, we write "do this, then  this, then this." In an iterator, we have to wrap that up and explicitly keep track of what step we're on.

### generators

It would be very nice if we could sometimes write iterators as a `.next()` method that gets called, and sometimes write out a generator. And of course, we both know that this is exactly how JavaScript works. To write a *generator*, we write a function, but we make two changes:

1. We declare the function using the `function*` keyword. Not a fat arrow. Not a plain `function`.
2. We don't `return` values or output them to `console.log`. We "yield" values using the `yield` keyword.

Let's start with the simplest example:

{:lang="js"}
~~~~~~~~
const Numbers = {
  [Symbol.iterator]: function* () {
    let i = 0;
    while (true) {
      yield i++;
    }
  }
}

for (let i of Numbers) {
  console.log(i);
}
//=>
  0
  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  ...
~~~~~~~~

That's it, really. No `.next()` method, just a `function*` that yields values. Let's do it again and show how a generator makes our linear state work:

{:lang="js"}
~~~~~~~~
const Fibonacci = {
  [Symbol.iterator]: function* () {
    let a, b;
    
    yield a = 0;
    
    yield b = 1;
    
    while (true) {
      [a, b] = [b, a + b]
      yield b;
    }
  }
}

for (let i of Fibonacci) {
  console.log(i);
  if (i > 200) break;
}
//=>
  0
  1
  1
  2
  3
  5
  8
  13
  21
  34
  55
  89
  144
  ...
~~~~~~~~

We're writing an iterator, but we're using a generator to do it. *Generators* are iterators, just with a different syntax. And this new syntax allows us to use JavaScript's natural management of state instead of constantly rolling our own.

A generator for iterating over trees is ridiculously simple:

{:lang="js"}
~~~~~~~~
const isIterable = (something) =>
  !!something[Symbol.iterator];
  
const TreeIterable = (iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let e of iterable) {
        if (isIterable(e)) {
          for (let ee of TreeIterable(e)) {
            yield ee;
          }
        }
        else {
          yield e;
        }
      }
    }
  })

for (let i of TreeIterator([1, [2, [3, 4], 5]])) {
  console.log(i);
}
//=>
  1
  2
  3
  4
  5
~~~~~~~~

We've gone with the full iterable here, a `TreeIterable(iterable)` returns an iterable that treats `iterable` as a tree. We take advantage of the `for...of` loop in a plain and direct way: For each element `e`, if it is iterable, treat it as a tree and iterate over it, yielding each of its elements. If `e` is not an iterable, yield `e`.

JavaScript handles the recursion for us using its own execution stack. This is clearly simpler than trying to maintain our own stack and remembering whether we are shifting and unshifting, or pushing and popping. And while this version has the extra scaffolding to make it a first-class iterable, it matches our simple generation code from above more-or-less directly.

### generators *are* iterators

Recall that an *iterable* is an object with a `[Symbol.iterator]` method. When you call that, you get an *iterator*, an object with a `.next()` method. But now we're making iterators that return a generator. What gives?

Well, generators *are* iterators. Let's prove it. Here is our `Number` iterable and a few of lazy iterable operations. We'll generate an infinite iterable of strings that represent the squares of numbers, and we'll find the palindrome that has at least four digits:

{:lang="js"}
~~~~~~~~
const firstIterable = (iterable) =>
  iterable[Symbol.iterator]().next().value;
  
const filterIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      return {
        next: () => {
          do {
            const {done, value} = iterator.next();
          } while (!done && !fn(value));
          return {done, value};
        }
      }
    }
  });

const mapIterableWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      return {
        next: () => {
          const {done, value} = iterator.next();
    
          return ({done, value: done ? undefined : fn(value)});
        }
      }
    }
  });
  
const Numbers = {
  [Symbol.iterator]: function* () {
    let i = 0;
    while (true) {
      yield i++;
    }
  }
}

const Squares = mapIterableWith((n) => n * n, Numbers);

const SquaresStrings = mapIterableWith((n) => `${n}`, Squares);

const Palindromes = filterIterableWith((s) => s === reverse(s), SquaresStrings)

const WithAtLeastFourDigits = filterIterableWith((s) => s.length > 3, Palindromes)

firstIterable(WithAtLeastFourDigits)
  //=> 10201
~~~~~~~~

As you can see, our operations all work on objects that have a `.next()` method, therefore we know that writing an iterator using `function*` and `yield` produces an object with a `.next()` method.

But JavaScript handles all of the state we need to save to write our code as a generator, even if other code will call it just like a normal object iterator. JavaScript also allows us to return values and not worry abut wrapping them in an object with `.done` and `.value` properties.

### Summary

A generator is a function that is defined with `function*` and uses `yield` to generate values. Using a generator instead of writing an object that has a `.next()` method allows us to write code that can be much simpler for cases like recursive iterations or state patterns. And we don't need to worry about wrapping our values in an object with `.done` and `.value` properties.