## Functional Iterators {#functional-iterators}

Let's consider a remarkably simple problem: Finding the sum of the elements of an array. In tail-recursive style, it looks like this:

{:lang="js"}
~~~~~~~~
const arraySum = ([first, ...rest], accumulator = 0) =>
  first === undefined
    ? accumulator
    : arraySum(rest, first + accumulator)

arraySum([1, 4, 9, 16, 25])
  //=> 55
~~~~~~~~

As we saw earlier, this entangles the mechanism of traversing the array with the business of summing the bits. So we can separate them using `fold`:

{:lang="js"}
~~~~~~~~
const callLeft = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...args, ...remainingArgs);

const foldArrayWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : fn(first, foldArrayWith(fn, terminalValue, rest));

const arraySum = callLeft(foldArrayWith, (a, b) => a + b, 0);

arraySum([1, 4, 9, 16, 25])
  //=> 55
~~~~~~~~

The nice thing about this is that the definition for `arraySum` mostly concerns itself with summing, and not with traversing over a collection of data. But it still relies on `foldArrayWith`, so it can only sum arrays.

What happens when we want to sum a tree of numbers? Or a linked list of numbers?

Well, we call `arraySum` with an array, and it has baked into it a method for traversing the array. Perhaps we could extract both of those things. Let's rearrange our code a bit:

{:lang="js"}
~~~~~~~~
const callRight = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const foldArrayWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : fn(first, foldArrayWith(fn, terminalValue, rest));

const foldArray = (array) => callRight(foldArrayWith, array);

const sumFoldable = (folder) => folder((a, b) => a + b, 0);

sumFoldable(foldArray([1, 4, 9, 16, 25]))
  //=> 55
~~~~~~~~

What we've done is turn an array into a function that folds an array with `const foldArray = (array) => callRight(foldArrayWith, array);`. The `sumFoldable` function doesn't care what kind of data structure we have, as long as it's foldable.

Here it is summing a tree of numbers:

{:lang="js"}
~~~~~~~~
const callRight = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const foldTreeWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : Array.isArray(first)
      ? fn(foldTreeWith(fn, terminalValue, first), foldTreeWith(fn, terminalValue, rest))
      : fn(first, foldTreeWith(fn, terminalValue, rest));

const foldTree = (tree) => callRight(foldTreeWith, tree);

const sumFoldable = (folder) => folder((a, b) => a + b, 0);

sumFoldable(foldTree([1, [4, [9, 16]], 25]))
  //=> 55
~~~~~~~~

We've found another way to express the principle of separating traversing a data structure from the operation we want to perform on that data structure, we've completely separated the knowledge of how to sum from the knowledge of how to fold an array or tree (or anything else, really).

### iterating

Folding is a universal operation, and with care we can accomplish any task with folds that could be accomplished with that stalwart of structured programming, the `for` loop. Nevertheless, there is some value in being able to express some algorithms as iteration.

JavaScript has a particularly low-level version of `for` loop that mimics the semantics of the `C` language. Summing the elements of an array can be accomplished with:

{:lang="js"}
~~~~~~~~
const arraySum = (array) => {
  let sum = 0;

  for (let i = 0; i < array.length; ++i) {
    sum += array[i];
  }
  return sum
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
~~~~~~~~

Once again, we're mixing the code for iterating over an array with the code for calculating a sum. And worst of all, we're getting really low-level with details like knowing that the elements of an array are indexed with consecutive integers that begin with `0`.

We can write this a slightly different way, using a `while` loop:

{:lang="js"}
~~~~~~~~
const arraySum = (array) => {
  let done,
      sum = 0,
      i = 0;

  while ((done = i == array.length, !done)) {
    const value = array[i++];
    sum += value;
  }
  return sum
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
~~~~~~~~

Notice that buried inside our loop, we have bound the names `done` and `value`. We can put those into a POJO (a [Plain Old JavaScript Object](#pojos)). It'll be a little awkward, but we'll be patient:

{:lang="js"}
~~~~~~~~
const arraySum = (array) => {
  let iter,
      sum = 0,
      index = 0;

  while (
    (eachIteration = {
        done: index === array.length,
        value: index < array.length ? array[index] : undefined
      },
      ++index,
      !eachIteration.done)
    ) {
    sum += eachIteration.value;
  }
  return sum;
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
~~~~~~~~

With this code, we make a POJO that has `done` and `value` keys. All the summing code needs to know is to add `eachIteration.value`. Now we can extract the ickiness into a separate function:

{:lang="js"}
~~~~~~~~
const arrayIterator = (array) => {
  let i = 0;

  return () => {
    const done = i === array.length;

    return {
      done,
      value: done ? undefined : array[i++]
    }
  }
}

const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;

  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum;
}

iteratorSum(arrayIterator([1, 4, 9, 16, 25]))
  //=> 55
~~~~~~~~

Now this is something else. The `arrayIterator` function takes an array and returns a function we can call repeatedly to obtain the elements of the array. The `iteratorSum` function iterates over the elements by calling the `iterator` function repeatedly until it returns `{ done: true }`.

We can write a different iterator for a different data structure. Here's one for linked lists:

{:lang="js"}
~~~~~~~~
const EMPTY = null;

const isEmpty = (node) => node === EMPTY;

const pair = (first, rest = EMPTY) => ({first, rest});

const list = (...elements) => {
  const [first, ...rest] = elements;

  return elements.length === 0
    ? EMPTY
    : pair(first, list(...rest))
}

const print = (aPair) =>
  isEmpty(aPair)
    ? ""
    : `${aPair.first} ${print(aPair.rest)}`

const listIterator = (aPair) =>
  () => {
    const done = isEmpty(aPair);
    if (done) {
      return {done};
    }
    else {
      const {first, rest} = aPair;

      aPair = aPair.rest;
      return { done, value: first }
    }
  }

const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;;

  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

const aListIterator = listIterator(list(1, 4, 9, 16, 25));

iteratorSum(aListIterator)
  //=> 55
~~~~~~~~

### unfolding and laziness

Iterators are functions. When they iterate over an array or linked list, they are traversing something that is already there. But they could just as easily manufacture the data as they go. Let's consider the simplest example:

{:lang="js"}
~~~~~~~~
const NumberIterator = (number = 0) =>
  () => ({ done: false, value: number++ })

fromOne = NumberIterator(1);

fromOne().value;
  //=> 1
fromOne().value;
  //=> 2
fromOne().value;
  //=> 3
fromOne().value;
  //=> 4
fromOne().value;
  //=> 5
~~~~~~~~

And here's another one:

{:lang="js"}
~~~~~~~~
const FibonacciIterator  = () => {
  let previous = 0,
      current = 1;

  return () => {
    const value = current;

    [previous, current] = [current, current + previous];
    return {done: false, value};
  };
};

const fib = FibonacciIterator()

fib().value
  //=> 1
fib().value
  //=> 1
fib().value
  //=> 2
fib().value
  //=> 3
fib().value
  //=> 5
~~~~~~~~

A function that starts with a seed and expands it into a data structure is called an *unfold*. It's the opposite of a fold. It's possible to write a generic unfold mechanism, but let's pass on to what we can do with unfolded iterators.

For starters, we can `map` an iterator, just like we map a collection:

{:lang="js"}
~~~~~~~~
const mapIteratorWith = (fn, iterator) =>
  () => {
    const {done, value} = iterator();

    return ({done, value: done ? undefined : fn(value)});
  }

const squares = mapIteratorWith((x) => x * x, NumberIterator(1));

squares().value
  //=> 1
squares().value
  //=> 4
squares().value
  //=> 9
~~~~~~~~

This business of going on forever has some drawbacks. Let's introduce an idea: A function that takes an iterator and returns another iterator. We can start with `take`, an easy function that returns an iterator that only returns a fixed number of elements:

{:lang="js"}
~~~~~~~~
const take = (iterator, numberToTake) => {
  let count = 0;

  return () => {
    if (++count <= numberToTake) {
      return iterator();
    } else {
      return {done: true};
    }
  };
};

const toArray = (iterator) => {
  let eachIteration,
      array = [];

  while ((eachIteration = iterator(), !eachIteration.done)) {
    array.push(eachIteration.value);
  }
  return array;
}

toArray(take(FibonacciIterator(), 5))
  //=> [1, 1, 2, 3, 5]

toArray(take(squares, 5))
  //=> [1, 4, 9, 16, 25]
~~~~~~~~

How about the squares of the first five odd numbers? We'll need an iterator that produces odd numbers. We can write that directly:

{:lang="js"}
~~~~~~~~
const odds = () => {
  let number = 1;

  return () => {
    const value = number;

    number += 2;
    return {done: false, value};
  }
}

const squareOf = callLeft(mapIteratorWith, (x) => x * x)

toArray(take(squareOf(odds()), 5))
  //=> [1, 9, 25, 49, 81]
~~~~~~~~

We could also write a filter for iterators to accompany our mapping function:

{:lang="js"}
~~~~~~~~
const filterIteratorWith = (fn, iterator) =>
  () => {
    do {
      const {done, value} = iterator();
    } while (!done && !fn(value));
    return {done, value};
  }

const oddsOf = callLeft(filterIteratorWith, (n) => n % 2 === 1);

toArray(take(squareOf(oddsOf(NumberIterator(1))), 5))
  //=> [1, 9, 25, 49, 81]
~~~~~~~~

Mapping and filtering iterators allows us to compose the parts we already have, rather than writing a tricky bit of code with ifs and whiles and boundary conditions.

### bonus

Many programmers coming to JavaScript from other languages are familiar with three "canonical" operations on collections: folding, filtering, and finding. In Smalltalk, for example, they are known as `collect`, `select`, and `detect`.

We haven't written anything that finds the first element of an iteration that meets a certain criteria. Or have we?

{:lang="js"}
~~~~~~~~
const firstInIteration = (fn, iterator) =>
  take(filterIteratorWith(fn, iterator), 1);
~~~~~~~~

This is interesting, because it is lazy: It doesn't apply `fn` to every element in an iteration, just enough to find the first that passes the test. Whereas if we wrote something like:

{:lang="js"}
~~~~~~~~
const firstInArray = (fn, array) =>
  array.filter(fn)[0];
~~~~~~~~

JavaScript would apply `fn` to every element. If `array` was very large, and `fn` very slow, this would consume a lot of unnecessary time. And if `fn` had some sort of side-effect, the program could be buggy.

### caveat

Please note that unlike most of the other functions discussed in this book, iterators are *stateful*. There are some important implications of stateful functions. One is that while functions like `take(...)` appear to create an entirely new iterator, in reality they return a decorated reference to the original iterator. So as you traverse the new decorator, you're changing the state of the original!

For all intents and purposes, once you pass an iterator to a function, you can expect that you no longer "own" that iterator, and that its state either has changed or will change.
