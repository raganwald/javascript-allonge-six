## Left-Variadic Functions {#left-variadic}

A *variadic function* is a function that is designed to accept a variable number of arguments.[^eng] In JavaScript, you can make a variadic function by gathering parameters. For example:

[^eng]: English is about as inconsistent as JavaScript: Functions with a fixed number of arguments can be unary, binary, ternary, and so forth. But can they be "variary?" No! They have to be "variadic."

{:lang="js"}
~~~~~~~~
const abccc = (a, b, ...c) => {
  console.log(a);
  console.log(b);
  console.log(c);
};

abccc(1, 2, 3, 4, 5)
  1
  2
  [3,4,5]
~~~~~~~~

This can be useful when writing certain kinds of destructuring algorithms. For example, we might want to have a function that builds some kind of team record. It accepts a coach, a captain, and an arbitrary number of players. Easy in ECMAScript 2015:

{:lang="js"}
~~~~~~~~
function team(coach, captain, ...players) {
  console.log(`${captain} (captain)`);
  for (let player of players) {
    console.log(player);
  }
  console.log(`squad coached by ${coach}`);
}

team('Luis Enrique', 'Xavi Hernández', 'Marc-André ter Stegen', 'Martín Montoya', 'Gerard Piqué')
  //=>
    Xavi Hernández (captain)
    Marc-André ter Stegen
    Martín Montoya
    Gerard Piqué
    squad coached by Luis Enrique
~~~~~~~~

But we can't go the other way around:

{:lang="js"}
~~~~~~~~
function team2(...players, captain, coach) {
  console.log(`${captain} (captain)`);
  for (let player of players) {
    console.log(player);
  }
  console.log(`squad coached by ${coach}`);
}
//=> Unexpected token
~~~~~~~~

ECMAScript 2015 only permits gathering parameters from the *end* of the parameter list. Not the beginning. What to do?

### a history lesson

In "Ye Olde Days,"[^ye] JavaScript could not gather parameters, and we had to either do backflips with `arguments` and `.slice`, or we wrote ourselves a `variadic` decorator that could gather arguments into the last declared parameter. Here it is:

[^ye]: Another history lesson. "Ye" in "Ye Olde," was not actually spelled with a "Y" in days of old, it was spelled with a [thorn](https://en.wikipedia.org/wiki/Thorn_(letter)), and is pronounced "the." Another word, "Ye" in "Ye of little programming faith," is pronounced "ye," but it's a different word altogether.

{:lang="js"}
~~~~~~~~
var __slice = Array.prototype.slice;

function rightVariadic (fn) {
  if (fn.length < 1) return fn;

  return function () {
    var ordinaryArgs = (1 <= arguments.length ? 
          __slice.call(arguments, 0, fn.length - 1) : []),
        restOfTheArgsList = __slice.call(arguments, fn.length - 1),
        args = (fn.length <= arguments.length ?
          ordinaryArgs.concat([restOfTheArgsList]) : []);
    
    return fn.apply(this, args);
  }
};

var firstAndButFirst = rightVariadic(function test (first, butFirst) {
  return [first, butFirst]
});

firstAndButFirst('why', 'hello', 'there', 'little', 'droid')
  //=> ["why",["hello","there","little","droid"]]
~~~~~~~~

We don't need `rightVariadic` any more, because instead of:

{:lang="js"}
~~~~~~~~
var firstAndButFirst = rightVariadic(function test (first, butFirst) {
  return [first, butFirst]
});
~~~~~~~~

We now simply write:

{:lang="js"}
~~~~~~~~
const firstAndButFirst = (first, ...butFirst) => [first, butFirst];
~~~~~~~~

This is a *right-variadic function*, meaning that it has one or more fixed arguments, and the rest are gathered into the rightmost argument.

### overcoming limitations

It's nice to have progress. But as noted above, we can't write:

{:lang="js"}
~~~~~~~~
const butLastAndLast = (...butLast, last) => [butLast, last];
~~~~~~~~

That's a *left-variadic function*. All left-variadic functions have one or more fixed arguments, and the rest are gathered into the leftmost argument. JavaScript doesn't do this. But if we wanted to write left-variadic functions, could we make ourselves a `leftVariadic` decorator to turn a function with one or more arguments into a left-variadic function?

We sure can, by using the techniques from `rightVariadic`. Mind you, we can take advantage of modern JavaScript to simplify the code:

{:lang="js"}
~~~~~~~~
const leftVaridic = (fn) => {
  if (fn.length < 1) {
    return fn;
  }
  else {
    return function (...args) {
      return fn.apply(this, [args.slice(0, args.length - fn.length + 1)].concat(args.slice(args.length - fn.length + 1)))
    }
  }
};

const butLastAndLast = leftVaridic((butLast, last) => [butLast, last]);

butLastAndLast('why', 'hello', 'there', 'little', 'droid')
  //=> [["why","hello","there","little"],"droid"]
~~~~~~~~