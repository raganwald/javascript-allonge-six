## Refactoring to Generator

In [Tortoises, Hares, and Teleporting Turtles](#tortoises), we looked at the "Tortoise and Hare" algorithm for detecting a linked list. Like many such algorithms, it "tangles" two different concerns:

1. The mechanism for iterating over a list.
2. The algorithm for detecting a loop in a list.

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

const forceAppend = (list1, list2) => {
  if (isEmpty(list1)) {
    return "FAIL!"
  }
  if (isEmpty(list1.rest)) {
    list1.rest = list2;
  }
  else {
    forceAppend(list1.rest, list2);
  }
}

const tortoiseAndHare = (aPair) => {
  let tortoisePair = aPair,
      harePair = aPair.rest;
  
  while (true) {
    if (isEmpty(tortoisePair) || isEmpty(harePair)) {
      return false;
    }
    if (tortoisePair.first === harePair.first) {
      return true;
    }
    
    harePair = harePair.rest;
    
    if (isEmpty(harePair)) {
      return false;
    }
    if (tortoisePair.first === harePair.first) {
      return true;
    }
    
    tortoisePair = tortoisePair.rest;
    harePair = harePair.rest;
  }
};

const aList = list(1, 2, 3, 4, 5);

tortoiseAndHare(aList)
  //=> false

forceAppend(aList, aList.rest.rest);

tortoiseAndHare(aList);
  //=> true
~~~~~~~~

We then went on to discuss how to use [functional iterators](#functional-iterators), [iterables](#iterables), and [generators](#generators) to untangle concerns like this. No matter how we implement them, iterators are stateful functions that iterate over a data structure. Every time we call them, they return the next element from the data structure. If and when they completes their traversal, they return `{done: true}`.

Iterators allow us to write (or refactor) functions to operate on iterators instead of data structures. That increases reuse. We can also write higher-order functions that operate directly on iterators such as mapping and selecting. That allows us to write [lazy algorithms](#lazy-iterables).

### refactoring the tortoise and hare

Now we'll refactor the Tortoise and Hare to use iterators instead of directly operating on linked lists. First, here's a `Pair` implementation that is also an iterable:

{:lang="javascript"}
~~~~~~~~
const EMPTY = {
  isEmpty: () => true
};

const isEmpty = (node) => node === EMPTY;

const Pair = (car, cdr = EMPTY) =>
  ({
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
  });

const forceAppend = (list1, list2) => {
  if (isEmpty(list1)) {
    return "FAIL!"
  }
  if (isEmpty(list1.cdr)) {
    list1.cdr = list2;
  }
  else {
    forceAppend(list1.cdr, list2);
  }
}

Pair.from = (iterable) =>
  (function interationToList (iteration) {
    const {done, value} = iteration.next();
    
    return done ? EMPTY : Pair(value, interationToList(iteration));
  })(iterable[Symbol.iterator]());

const tortoiseAndHare = (iterable) => {
  const hare = iterable[Symbol.iterator]();
  let hareResult = (hare.next(), hare.next());
  
  for (let tortoiseValue of iterable) {
    
    hareResult = hare.next();
    
    if (hareResult.done) {
      return false;
    }
    if (tortoiseValue === hareResult.value) {
      return true;
    }
    
    hareResult = hare.next();
    
    if (hareResult.done) {
      return false;
    }
    if (tortoiseValue === hareResult.value) {
      return true;
    }
  }
  return false;
};

tortoiseAndHare([1, 2, 3, 4, 5]);
  //=> false
  
const oneToFive = Pair.from([1, 2, 3, 4, 5]);
forceAppend(oneToFive, oneToFive.cdr.cdr);

tortoiseAndHare(oneToFive);
  //=> true
~~~~~~~~

We have now refactored it into a function that operates on any iterable, not just one linked lists. It's classic "Duck Typed" Object-Orientation. So, how shall we put it to work?

## A Drunken Walk Across A Chequerboard

Here's another job interview puzzle.[^yecch]

[^yecch]: This book does not blindly endorse asking programmers to solve this or any abstract problem in a job interview.

*Consider a finite checkerboard. On each square we randomly place an arrow pointing to one of its four sides. For convenience, we shall uniformly label the directions: N, S, E, and W. A chequer is placed randomly on the checkerboard. Each move consists of moving the red chequer one square in the direction of the arrow in the square it occupies. If the arrow should cause the chequer to move off the edge of the board, the game halts.*

*The problem is this: The game board is hidden from us. A player moves the chequer, following the rules. As a player moves the chequer, he calls out the direction of movement, e.g. "N, E, N, S, N, E..." Write an algorithm that will determine whether the game halts, strictly from the called out directions, in finite time and space.*

### the insight

Our solution will rest on the observation that as the chequer follows a path, if it ever visits a square for a second time, it will cycle indefinitely without falling off the board. Otherwise, on a finite board, it must eventually fall off the board after at most visiting every square once.

Therefore, if we think of this as detecting whether the chequer revisits a square in constant space, we can see this is isomorphic to detecting whether a linked list has a loop by checking to see whether it revisits the same node.

Our algorithm will be to convert the called-out directions into positions. If the player halts the game, obviously we report that the game halts. If the chequer revists a square, we report that the game does not halt. And if we haven't detected that the chequer revisits a square, we continue to listen for more moves.

### the game

We'll start with the presumption that the game is an iterable. As we iterate over it, we are given letters. Let's write the game out:

{:lang="javascript"}
~~~~~~~~
const Game = (size =  Math.floor(Math.random() * 8) + 8) => {
  const board = [];
  
  for (let i = 0; i < size; ++i) {
    board[i] = [];
    for (let j = 0; j < size; ++j) {
      board[i][j] = '←→↓↑'[Math.floor(Math.random() * 4)];
    }
  }
  const ARROW_TO_DIRECTION = {
    "←": "W",
    "→": "E",
    "↓": "S",
    "↑": "N"
  };
  const MOVE = {
    "←": ([x, y]) => [x - 1, y],
    "→": ([x, y]) => [x + 1, y],
    "↓": ([x, y]) => [x, y - 1],
    "↑": ([x, y]) => [x, y + 1] 
  };
  
  let initialPosition = [
    2 + Math.floor(Math.random() * (size - 4)), 
    2 + Math.floor(Math.random() * (size - 4))
  ];

  return ({
    [Symbol.iterator]: function* () {
      let [x, y] = initialPosition;
      
      while (x >= 0 && y >=0 && x < size && y < size) {
        const arrow = board[x][y];
        
        yield ARROW_TO_DIRECTION[arrow];
        [x, y] = MOVE[arrow]([x, y]);
      }
    }
  });
};

const takeIterable = (numberToTake, iterable) =>
  ({
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
  });

Array.from(takeIterable(10, game(8)))
  //=> ["N","E","N","W","S","E","N","W","S","E"]
~~~~~~~~

### stateful mapping

Our goal is to transform the iteration of directions into an iteration that the Tortoise and Hare can use to detect revisiting the same square. Our approach is to convert the directions into offsets representing the position of the chequer relative to its starting position.

We'll use a `statefulMap`:

{:lang="javascript"}
~~~~~~~~
const statefulMapIterableWith = (fn, seed, iterable) =>
  ({
    [Symbol.iterator]: () => {
      const iterator = iterable[Symbol.iterator]();
      let state = seed;
      
      return {
        next: () => {
          const {done, value} = iterator.next();
    
          return ({done, value: done ? undefined : state = fn(state, value)});
        }
      }
    }
  });
  
const rollingSum = statefulMapIterableWith((x, y) => x + y, 0, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

Array.from(rollingSum)
  //=> [1,3,6,10,15,21,28,36,45,55]
~~~~~~~~

Like a map, a `statefulMap` takes an iterable and maps it to a new iterable. And like a reduce, a `statefulMap` carries an intermediate state from element to element.

Here's how we use `statefulMap` to convert an iterable of directions into an iterable of positions:

{:lang="javascript"}
~~~~~~~~
const callLeft = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...args, ...remainingArgs);
      
const positions = callLeft(statefulMapIterableWith, [0, 0], (position, direction) => {
  const MOVE = {
    "W": ([x, y]) => [x - 1, y],
    "E": ([x, y]) => [x + 1, y],
    "S": ([x, y]) => [x, y - 1],
    "N": ([x, y]) => [x, y + 1] 
  };
  
  return MOVE[direction](position);
});

Array.from(takeIterable(10, positionsOf(Game(8))))
  //=> [[-1,0],[-2,0],[-3,0],[-3,-1],[-3,-2],[-3,-1],[-3,-2],[-3,-1],[-3,-2],[-3,-1]]
~~~~~~~~

### the solution

So. We can take a `Game` instance and produce an iterable that iterates over arrays representing relative positions. If it terminates on its own, the game obviously terminates. And if it repeats itself, the game does not terminate.

Our `tortoiseAndHare`  function takes an iterable and detects this for us, but it insists on comparing elements with `===`. That doesn't work for arrays, so let's improve our algorithm slightly and apply it to our positions:

{:lang="javascript"}
~~~~~~~~
const tortoiseAndHare = (iterable, isSame = ((x, y) => x === y)) => {
  const hare = iterable[Symbol.iterator]();
  let hareResult = (hare.next(), hare.next());
  
  for (let tortoiseValue of iterable) {
    
    hareResult = hare.next();
    
    if (hareResult.done) {
      return false;
    }
    if (isSame(tortoiseValue, hareResult.value)) {
      return true;
    }
    
    hareResult = hare.next();
    
    if (hareResult.done) {
      return false;
    }
    if (isSame(tortoiseValue, hareResult.value)) {
      return true;
    }
  }
  return false;
};
  
const terminates = (game) =>
  tortoiseAndHare(positionsOf(game), (t, h) => t[0] === h[0] && t[1] === h[1])
  
terminates(Game(100))
~~~~~~~~

### preliminary conclusion

Untangling the mechanism of following a linked list from the algorithm of searching for a loop allows us to repurpose the Tortoise and Hare algorithm to solve a question about a path looping.

### no-charge extra conclusion

Can we also refactor the "Teleporting Turtle" algorithm to take an iterable? If so, we should be able to swap algorithms for our game termination detection without rewriting everything in sight. Let's try it:

We start with:

{:lang="javascript"}
~~~~~~~~
const teleportingTurtle = (list) => {
  let speed = 1,
      rabbit = list,
      turtle = rabbit;
  
  while (true) {
    for (let i = 0; i <= speed; i += 1) {
      rabbit = rabbit.rest;
      if (rabbit == null) {
        return false;
      }
      if (rabbit === turtle) {
        return true;
      }
    }
    turtle = rabbit;
    speed *= 2;
  }
  return false;
};
~~~~~~~~

And refactor it to become:

{:lang="javascript"}
~~~~~~~~
const teleportingTurtle = (iterable, isSame = ((x, y) => x === y)) => {
  const hare = iterable[Symbol.iterator]();
  let hareResult = hare.next(),
      speed = 1;
  
  for (let tortoiseResult of iterable) {
    for (let i = 0; i <= speed; i += 1) {
      hareResult = hare.next();
      if (hareResult.done) {
        return false;
      }
      if (isSame(tortoiseResult, hareResult.value)) {
        return true;
      }
    }
    turtle = rabbit;
    speed *= 2;
  }
  return false;
};
~~~~~~~~

Now we can plug it into our termination detector:

{:lang="javascript"}
~~~~~~~~
const terminates = (game) =>
  teleportingTurtle(positionsOf(game), (t, h) => t[0] === h[0] && t[1] === h[1])
  
terminates(Game(4))
  //=> true
terminates(Game(4))
  //=> false
terminates(Game(4))
  //=> false
terminates(Game(4))
  //=> true
~~~~~~~~

Refactoring an algorithm to work with iterables allows us to use the same algorithm to solve different problems, and to swap algorithms for the same problem. This is natural, we have created an abstraction that allows us to plug different items into either side of its interface.
