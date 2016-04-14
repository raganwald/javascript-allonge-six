## Self-Iterables {#self-iterables}

In [Generating Iterables](#generating-iterables), we saw that some iterables are best represented with a generator function, rather than directly coding an object with a `.next()` method. One simple example given was an iterable over the fibonacci sequence. The iterator implementation was:

{:lang="js"}
~~~~~~~~
const Fibonacci = {
  [Symbol.iterator]: () => {
    let a = 0, b = 1, state = 0;
    
    return {
      next: () => {
        switch (state) {
          case 0:
            state = 1;
            return {value: a};
          case 1:
            state = 2;
            return {value: b};
          case 2:
            [a, b] = [b, a + b];
            return {value: b};
        }
      }
    }
  }
};
~~~~~~~~

While the generator version was:

{:lang="js"}
~~~~~~~~
const Fibonacci = {
  [Symbol.iterator]: function* () {
    let a = 0, b = 1;
    
    yield a;
    
    yield b;
    
    while (true) {
      [a, b] = [b, a + b]
      yield b;
    }
  }
};
~~~~~~~~

The generator function version is much simpler, because it uses JavaScript's natural control flow to model its state implicitly, rather than explicitly managing state. Generator functions are handy for making iterable like this.

So far, all we have ever done with these generator functions is stick them inside iterables, like this:

{:lang="js"}
~~~~~~~~
const someIterable = {
  [Symbol.iterator]: function* () {
    // yield stuff
  }
}
~~~~~~~~

But what else can we do with a generator? Well, we can invoke a generator function directly. For example, instead of working with an iterable collection:

{:lang="js"}
~~~~~~~~
const EvenNumbers = {
  [Symbol.iterator]: function* () {
    let i = 0;
    while (true) {
      i += 2;
      yield i;
    }
  }
}

for (let n of EvenNumbers) {
  console.log(n)
}
  //=>
    2
    4
    6
    8
    10
    ...
~~~~~~~~

We could write:

{:lang="js"}
~~~~~~~~
const evenNumberGenerator = function* () {
  let i = 0;
  while (true) {
    i += 2;
    yield i;
  }
};

for (let n of evenNumberGenerator()) {
  console.log(n)
}
  //=>
    2
    4
    6
    8
    10
    ...
~~~~~~~~

Notice that `evenNumberGenerator` isn't an iterable. We *invoke* `evenNumberGenerator` and get an iterable back. How does *that* work? How do generator functions return iterables?

### how do generators return iterables?

Let's back up to our iterator solution for `Fibonacci`. We'll extract the function, then use it like a generator:

{:lang="js"}
~~~~~~~~
const wrongFibonacciIterator = () => {
  let a = 0, b = 1, state = 0;
  
  return {
    next: () => {
      switch (state) {
        case 0:
          state = 1;
          return {value: a};
        case 1:
          state = 2;
          return {value: b};
        case 2:
          [a, b] = [b, a + b];
          return {value: b};
      }
    }
  }
};

for (let n of wrongFibonacciIterator()) {
  console.log(n)
}
  //=> undefined is not a function (evaluating 'iterable[Symbol.iterator]()')
~~~~~~~~

So. When we invoke a generator function, it returns something that has a `[Symbol.iterator]` method, but when we invoke our hand-written iterator method, it return something that doesn't have a `[Symbol.iterator]` method.

That is kind of obvious, we wrote `fibonacciIterator` ourselves, it returns an *iterator*, an object that only has a `next()` method. Let's give it a `[Symbol.iterator]` method. What should that return? Well, an iterator if it's to work like an iterable.

So, we have a `[Symbol.iterator]` method that returns an iterator object. And we want the iterator object to have a `[Symbol.iterator]` method that returns an iterator object. What's the right iteator object to return?

Itself. Like this:

{:lang="js"}
~~~~~~~~
const fibonacciIterator = () => {
  let a = 0, b = 1, state = 0;
  
  return {
    [Symbol.iterator]: function () {
      return this;
    },
    next: () => {
      switch (state) {
        case 0:
          state = 1;
          return {value: a};
        case 1:
          state = 2;
          return {value: b};
        case 2:
          [a, b] = [b, a + b];
          return {value: b};
      }
    }
  }
};

for (let n of fibonacciIterator()) {
  console.log(n)
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
    ...
~~~~~~~~

Now, we have a function that returns an iterator that is also an iterable, in that it returns itself. We aren't using the `function*` and `yield` syntax, but it behaves just like a generator function. But why does the iterator's `[Symbol.iterator]` method return itself? Why not write something like `[Symbol.iterator]: () => fibonacciIterator()` so that it returns a brand new fibonacci iterator?

### returning `this`

Earlier, we wrote `takeFromCollection`. Here's a simpler version written as a generator function:

{:lang="js"}
~~~~~~~~
const takeFrom = function* (numberToTake, iterable) {
  let remaining = numberToTake;
  
  for (let value of iterable) {
    if (remaining-- <= 0) break;
    yield value;
  }
}
~~~~~~~~

It takes an iterable and iterates over its elements with `for (let value of iterable)`. Let's consider this expression:

{:lang="js"}
~~~~~~~~
const fib = fibonacciIterator();
  
[
  [...takeFrom(5, fib)],
  [...takeFrom(5, fib)],
  [...takeFrom(5, fib)],
  [...takeFrom(5, fib)],
  [...takeFrom(5, fib)]
]
  //=>
    [
      [0,1,1,2,3],
      [8,13,21,34,55],
      [144,233,377,610,987],
      [2584,4181,6765,10946,17711],
      [46368,75025,121393,196418,317811]
    ]
~~~~~~~~

`fib` self-iterable. It's stateful. When we first evaluate `takeFrom(5, fib)`, we get the first five elements from it. The next time we evaluate `takeFrom(5, fib)`, we treat `fib` as an iterable, get its iterator with `[Symbol.iterator]`, and get itself back. But it has already given the first five elements, so now we get the *next* five elements.

But what if the iterator's `[Symbol.iterator]` method returns a brand-new iterator?

{:lang="js"}
~~~~~~~~
const badFibonacciIterator = () => {
  let a = 0, b = 1, state = 0;
  
  return {
    [Symbol.iterator]: badFibonacciIterator,
    next: () => {
      switch (state) {
        case 0:
          state = 1;
          return {value: a};
        case 1:
          state = 2;
          return {value: b};
        case 2:
          [a, b] = [b, a + b];
          return {value: b};
      }
    }
  }
};

const badFib = badFibonacciIterator();
  
[
  [...takeFromCollection(5, badFib)],
  [...takeFromCollection(5, badFib)],
  [...takeFromCollection(5, badFib)],
  [...takeFromCollection(5, badFib)],
  [...takeFromCollection(5, badFib)]
]
//=>
  [
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3]
  ]
~~~~~~~~

When we call `badFibonacciIterator()`, we get an iterator that is also an iterable, but when you call `[Symbol.iterator]`, you are calling `badFibonacciIterator()` all over again, and getting a brand-new iterator that has brand-new state all over again, so our five calls to `[...takeFromCollection(5, badFib)]` all return the first five elements of the sequence.

When an iterator is also an iterable that returns itself, we say that it is a *self-iterable*. This is, as we see, quite useful for taking advantage of JavaScript's support for iterables while maintaining state.

Let's compare our hand-written self-iterable to a generator function:

{:lang="js"}
~~~~~~~~
const fibonacciGenerator = function* () {
  let a = 0, b = 1;
  
  yield a;
  
  yield b;
  
  while (true) {
    [a, b] = [b, a + b]
    yield b;
  }
};

const generatedFib = fibonacciGenerator();

[
  [...takeFromCollection(5, generatedFib)],
  [...takeFromCollection(5, generatedFib)],
  [...takeFromCollection(5, generatedFib)],
  [...takeFromCollection(5, generatedFib)],
  [...takeFromCollection(5, generatedFib)]
]
//=>
  [
    [0,1,1,2,3],
    [8,13,21,34,55],
    [144,233,377,610,987],
    [2584,4181,6765,10946,17711],
    [46368,75025,121393,196418,317811]
  ]
~~~~~~~~

Thus, we see that *generator functions return self-iterables*.

### when iterables? and when self-iterables?

Let's revisit one of our convenience functions, `takeFromCollection`. Here it a simple version without any of the collection methods, and returns a self-iterable:

{:lang="js"}
~~~~~~~~
const takeFromCollection = (numberToTake, iterable) =>
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      let remainingElements = numberToTake;
    
      return {
        [Symbol.iterator]: function () { return this; },
        next: () => {
          let {done, value} = iterator.next();
        
          done = done || remainingElements-- <= 0;
  
          return ({done, value: done ? undefined : value});
        }
      }
    }
  });
~~~~~~~~

We can see that when we call `takeFromCollection`, we get an iterable back. When we call that iterable's `[Symbol.iterator]` method, we get a brand-new iterator. Every time. We can test this:

{:lang="js"}
~~~~~~~~
const Fibonacci = {
  [Symbol.iterator]: function* () {
    let a = 0, b = 1;
  
    yield a;
  
    yield b;
  
    while (true) {
      [a, b] = [b, a + b]
      yield b;
    }
  }
};

const FirstFiveFibs = takeFromCollection(5, Fibonacci);

[
  [...FirstFiveFibs],
  [...FirstFiveFibs],
  [...FirstFiveFibs],
  [...FirstFiveFibs],
  [...FirstFiveFibs]
]
//=>
  [
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3],
    [0,1,1,2,3]
  ]
~~~~~~~~

We keep getting the first five fibnacci numbers when we gather `FirstFiveFibs` into arrays, because the way `takeFromCollection` is written, it returns an iterable that generates a brand new iterator every time you call `[Symbol.iterator]`.

This is exactly the behaviour we desire if we want `FirstFiveFibs` to behave like an iterable collection. Every time we iterate over it, we want to iterate over the whole thing. It's the same thing with other operations, like `flterWith`:

{:lang="js"}
~~~~~~~~
const filterCollectionWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        if (fn(element)) {
        yield element;
        }
      }
    }
  });
  
const Numbers = {
  [Symbol.iterator]: function* () {
    for (let i = 0; ++i; true) {
      yield i;
    }
  }
};

const EvenNumbers = filterCollectionWith((x) => x % 2 === 0, Numbers);
~~~~~~~~

`Numbers` and `EvenNumbers` are used as collections of numbers. If we want the first ten even numbers, can just `takeFromCollection(10, EvenNumbers)` and always get the first ten. Things that are intended to behave like collections should be proper iterables, so that when we iterate over them, we always get a fresh iterator, rather than continuing where we left off.

However, we don't always want to do that. Sometimes we want a stateful iterator. Maybe we want to make a five-by-five matrix of numbers. In that case, we want a function that returns a self-iterable:

{:lang="js"}
~~~~~~~~
const numbers = function* () {
  for (let i = 0; ++i; true) {
    yield i;
  }
};

const Numbers = {
  [Symbol.iterator]: numbers
};

const fiveNumbers = takeFromCollection(5, numbers());

[
  [...fiveNumbers],
  [...fiveNumbers],
  [...fiveNumbers],
  [...fiveNumbers],
  [...fiveNumbers]
]
  //=>
    [
      [1,2,3,4,5],
      [7,8,9,10,11],
      [13,14,15,16,17],
      [19,20,21,22,23],
      [25,26,27,28,29]
    ]
~~~~~~~~

Now, `takeFromCollection` returns a single iterable. And that returns a new iterator every time you call ``