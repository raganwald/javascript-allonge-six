## Interlude: The Carpenter Interviews for a Job

"The Carpenter" was a JavaScript programmer, well-known for a meticulous attention to detail and love for hand-crafted, exquisitely joined code. The Carpenter normally worked through personal referrals, but from time to time a recruiter would slip through his screen. One such recruiter was Bob Plissken. Bob was well-known in the Python community, but his clients often needed experience with other languages.

Plissken lined up a technical interview with a well-funded startup in San Francisco. The Carpenter arrived early for his meeting with "Thing Software," and was shown to conference room 13. A few minutes later, he was joined by one of the company's developers, Christine.

### the problem

After some small talk, Christine explained that they liked to ask candidates to whiteboard some code. Despite his experience and industry longevity, the Carpenter did not mind being asked to demonstrate that he was, in fact, the person described on the resumé.

Many companies use white-boarding code as an excuse to have a technical conversation with a candidate, and The Carpenter felt that being asked to whiteboard code was an excuse to have a technical conversation with a future colleague. "Win, win" he thought to himself.

[![Chessboard](images/chessboard.jpg)](https://www.flickr.com/photos/stigrudeholm/6710684795)

Christine intoned the question, as if by rote:

> Consider a finite checkerboard of unknown size. On each square, we randomly place an arrow pointing to one of its four sides. A chequer is placed randomly on the checkerboard. Each move consists of moving the chequer one square in the direction of the arrow in the square it occupies. If the arrow should cause the chequer to move off the edge of the board, the game halts.

> The problem is this: The game board is hidden from us. A player moves the chequer, following the rules. As the player moves the chequer, they calls out the direction of movement, e.g. "↑, →, ↑, ↓, ↑, →..." Write an algorithm that will determine whether the game halts, strictly from the called out directions, in finite time and space.

"So," The Carpenter asked, "I am to write an algorithm that takes a possibly infinite stream of..."

Christine interrupted. "To save time, we have written a template of the solution for you in ECMASCript 2015 notation. Fill in the blanks. Your code should not presume anything about the gameboard's size or contents, only that it is given an arrow every time though the while loop. You may use [babeljs.io](http://babeljs.io), or [ES6Fiddle](http://www.es6fiddle.net) to check your work. "

Christine quickly scribbled on the whiteboard:

{:lang="js"}
~~~~~~~~
const Game = (size = 8) => {
  
  // initialize the board
  const board = [];
  for (let i = 0; i < size; ++i) {
    board[i] = [];
    for (let j = 0; j < size; ++j) {
      board[i][j] = '←→↓↑'[Math.floor(Math.random() * 4)];
    }
  }
  
  // initialize the position
  let initialPosition = [
    2 + Math.floor(Math.random() * (size - 4)), 
    2 + Math.floor(Math.random() * (size - 4))
  ];
  
  // ???
  let [x, y] = initialPosition;
  
  const MOVE = {
    "←": ([x, y]) => [x - 1, y],
    "→": ([x, y]) => [x + 1, y],
    "↓": ([x, y]) => [x, y - 1],
    "↑": ([x, y]) => [x, y + 1] 
  };
  while (x >= 0 && y >=0 && x < size && y < size) {
    const arrow = board[x][y];
    
    // ???
    
    [x, y] = MOVE[arrow]([x, y]);
  }
  // ???
};
~~~~~~~~

"What," Christine asked, "Do you write in place of the three `// ???` placeholders to determine whether the game halts?"

### the carpenter's solution

The Carpenter was not surprised at the problem. Bob Plissken was a crafty, almost reptilian recruiter that traded in information and secrets. Whenever Bob sent a candidate to a job interview, he debriefed them afterwards and got them to disclose what questions were asked in the interview. He then coached subsequent candidates to give polished answers to the company's pet technical questions.

And just as companies often pick a problem that gives them broad latitude for discussing alternate approaches and determining that depth of a candidate's experience, The Carpenter liked to sketch out solutions that provided an opportunity to judge the interviewer's experience and provide an easy excuse to discuss the company's approach to software design.

Bob had, in fact, warned The Carpenter that "Thing" liked to ask either or both of two questions: Determine how to detect a loop in a linked list, and determine whether the chequerboard game would halt. To save time, The Carpenter had prepared the same answer for both questions.

The Carpenter coughed softly, then began. "To begin with, I'll transform a game into an iterable that generates arrows, using the 'Starman' notation for generators."

"I will add just five lines of code the `Game` function, and two of those are closing braces:"

{:lang="js"}
~~~~~~~~
  return ({
    [Symbol.iterator]: function* () {
~~~~~~~~

And:

{:lang="js"}
~~~~~~~~
        yield arrow;
~~~~~~~~

And:

{:lang="js"}
~~~~~~~~
    }
  });
~~~~~~~~

"The finished function reads:"

{:lang="js"}
~~~~~~~~
const Game = (size = 8) => {
  
  // initialize the board
  const board = [];
  for (let i = 0; i < size; ++i) {
    board[i] = [];
    for (let j = 0; j < size; ++j) {
      board[i][j] = '←→↓↑'[Math.floor(Math.random() * 4)];
    }
  }
  
  // initialize the position
  let initialPosition = [
    2 + Math.floor(Math.random() * (size - 4)), 
    2 + Math.floor(Math.random() * (size - 4))
  ];
  
  return ({
    [Symbol.iterator]: function* () {
      let [x, y] = initialPosition;
  
      const MOVE = {
        "←": ([x, y]) => [x - 1, y],
        "→": ([x, y]) => [x + 1, y],
        "↓": ([x, y]) => [x, y - 1],
        "↑": ([x, y]) => [x, y + 1] 
      };
      
      while (x >= 0 && y >=0 && x < size && y < size) {
        const arrow = board[x][y];
        
        yield arrow;
        [x, y] = MOVE[arrow]([x, y]);
      }
    }
  });
};
~~~~~~~~

"Now that we have an iterable, we can transform the iterable of arrows into an iterable of positions." The Carpenter sketched quickly. "We'll need some common utilities. You'll find equivalents in a number of JavaScript libraries, but I'll quote those given in [JavaScript Allongé](https://leanpub.com/javascriptallongesix):"

"For starters, `takeFromCollection` transforms an iterable into one that yields at most a fixed number of elements. It's handy for debugging. We'll use it to check that our `Game` is working as an iterable:"

{:lang="js"}
~~~~~~~~
const takeFromCollection = (numberToTake, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      let remainingElements = numberToTake;
      
      for (let element of iterable) {
        if (remainingElements-- <= 0) break;
        yield element;
      }
    }
  });

Array.from(takeFromCollection(10, Game()))
  //=>
    ["↑","←","→","←","→","←","→","←","→","←"]
~~~~~~~~

"This doesn't actually end up in our solution, it's just to check our work as we go along. And you can find it in libraries, it's not something we need to reinvent whenever we work with iterables."

"But now to the business. We want to take the arrows and convert them to positions. For that, we'll map the Game iterable to positions. A `statefulMap` is a lazy map that preserves state from iteration to iteration. That's what we need, because we need to know the current position to map each move to the next position."

"Again, this is a standard idiom we can obtain from libraries, we don't reinvent the wheel. I'll show it here for clarity:"

{:lang="js"}
~~~~~~~~
const statefulMapIterableWith = (fn, seed, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      let value,
          state = seed;
      
      for (let element of iterable) {
        [state, value] = fn(state, element);
        yield value;
      }
    }
  });
  
const indexed = statefulMapIterableWith(
  (index, value) => {
    return [index + 1, [index, value]]
  },
  0,
  ["prince", "of", "darkness"])

Array.from(indexed)
  //=>
    [[0,"prince"],[1,"of"],[2,"darkness"]]
~~~~~~~~

"Armed with this, it's straightforward to map an iterable of directions to an iterable of strings representing positions:"

{:lang="js"}
~~~~~~~~
const positionsOf = (game) =>
  statefulMapIterableWith(
    (position, direction) => {
      const MOVE = {
        "←": ([x, y]) => [x - 1, y],
        "→": ([x, y]) => [x + 1, y],
        "↓": ([x, y]) => [x, y - 1],
        "↑": ([x, y]) => [x, y + 1] 
      };
      const [x, y] =  MOVE[direction](position);
      
      return [position, `x: ${x}, y: ${y}`];
    },
    [0, 0],
    game);

Array.from(takeFromCollection(10, positionsOf(Game())))
  //=>
    ["x: -1, y: 0","x: 0, y: 1","x: -1, y: 0",
     "x: 0, y: -1","x: 0, y: 1","x: 0, y: -1",
     "x: 0, y: 1","x: 0, y: -1","x: 0, y: 1",
     "x: 0, y: -1"]
~~~~~~~~

The Carpenter reflected. "Having turned our game loop into an iterable, we can now see that our problem of whether the game terminates is isomorphic to the problem of detecting whether the positions given ever repeat themselves: If the chequer ever returns to a position it has previously visited, it will cycle endlessly."

"We could draw positions as nodes in a graph, connected by arcs representing the arrows. Detecting whether the game terminates is equivalent to detecting whether the graph contains a cycle."

![The Tortoise and the Hare](images/tortoise-hare.png)

"There's an old joke that a mathematician is someone who will take a five-minute problem, then spend an hour proving it is equivalent to another problem they have already solved. I approached this question in that spirit. Now that we have created an iterable of values that can be compared with `===`, I can show you this function:"

{:lang="js"}
~~~~~~~~
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
~~~~~~~~

"A long time ago," The Carpenter explained, "Someone asked me a question in an interview. I have never forgotten the question, or the general form of the solution. The question was, *Given a linked list, detect whether it contains a cycle. Use constant space.*"

"This is, of course, the most common solution, it is [Floyd's cycle-finding algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Tortoise_and_hare), although there is some academic dispute as to whether Robert Floyd actually discovered it or was misattributed by Knuth."

"Thus, the solution to the game problem is:"

{:lang="js"}
~~~~~~~~
  const terminates = (game) =>
    tortoiseAndHare(positionsOf(game))
~~~~~~~~

"This solution makes use of iterables and a single utility function, `statefulMapIterableWith`. It also cleanly separates the mechanics of the game from the algorithm for detecting cycles in a graph."

### the aftermath

The Carpenter sat down and waited. This type of solution provided an excellent opportunity to explore lazy versus eager evaluation, the performance of iterators versus native iteration, single responsibility design, and many other rich topics.

The Carpenter was confident that although nobody would write this exact code in production, prospective employers would also recognize that nobody would try to detect whether a chequer game terminates in production, either. It's all just a pretext for kicking off an interesting conversation, right?

[![Time](images/time.jpg)](https://www.flickr.com/photos/jlhopgood/6795353385)

Christine looked at the solution on the board, frowned, and glanced at the clock on the wall. "*Well, where has the time gone?*"

"We at the Thing Software company are very grateful you made some time to visit with us, but alas, that is all the time we have today. If we wish to talk to you further, we'll be in touch."

The Carpenter never did hear back from them, but the next day there was an email containing a generous contract from Friends of Ghosts ("FOG"), a codename for a stealth startup doing interesting work, and the Thing interview was forgotten.

Some time later, The Carpenter ran into Bob Plissken at a local technology meet-up. "John! What happened at Thing?" Bob wanted to know, "I asked them what they thought of you, and all they would say was, *Writes unreadable code*. I thought it was a lock! I thought you'd finally make your escape from New York."

The Carpenter smiled. "I forgot about them, it's been a while. So, do They Live?"