## Iterables and Generators {#iterables}

Objects in JavaScript can model all sorts of things. At a very abstract level, models usually represent either heterogeneous or homogeneous collections of things. Or what we might call "things" and "collections of things."

When discussing functions, we looked at the benefits of writing [Functional Iterators](#functional-iterators). We can do the same thing for objects. Here's a stack that has its own functional iterator method:

{:lang="js"}
~~~~~~~~
const Stack = () =>
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

const stack = Stack();

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

Recall how we wrote a `sum` function that was a fold over an iterator function:

{:lang="js"}
~~~~~~~~
const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;;
  
  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}
~~~~~~~~

We can use it with our stack:

{:lang="js"}
~~~~~~~~
const stack = Stack();

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
      sum = 0;;
  
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
const Stack = () =>
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

const stack = Stack();

stack.push(2000);
stack.push(10);
stack.push(5)

const collectionSum = (collection) => {
  const iterator = collection.iterator();
  
  let eachIteration,
      sum = 0;;
  
  while ((eachIteration = iterator.next(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

collectionSum(stack)
  //=> 2015
~~~~~~~~

Nowour `.iterator()` method is returning an iterator object. When working with objects, we do things the object way. But having started by building functional iterators, we understand what is happening underneath the object's scaffolding.

### Iterables

Prople have been writing iterators since JavaScript was first released in the late 1990s. Since there was no particular standard way to do it, people used all sorts of methods, and their methods returned all sorts of things: Objects with various interfaces, functional iterators, you name it.

So, when a standard way to write iterators was added to the JavaScript language, it didn't make sense to use a method like `.iterator()` for it: That would conflict with existing code. Instead, the language encourages new code to be written with a different name for the method that a collection object uses to return its iterator.

To ensure that the method would not confluct with any existing code, JavaScript provides a *symbol*. Symbols are unique constants that are guaranteed not to conflict with existing strings. Symbols are a longstanding technique in programming going back to Lisp, where the `GENSYM` function generated... You guessed it... Symbols.

The expression `Symbol.iterator` evaluates to a special symbol representing the name of the method that objects should use if they return an iterator object.

Our stack does, so instead of binding the existing iterator method to the name `iterator`, we bind it to the `Symbol.iterator`. We'll do that using the `[` `]` syntax for using an expression as an object literal key:

{:lang="js"}
~~~~~~~~
const Stack = () =>
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

const stack = Stack();

stack.push(2000);
stack.push(10);
stack.push(5);

const collectionSum = (collection) => {
  const iterator = collection[Symbol.iterator]();
  
  let eachIteration,
      sum = 0;;
  
  while ((eachIteration = iterator.next(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

collectionSum(stack)
  //=> 2015
~~~~~~~~

That seems like a lot of hooey: Using [Symbol.iterator] instead of `.iterator` seems like adding an extra moving part for nothing. DO we get anything in return?

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

const pair = (first, rest = EMPTY) =>
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
    : pair(first, list(...rest))
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

One caveat of spreading iterables: JavaScript creates an array out of the elements of the iterable. That might be very wasteful for extremely large collections. For example, if we spread a large collection just to find an element in teh collection, it might have been wiser to iterate over the element using its iterator directly.

And if we have an infinite collection, spreading is going to fail outright.

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

['all the numbers', ...Numbers]
  //=> infinite loop!
  
firstAndSecondElement(...Numbers)
  //=> infinite loop!
~~~~~~~~

However, iterables are extremely useful. And since they do the same thing as functional iterators, we can write object-oriented analogues for them. Here's `mapIterableWith`, it takes any iterable and returns a mapping over the original iterable:

{:lang="js"}
~~~~~~~~
const mapIteratableWith = (fn, iterable) =>
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

And here's `filterIterableWith` and `until`:

{:lang="js"}
~~~~~~~~
const filterIteratableWith = (fn, iterable) =>
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
  
const until = (fn, iterable) =>
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
  
const compose = (fn, ...rest) =>
  (...args) =>
    (rest.length === 0)
      ? fn(...args)
      : fn(compose(...rest)(...args))

const squaresOf = callLeft(mapIteratableWith, (x) => x * x);
const oddsOf = callLeft(filterIteratableWith, (x) => x % 2 === 1);
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

### why do we care about iterables?

These are all interesting, but let's reiterate why we care. We build single-responsibility objects, and single-responsibility functions, and we compose these together to build more full-featured objects and algorithms.


