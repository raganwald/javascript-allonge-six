## Plain Old JavaScript Objects {#pojos}

Lists are not the only way to represent collections of things, but they are the "oldest" data structure in the history of high level languages, because they map very closely to the way the hardware is organized in a computer. Lists are obviously very handy for homogeneous collections of things, like a shopping list:

{:lang="javascript"}
~~~~~~~~
const remember = ["the milk", "the coffee beans", "the biscotti"];
~~~~~~~~

And they can be used to store heterogeneous things in various levels of structure:

{:lang="javascript"}
~~~~~~~~
const user = [["Reginald", "Braithwaite"],[ "author", ["JavaScript Allongé", "JavaScript Spessore", "CoffeeScript Ristretto"]]];
~~~~~~~~

Remembering that the name is the first item is error-prone, and being expected to look at `user[0][1]` and know that we are talking about a surname is unreasonable. So back when lists were the only things available, programmers would introduce constants to make things easier on themselves:

{:lang="javascript"}
~~~~~~~~
const NAME = 0,
      FIRST = 0,
      LAST = 1,
      OCCUPATION = 1,
      TITLE = 0,
      RESPONSIBILITIES = 1;

const user = [["Reginald", "Braithwaite"],[ "author", ["JavaScript Allongé", "JavaScript Spessore", "CoffeeScript Ristretto"]]];
~~~~~~~~

Now they could write `user[NAME][LAST]` or `user[OCCUPATION][TITLE]` instead of `user[0][1]` or `user[1][0]`. Over time, this need to build heterogeneous data structures with access to members by name evolved into the [Dictionary] data type, a mapping from a unique set of objects to another set of objects.

[Dictionary]: https://en.wikipedia.org/wiki/Associative_array

Dictionaries store key-value pairs, so instead of binding `NAME` to `0` and then storing a name in an array at index `0`, we can bind a name directly to `name` in a dictionary, and we let JavaScript sort out whether the implementation is a list of key-value pairs, a hashed collection, a tree of some sort, or anything else.

JavaScript has dictionaries, and it calls them "objects." The word "object" is loaded in programming circles, due to the widespread use of the term "object-oriented programming" that was coined by Alan Kay but has since come to mean many, many things to many different people.

In JavaScript, an object is a map from string keys to values.

### literal object syntax

JavaScript has a literal syntax for creating objects. This object maps values to the keys `year`, `month`, and `day`:

    { year: 2012, month: 6, day: 14 }

Two objects created with separate evaluations have differing identities, just like arrays:

    { year: 2012, month: 6, day: 14 } === { year: 2012, month: 6, day: 14 }
      //=> false

Objects use `[]` to access the values by name, using a string:

    { year: 2012, month: 6, day: 14 }['day']
      //=> 14

Values contained within an object work just like values contained within an array, we access them by reference to the original:

    const unique = () => [],
          x = unique(),
          y = unique(),
          z = unique(),
          o = { a: x, b: y, c: z };

    o['a'] === x && o['b'] === y && o['c'] === z
      //=> true

Names needn't be alphanumeric strings. For anything else, enclose the label in quotes:

    { 'first name': 'reginald', 'last name': 'lewis' }['first name']
      //=> 'reginald'

If the name is an alphanumeric string conforming to the same rules as names of variables, there's a simplified syntax for accessing the values:

    const date = { year: 2012, month: 6, day: 14 };

    date['day'] === date.day
      //=> true

Expressions can be used for keys as well. The syntax is to enclose the key's expression in `[` and `]`:

    {
      ["p" + "i"]: 3.14159265
    }
      //=> {"pi":3.14159265}

All containers can contain any value, including functions or other containers, like a fat arrow function:

    const Mathematics = {
      abs: (a) => a < 0 ? -a : a
    };

    Mathematics.abs(-5)
      //=> 5

Or proper functions:

    const SecretDecoderRing = {
      encode: function (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode: function (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }

Or named function expressions:

    const SecretDecoderRing = {
      encode: function encode (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode: function decode (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }

It is very common to associate named function expressions with keys in objects, and there is a "compact method syntax" for binding named function expressions to keywords:

    const SecretDecoderRing = {
      encode (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }

(There are some other technical differences between binding a named function expression and using compact method syntax, but they are not relevant here. We will generally prefer compact method syntax whenever we can.)

### destructuring objects

Just as we saw with arrays, we can write destructuring assignments with literal object syntax. So, we can write:

{:lang="javascript"}
~~~~~~~~
const user = {
  name: { first: "Reginald",
          last: "Braithwaite"
        },
  occupation: { title: "Author",
                responsibilities: [ "JavaScript Allongé",
                                    "JavaScript Spessore",
                                    "CoffeeScript Ristretto"
                                  ]
              }
};

user.name.last
  //=> "Braithwaite"

user.occupation.title
  //=> "Author"
~~~~~~~~

And we can also write:

{:lang="javascript"}
~~~~~~~~
const {name: { first: given, last: surname}, occupation: { title: title } } = user;

surname
  //=> "Braithwaite"

title
  //=> "Author"
~~~~~~~~

And of course, we destructure parameters:

{:lang="javascript"}
~~~~~~~~
const description = ({name: { first: given }, occupation: { title: title } }) =>
  `${given} is a ${title}`;

description(user)
  //=> "Reginald is a Author"
~~~~~~~~

Terrible grammar and capitalization, but let's move on. It is very common to write things like `title: title` when destructuring objects. When the label is a valid variable name, it's often the most obvious variable name as well. So JavaScript supports a further syntactic optimization:

{:lang="javascript"}
~~~~~~~~
const description = ({name: { first }, occupation: { title } }) =>
  `${first} is a ${title}`;

description(user)
  //=> "Reginald is a Author"
~~~~~~~~

And that same syntax works for literals:

{:lang="javascript"}
~~~~~~~~
const abbrev = ({name: { first, last }, occupation: { title } }) => {
  return { first, last, title};
}

abbrev(user)
  //=> {"first":"Reginald","last":"Braithwaite","title":"Author"}
~~~~~~~~

### revisiting linked lists

Earlier, we used two-element arrays as nodes in a linked list:

{:lang="javascript"}
~~~~~~~~
const cons = (a, d) => [a, d],
      car  = ([a, d]) => a,
      cdr  = ([a, d]) => d;
~~~~~~~~

In essence, this simple implementation used functions to create an abstraction with named elements. But now that we've looked at objects, we can use an object instead of a two-element array. While we're at it, let's use contemporary names. So our linked list nodes will be formed from `{ first, rest }`

In that case, a linked list of the numbers `1`, `2`, and `3` will look like this: `{ first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY } } }`.

We can then perform the equivalent of `[first, ...rest]` with direct property accessors:

{:lang="javascript"}
~~~~~~~~
const EMPTY = {};
const OneTwoThree = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY } } };

OneTwoThree.first
  //=> 1

OneTwoThree.rest
  //=> {"first":2,"rest":{"first":3,"rest":{}}}

OneTwoThree.rest.rest.first
  //=> 3
~~~~~~~~

Taking the length of a linked list is easy:

{:lang="javascript"}
~~~~~~~~
const length = (node, delayed = 0) =>
  node === EMPTY
    ? delayed
    : length(node.rest, delayed + 1);

length(OneTwoThree)
  //=> 3
~~~~~~~~

What about mapping? Well, let's start with the simplest possible thing, making a *copy* of a list. As we saw above, and discussed in [Garbage, Garbage Everywhere](#garbage), it is fast to iterate forward through a linked list. What isn't fast is naïvely copying a list:

{:lang="javascript"}
~~~~~~~~
const slowcopy = (node) =>
  node === EMPTY
    ? EMPTY
    : { first: node.first, rest: slowcopy(node.rest)};

slowcopy(OneTwoThree)
  //=> {"first":1,"rest":{"first":2,"rest":{"first":3,"rest":{}}}}
~~~~~~~~

The problem here is that linked lists are constructed back-to-front, but we iterate over them front-to-back. So to copy a list, we have to save all the bits on the call stack and then construct the list from back-to-front as all the recursive calls return.

We could follow the strategy of delaying the work. Let's write that naively:

{:lang="javascript"}
~~~~~~~~
const copy2 = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : copy2(node.rest, { first: node.first, rest: delayed });

copy2(OneTwoThree)
  //=> {"first":3,"rest":{"first":2,"rest":{"first":1,"rest":{}}}}
~~~~~~~~

Well, well, well. We have unwittingly *reversed* the list. This makes sense, if lists are constructed from back to front, and we make a linked list out of items as we iterate through it, we're going to get a backwards copy of the list. This isn't a bad thing by any stretch of the imagination. Let's call it what it is:

{:lang="javascript"}
~~~~~~~~
const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(node.rest, { first: node.first, rest: delayed });
~~~~~~~~

And now, we can make a reversing map:

{:lang="javascript"}
~~~~~~~~
const reverseMapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverseMapWith(fn, node.rest, { first: fn(node.first), rest: delayed });

reverseMapWith((x) => x * x, OneTwoThree)
  //=> {"first":9,"rest":{"first":4,"rest":{"first":1,"rest":{}}}}
~~~~~~~~

And a regular `mapWith` follows:

{:lang="javascript"}
~~~~~~~~
const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(node.rest, { first: node.first, rest: delayed });

const mapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? reverse(delayed)
    : mapWith(fn, node.rest, { first: fn(node.first), rest: delayed });

mapWith((x) => x * x, OneTwoThree)
  //=> {"first":1,"rest":{"first":4,"rest":{"first":9,"rest":{}}}}
~~~~~~~~

Our `mapWith` function takes twice as long as a straight iteration, because it iterates over the entire list twice, once to map, and once to reverse the list. Likewise, it takes twice as much memory, because it constructs a reverse of the desired result before throwing it away.

Mind you, this is still much, much faster than making partial copies of arrays. For a list of length *n*, we created *n* superfluous nodes and copied *n* superfluous values. Whereas our naïve array algorithm created 2*n* superfluous arrays and copied *n*^2^ superfluous values.
