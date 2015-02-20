## Collections, Iteration, and Laziness {#iterables}

![Coffee Labels at the Saltspring Coffee Processing Facility](images/saltspring/coffee-labels.jpg)

Many objects in JavaScript can model collections of things. A collection is like a box containing stuff. Sometimes you just want to move the box around. But sometimes you want to open it up and do things with its contents.

Things like "put a label on every bag of coffee in this box," Or, "Open the box, take out the bags of decaf, and make a new box with just the decaf." Or, "go through the bags in this box, and take out the first one marked 'Espresso' that contains at least 454 grams of beans."

All of these actions involve going through the contents one by one. Acting on the elements of a collection one at a time is called *iterating over the contents*, and JavaScript has a standard way to iterate over the contents of collections.

### a look back at functional iterators

When discussing functions, we looked at the benefits of writing [Functional Iterators](#functional-iterators). We can do the same thing for objects. Here's a stack that has its own functional iterator method:

{:lang="js"}
~~~~~~~~
const Stack1 = () =>
  ({
    array:[],
    index: -1,
    push: function (value) {
      return this.array[this.index += 1] = value;
    },
    pop: function () {
      const value = this.array[this.index];
    
      this.array[this.index] = undefined;
      if (this.index >= 0) { 
        this.index -= 1 
      }
      return value
    },
    isEmpty: function () {
      return this.index < 0
    },
    iterator: function () {
      let iterationIndex = this.index;
      
      return () => {
        if (iterationIndex > this.index) {
          iterationIndex = this.index;
        }
        if (iterationIndex < 0) {
          return {done: true};
        }
        else {
          return {done: false, value: this.array[iterationIndex--]}
        }
      }
    }
  });

const stack = Stack1();

stack.push("Greetings");
stack.push("to");
stack.push("you!")

const iter = stack.iterator();
iter().value
  //=> "you!"
iter().value
  //=> "to"
~~~~~~~~

The way we've written `.iterator` as a method, each object knows how to return an iterator for itself.

A> Note that the `.iterator()` method is implemented with the `function` keyword, so when we invoke it with `stack.iterator()`, JavaScript sets `this` to the value of `stack`. But what about the function `.iterator()` returns? It is defined with a fat arrow `() => { ... }`. What is the value of `this` within that function?
A>
A> Since JavaScript doesn't bind `this` within a fat arrow function, we follow the same rules of variable scoping as any other variable name: We check in the environment enclosing the function. Although the `.iterator()` method has returned, its environment is the one that encloses our `() => { ... }` function, and that's where `this` is bound to the value of `stack`.
A>
A> Therefore, the iterator function returned by the `.iterator()` method has `this` bound to the `stack` object, even though we call it with `iter()`.

And here's a `sum` function implemented as a fold over a functional iterator:

{:lang="js"}
~~~~~~~~
const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;
  
  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}
~~~~~~~~

We can use it with our stack:

{:lang="js"}
~~~~~~~~
const stack = Stack1();

stack.push(1);
stack.push(2);
stack.push(3);

iteratorSum(stack.iterator())
  //=> 6
~~~~~~~~

We could save a step and write `collectionSum`, a function that folds over any object, provided that the object implements an `.iterator` method:

{:lang="js"}
~~~~~~~~
const collectionSum = (collection) => {
  const iterator = collection.iterator();
  
  let eachIteration,
      sum = 0;
  
  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

collectionSum(stack)
  //=> 6
~~~~~~~~

If we write a program with the presumption that "everything is an object," we can write maps, folds, and filters that work on objects. We just ask the object for an iterator, and work on the iterator. Our functions don't need to know anything about how an object implements iteration, and we get the benefit of lazily traversing our objects.

This is a good thing.

### iterator objects

Iteration for functions and objects has been around for many, many decades. For simple linear collections like arrays, linked lists, stacks, and queues, functional iterators are the simplest and easiest way to implement iterators.

In programs involving large collections of objects, it can be handy to implement iterators as objects, rather than functions. The mechanics of iterating can then be factored using the same tools that are used to factor the mechanics of all other objects in the system.

Fortunately, an iterator object is almost as simple as an iterator function. Instead of having a function that you call to get the next element, you have an object with a `.next()` method.

Like this:

{:lang="js"}
~~~~~~~~
const Stack2 = ()) =>
  ({
    array: [],
    index: -1,
    push: function (value) {
      return this.array[this.index += 1] = value;
    },
    pop: function () {
      const value = this.array[this.index];
    
      this.array[this.index] = undefined;
      if (this.index >= 0) { 
        this.index -= 1 
      }
      return value
    },
    isEmpty: function () {
      return this.index < 0
    },
    iterator: function () {
      let iterationIndex = this.index;
      
      return {
        next: () => {
          if (iterationIndex > this.index) {
            iterationIndex = this.index;
          }
          if (iterationIndex < 0) {
            return {done: true};
          }
          else {
            return {done: false, value: this.array[iterationIndex--]}
          }
        }
      }
    }
  });

const stack = Stack2();

stack.push(2000);
stack.push(10);
stack.push(5)

const collectionSum = (collection) => {
  const iterator = collection.iterator();
  
  let eachIteration,
      sum = 0;
  
  while ((eachIteration = iterator.next(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

collectionSum(stack)
  //=> 2015
~~~~~~~~

Now our `.iterator()` method is returning an iterator object. When working with objects, we do things the object way. But having started by building functional iterators, we understand what is happening underneath the object's scaffolding.

### iterables

People have been writing iterators since JavaScript was first released in the late 1990s. Since there was no particular standard way to do it, people used all sorts of methods, and their methods returned all sorts of things: Objects with various interfaces, functional iterators, you name it.

So, when a standard way to write iterators was added to the JavaScript language, it didn't make sense to use a method like `.iterator()` for it: That would conflict with existing code. Instead, the language encourages new code to be written with a different name for the method that a collection object uses to return its iterator.

To ensure that the method would not conflict with any existing code, JavaScript provides a *symbol*. Symbols are unique constants that are guaranteed not to conflict with existing strings. Symbols are a longstanding technique in programming going back to Lisp, where the `GENSYM` function generated... You guessed it... Symbols.[^symbol]

[^symbol]: You can read more about JavaScript symbols in Axel Rauschmayer's [Symbols in ECMAScript 6](http://www.2ality.com/2014/12/es6-symbols.html).

The expression `Symbol.iterator` evaluates to a special symbol representing the name of the method that objects should use if they return an iterator object.

Our stack does, so instead of binding the existing iterator method to the name `iterator`, we bind it to the `Symbol.iterator`. We'll do that using the `[` `]` syntax for using an expression as an object literal key:

{:lang="js"}
~~~~~~~~
const Stack3 = () =>
  ({
    array: [],
    index: -1,
    push: function (value) {
      return this.array[this.index += 1] = value;
    },
    pop: function () {
      const value = this.array[this.index];
    
      this.array[this.index] = undefined;
      if (this.index >= 0) { 
        this.index -= 1 
      }
      return value
    },
    isEmpty: function () {
      return this.index < 0
    },
    [Symbol.iterator]: function () {
      let iterationIndex = this.index;
      
      return {
        next: () => {
          if (iterationIndex > this.index) {
            iterationIndex = this.index;
          }
          if (iterationIndex < 0) {
            return {done: true};
          }
          else {
            return {done: false, value: this.array[iterationIndex--]}
          }
        }
      }
    }
  });

const stack = Stack3();

stack.push(2000);
stack.push(10);
stack.push(5)

const collectionSum = (collection) => {
  const iterator = collection[Symbol.iterator]();
  
  let eachIteration,
      sum = 0;
  
  while ((eachIteration = iterator.next(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

collectionSum(stack)
  //=> 2015
~~~~~~~~

Using `[Symbol.iterator]` instead of `.iterator` seems like adding an extra moving part for nothing. Do we get anything in return?

Indeed we do. Behold the `for...of` loop:

{:lang="js"}
~~~~~~~~
const iterableSum = (iterable) => {
  let sum = 0;
  
  for (let num of iterable) {
    sum += num;
  }
  return sum
}

iterableSum(stack)
  //=> 2015
~~~~~~~~

The `for...of` loop works directly with any object that is *iterable*, meaning it works with any object that has a `Symbol.iterator` method that returns a object iterator. Here's another linked list, this one is iterable:

{:lang="js"}
~~~~~~~~
const EMPTY = {
  isEmpty: () => true
};

const isEmpty = (node) => node === EMPTY;

const Pair1 = (first, rest = EMPTY) =>
  ({
    first,
    rest,
    isEmpty: () => false,
    [Symbol.iterator]: function () {
      let currentPair = this;
      
      return {
        next: () => {
          if (currentPair.isEmpty()) {
            return {done: true}
          }
          else {
            const value = currentPair.first;
            
            currentPair = currentPair.rest;
            return {done: false, value}
          }
        }
      }
    }
  });

const list = (...elements) => {
  const [first, ...rest] = elements;
  
  return elements.length === 0
    ? EMPTY
    : Pair1(first, list(...rest))
}

const someSquares = list(1, 4, 9, 16, 25);
    
iterableSum(someSquares)
  //=> 55
~~~~~~~~

As we can see, we can use `for...of` with linked lists just as easily as with stacks. And there's one more thing: You recall that the spread operator (`...`) can spread the elements of an array in an array literal or as parameters in a function invocation.

Now is the time to note that we can spread any iterable. So we can spread the elements of an iterable into an array literal:

{:lang="js"}
~~~~~~~~
['some squares', ...someSquares]
  //=> ["some squares", 1, 4, 9, 16, 25]
~~~~~~~~

And we can also spread the elements of an array literal into parameters:

{:lang="js"}
~~~~~~~~
const firstAndSecondElement = (first, second) =>
  ({first, second})
  
firstAndSecondElement(...stack)
  //=> {"first":5,"second":10}
~~~~~~~~

This can be extremely useful.

One caveat of spreading iterables: JavaScript creates an array out of the elements of the iterable. That might be very wasteful for extremely large collections. For example, if we spread a large collection just to find an element in the collection, it might have been wiser to iterate over the element using its iterator directly.

And if we have an infinite collection, spreading is going to fail outright.

### iterables out to infinity

Iterables needn't represent finite collections:

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
}
~~~~~~~~

There are useful things we can do with iterables representing an infinite number of elements. Before we point out something we can do with them, let's point out what we can't do with them:

{:lang="js"}
~~~~~~~~
['all the numbers', ...Numbers]
  //=> infinite loop!
  
firstAndSecondElement(...Numbers)
  //=> infinite loop!
~~~~~~~~

Attempting to spread an infinite iterable into an array is always going to fail.

We can look at useful things to do with both infinite and finite iterables. But first, let's define some operations on iterables. Here's `mapIterableWith`, it takes any iterable and returns an iterable representing a mapping over the original iterable:

{:lang="js"}
~~~~~~~~
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
~~~~~~~~

This illustrates the general pattern of working with iterables: An *iterable* is an object, representing a collection, with a `[Symbol.iterator]` method, that returns an iteration over the elements of a collection. The iteration over elements is an *iterator*. An iterator is also an object, but with a `.next()` method taht is invoked repeatedly to obtain the elements in order.

Many operations on iterables return iterables. Our `mapIterableWith` returns an iterable. But the iterable it returns is not the same kind of collection as the iterable it consumes. If we give it a `Stack3`, we don't get a stack back. We just get an iterable. (If we want a specific kind of collection, we have to gather the iterable into a collection. We'll see how to do that below.)

Here are two more operations on iterables, `filterIterableWith` and `untilIterable`:

{:lang="js"}
~~~~~~~~
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
  
const untilIterable (fn, iterable) =>
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      return {
        next: () => {
          let {done, value} = iterator.next();
          
          done = done || fn(value);
    
          return ({done, value: done ? undefined : value});
        }
      }
    }
  });
~~~~~~~~

And here's a computation performed using operations on iterables: We'll print the odd squares that are less than or equal to one hundred:

{:lang="js"}
~~~~~~~~
const compose = (fn, ...rest) =>
  (...args) =>
    (rest.length === 0)
      ? fn(...args)
      : fn(compose(...rest)(...args))

const callLeft = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...args, ...remainingArgs);

const squaresOf = callLeft(mapIterableWith, (x) => x * x);
const oddsOf = callLeft(mapIterableWith, (x) => x % 2 === 1);
const untilTooBig = callLeft(until, (x) => x > 100);

for (let s of compose(untilTooBig, oddsOf, squaresOf)(Numbers)) {
  console.log(s)
}
  //=>
    1
    9
    25
    49
    81
~~~~~~~~

For completeness, here are two more handy iterable functions. `firstIterable` returns the first element of an iterable (if it has one), and `restIterable` returns an iterable that iterates over all but the first element of an iterable. They are equivalent to destructuring arrays with `[first, ...rest]`:

{:lang="js"}
~~~~~~~~
const firstIterable = (iterable) =>
  iterable[Symbol.iterator]().next().value;

const restIterable = (iterable) => 
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      iterator.next();
      return iterator;
    }
  });
~~~~~~~~

### from

Having iterated over a collection, are we limited to `for..do` and/or gathering the elements in an array literal and/or gathering the elements into the parameters of a function? No, of course not, we can do anything we like with them.

One useful thing is to write a `.from` function that gathers an iterable into a particular collection type. JavaScript's built-in `Array` class already has one:

{:lang="js"}
~~~~~~~~
Array.from(compose(untilTooBig, oddsOf, squaresOf)(Numbers))
  //=> [1, 9, 25, 49, 81]
~~~~~~~~

We can do the same with our own collections. As you recall, functions are mutable objects. And we can assign properties to functions with a `.` or even `[` and `]`. And if we assign a function to a property, we've created a method.

So let's do that:

{:lang="js"}
~~~~~~~~
Stack3.from = function (iterable) {
  const stack = this();
  
  for (let element of iterable) {
    stack.push(element);
  }
  return stack;
}

Pair1.from = (iterable) =>
  (function interationToList (iteration) {
    const {done, value} = iteration.next();
    
    return done ? EMPTY : Pair1(value, interationToList(iteration));
  })(iterable[Symbol.iterator]())
~~~~~~~~

Now we can go "end to end," If we want to map a linked list of numbers to a linked list of the squares of some numbers, we can do that:

{:lang="js"}
~~~~~~~~
const numberList = Pair1.from(untilIterable((x) => x > 10, Numbers));

Pair1.from(squaresOf(numberList))
  //=> {"first":0,
        "rest":{"first":1,
                "rest":{"first":4,
                        "rest":{ ...
~~~~~~~~

### why operations on iterables?

The operations on iterables are interesting, but let's reiterate why we care: In JavaScript, we build single-responsibility objects, and single-responsibility functions, and we compose these together to build more full-featured objects and algorithms.

> Composing an iterable with a `mapIterable` method cleaves the responsibility for knowing how to map from the fiddly bits of how a linked list differs from a stack

in the older style of object-oriented programming, we built "fat" objects. Each collection knew how to map itself (`.map`), how to fold itself (`.reduce`), how to filter itself (`.filter`) and how to find one element within itself (`.find`). If we wanted to flatten collections to arrays, we wrote a `.toArray` method for each type of collection.

Over time, this informal "interface" for collections grows by accretion. Some methods are only added to a few collections, some are added to all. But our objects grow fatter and fatter. We tell ourselves that, well, a collection ought to know how to map itself.

But we end up recreating the same bits of code in each `.map` method we create, in each `.reduce` method we create, in each `.filter` method we create, and in each `.find` method. Each one has its own variation, but the overall form is identical. That's a sign that we should work at a higher level of abstraction, and working with iterables is that higher level of abstraction.

This "fat object" style springs from a misunderstanding: When we say a collection should know how to perform a map over itself, we don't need for the collection to handle every single detail. That would be like saying that when we ask a bank teller for some cash, they personally print every bank note.

Object-oriented collections should definitely have methods for mapping, reducing, filtering, and finding. And they should know how to accomplish the desired result, but they should do so by delegating as much of the work as possible to operations like `mapIterableWith`.

Composing an iterable with a `mapIterable` method cleaves the responsibility for knowing how to map from the fiddly bits of how a linked list differs from a stack. And if we want to create convenience methods, we can reuse common pieces:

{:lang="js"}
~~~~~~~~
const extend = function (consumer, ...providers) {
  for (let i = 0; i < providers.length; ++i) {
    const provider = providers[i];
    for (let key in provider) {
      if (provider.hasOwnProperty(key)) {
        consumer[key] = provider[key]
      }
    }
  }
  return consumer
};
  
const mapIterableWith = (fn, iterable) =>
  extend({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      return {
        next: () => {
          const {done, value} = iterator.next();
    
          return ({done, value: done ? undefined : fn(value)});
        }
      }
    }
  }, LazyIterable);
  
const reduceIterableWith = (fn, seed, iterable) => {
  const iterator = iterable[Symbol.iterator]();
  let iterationResult,
      accumulator = seed;
  
  while ((iterationResult = iterator.next(), !iterationResult.done)) {
    accumulator = fn(accumulator, iterationResult.value);
  }
  return accumulator;
};
  
const filterIterableWith = (fn, iterable) =>
  extend({
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
  }, LazyIterable);

const untilIterable = (fn, iterable) =>
  extend({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
    
      return {
        next: () => {
          let {done, value} = iterator.next();
        
          done = done || fn(value);
  
          return ({done, value: done ? undefined : value});
        }
      }
    }
  }, LazyIterable);
  
const firstIterable = (iterable) =>
  iterable[Symbol.iterator]().next().value;

const restIterable = (iterable) => 
  extend({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      
      iterator.next();
      return iterator;
    }
  }, LazyIterable);
  
const takeIterable = (numberToTake, iterable) =>
  extend({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      let remainingElements = numberToTake;
    
      return {
        next: () => {
          let {done, value} = iterator.next();
        
          done = done || remainingElements-- <= 0;
  
          return ({done, value: done ? undefined : value});
        }
      }
    }
  }, LazyIterable);
    
const LazyIterable = {
   map: function (fn) {
     return mapIterableWith(fn, this);
   },
   reduce: function (fn, seed) {
     return reduceIterableWith(fn, seed, this);
   },
   filter: function (fn) {
     return filterIterableWith(fn, this);
   },
   find: function (fn) {
     return filterIterableWith(fn, this).first();
   },
   first: function () {
     return firstIterable(this);
   },
   rest: function () {
     return restIterable(this);
   },
   until: function (numberToTake) {
     return untilIterable(numberToTake, this);
   },
   take: function (numberToTake) {
     return takeIterable(numberToTake, this);
   }
}

// Pair, a/k/a linked lists

const EMPTY = {
  isEmpty: () => true
};

const isEmpty = (node) => node === EMPTY;

const Pair = (car, cdr = EMPTY) =>
  extend({
    car,
    cdr,
    isEmpty: () => false,
    [Symbol.iterator]: function () {
      let currentPair = this;
      
      return {
        next: () => {
          if (currentPair.isEmpty()) {
            return {done: true}
          }
          else {
            const value = currentPair.car;
            
            currentPair = currentPair.cdr;
            return {done: false, value}
          }
        }
      }
    }
  }, LazyIterable);

Pair.from = (iterable) =>
  (function interationToList (iteration) {
    const {done, value} = iteration.next();
    
    return done ? EMPTY : Pair(value, interationToList(iteration));
  })(iterable[Symbol.iterator]());
  
// Stack

const Stack = () =>
  extend({
    array: [],
    index: -1,
    push: function (value) {
      return this.array[this.index += 1] = value;
    },
    pop: function () {
      const value = this.array[this.index];
    
      this.array[this.index] = undefined;
      if (this.index >= 0) { 
        this.index -= 1 
      }
      return value
    },
    isEmpty: function () {
      return this.index < 0
    },
    [Symbol.iterator]: function () {
      let iterationIndex = this.index;
      
      return {
        next: () => {
          if (iterationIndex > this.index) {
            iterationIndex = this.index;
          }
          if (iterationIndex < 0) {
            return {done: true};
          }
          else {
            return {done: false, value: this.array[iterationIndex--]}
          }
        }
      }
    }
  }, LazyIterable);
  
Stack.from = function (iterable) {
  const stack = this();
  
  for (let element of iterable) {
    stack.push(element);
  }
  return stack;
}

// Pair and Stack in action
  
Stack.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  .map((x) => x * x)
  .filter((x) => x % 2 == 0)
  .first()

//=> 100
  
Pair.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  .map((x) => x * x)
  .filter((x) => x % 2 == 0)
  .reduce((seed, element) => seed + element, 0)
  
//=> 220
~~~~~~~~

### lazy iterables {#lazy-iterables}

"Laziness" is a very pejorative word when applied to people. But it can be an excellent strategy for efficiency in algorithms. Let's be precise: *Laziness* is the characteristic of not doing any work until you know you need the result of the work.

Here's an example. Compare these two:

{:lang="js"}
~~~~~~~~
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  .map((x) => x * x)
  .filter((x) => x % 2 == 0)
  .reduce((seed, element) => seed + element, 0)
  
Pair.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  .map((x) => x * x)
  .filter((x) => x % 2 == 0)
  .reduce((seed, element) => seed + element, 0)
~~~~~~~~

Both expressions evaluate to `220`. And they array is faster in practice, because it is a built-in data type that performs its work in the engine, while the linked list does its work in JavaScript.

But it's still illustrative to dissect something important: Array's `.map` and `.filter` methods gather their results into new arrays. Thus, calling `.map.filter.reduce` produces two temporary arrays that are discarded when `.reduce` performs its final computation.

Whereas the `.map` and `.filter` methods on `Pair` work with iterators. They produce small iterable objects that refer back to the original iteration. This reduces the memory footprint. When working with very large collections and many operations, this can be important.

The effect is even more pronounced when we use methods like `first`, `until`, or `take`:

{:lang="js"}
~~~~~~~~
Stack.from([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
            10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
            20, 21, 22, 23, 24, 25, 26, 27, 28, 29])
  .map((x) => x * x)
  .filter((x) => x % 2 == 0)
  .first()
~~~~~~~~

This expression begins with a stack containing 30 elements. The top two are `29` and `28`. It maps to the squares of all 30 numbers, but our code for mapping an iteration returns an iterable that can iterate over the squares of our numbers, not an array or stack of the squares. Same with `.filter`, we get an iterable that can iterate over the even squares, but not an actual stack or array.

Finally, we take the first element of that filtered, squared iterable and now JavaScript actually iterates over the stack's elements, and it only needs to square two of those elements, `29` and `28`, to return the answer.

We can confirm this:

{:lang="js"}
~~~~~~~~
Stack.from([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
            10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
            20, 21, 22, 23, 24, 25, 26, 27, 28, 29])
  .map((x) => {
    console.log(`squaring ${x}`);
    return x * x
  })
  .filter((x) => {
    console.log(`filtering ${x}`);
    return x % 2 == 0
  })
  .first()

//=>
  squaring 29
  filtering 841
  squaring 28
  filtering 784
  784
~~~~~~~~

If we write the almost identical thing with an array, we get a different behaviour:

{:lang="js"}
~~~~~~~~
[ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
  .reverse()
  .map((x) => {
    console.log(`squaring ${x}`);
    return x * x
  })
  .filter((x) => {
    console.log(`filtering ${x}`);
    return x % 2 == 0
  })[0]

//=>
  squaring 0
  squaring 1
  squaring 2
  squaring 3
  ...
  squaring 28
  squaring 29
  filtering 0
  filtering 1
  filtering 4
  ...
  filtering 784
  filtering 841
  784
~~~~~~~~

Arrays copy-on-read, so every time we perform a map or filter, we get a new array and perform all the computations. This might be expensive.

You recall we briefly touched on the idea of infinite collections? Let's make iterable numbers. They *have* to be lazy, otherwise we couldn't write things like:

{:lang="js"}
~~~~~~~~
const Numbers = extend({
  [Symbol.iterator]: () => {
    let n = 0;
    
    return {
      next: () =>
        ({done: false, value: n++})
    }
  }
}, LazyCollection);

const firstCubeOver1234 =
  Numbers
    .map((x) => x * x * x)
    .filter((x) => x > 1234)
    .first()

//=> 1331
~~~~~~~~

Balanced against their flexibility, our "lazy iterables" use structure sharing. If we mutate a collection after taking an iterable, we might get an unexpected result. This is why "pure" functional languages like Haskell combine lazy semantics with immutable collections, and why even "impure" languages like Clojure emphasize the use of immutable collections.

### eager iterables

Arrays have *eager* semantics for `.map`, `.filter`, `.rest` and `.take`. They return another array, not a lazy iterable. Whereas, the `Stack` and `Pair` collections we wrote have *lazy* semantics: They return a lazy iterable and when we want a true collection, we have to gather the elements into an array or another collection using `.from`:

{:lang="js"}
~~~~~~~~
const evenSquares = Pair.from(
  Pair.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .map((x) => x * x)
    .filter((x) => x % 2 == 0)
  );

[...evenSquares]
  //=> [4,16,36,64,100]
~~~~~~~~

Or if we want to design a collection with eager semantics for `.map`, `.filter`, `.rest` and `.take`, we can do that:

{:lang="js"}
~~~~~~~~
const EagerIterable = (gatherable) =>
  ({
     map: function (fn) {
       return gatherable.from(mapIterableWith(fn, this));
     },
     reduce: function (fn, seed) {
       return reduceIterableWith(fn, seed, this);
     },
     filter: function (fn) {
       return gatherable.from(filterIterableWith(fn, this));
     },
     find: function (fn) {
       return filterIterableWith(fn, this).first();
     },
     first: function () {
       return firstIterable(this);
     },
     rest: function () {
       return gatherable.from(restIterable(this));
     },
     take: function (numberToTake) {
       return gatherable.from(takeIterable(numberToTake, this));
     }
  })
  
const EagerStack = () =>
  extend({
    array: [],
    index: -1,
    push: function (value) {
      return this.array[this.index += 1] = value;
    },
    pop: function () {
      const value = this.array[this.index];
    
      this.array[this.index] = undefined;
      if (this.index >= 0) { 
        this.index -= 1 
      }
      return value
    },
    isEmpty: function () {
      return this.index < 0
    },
    [Symbol.iterator]: function () {
      let iterationIndex = this.index;
      
      return {
        next: () => {
          if (iterationIndex > this.index) {
            iterationIndex = this.index;
          }
          if (iterationIndex < 0) {
            return {done: true};
          }
          else {
            return {done: false, value: this.array[iterationIndex--]}
          }
        }
      }
    }
  }, EagerIterable(EagerStack));
  
EagerStack.from = function (iterable) {
  const stack = this();
  
  for (let element of iterable) {
    stack.push(element);
  }
  return stack;
}

EagerStack
  .from([1, 2, 3, 4, 5])
  .map((x) => x * 2)
  
//=> {"array":[10,8,6,4,2],"index":4}
~~~~~~~~

And we can go back and forth between them. For example, if we want a lazy map of an array, we can use the `mapIterableWith` function to return a lazy iterable. And as we just noted, we can use `.from` to eagerly gather any iterable into a collection.

### summary

Iterators are a JavaScript feature that allow us to separate the concerns of how to iterate over a collection from what we want to do with the elements of a collection. *Iterable* collections can be iterated over or gathered into another collection, either lazily or eagerly.

Separating concerns with iterators speaks to JavaScript's fundamental nature: It's a language that *wants* to compose functionality out of small, singe-responsibility pieces, whether those pieces are functions or objects built out of functions.
