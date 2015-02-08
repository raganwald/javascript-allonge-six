## Plain Old JavaScript Objects {#pojos}

Lists are not the only way to represent collections of things, but they are the "oldest" data structure in the history of high level languages, because they map very closely to the way the hardware is organized in a computer. List are obviously very handy for homogeneous collections of things, like a shopping list:

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

Java has dictionaries, and it calls them "objects." The word "object" is loaded in programming circles, due to the widespread use of the term "object-oriented programming" that was coined by Alan Kay but has since come to mean many, many things to many different people.

In JavaScript, an object[^pojo] is a map from string keys to values.

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

    { year: 2012, month: 6, day: 14 }['day'] ===
        { year: 2012, month: 6, day: 14 }.day
      //=> true

All containers can contain any value, including functions or other containers:

    const Mathematics = {
      abs: function (a) {
             return a < 0 ? -a : a
           }
    };

    Mathematics.abs(-5)
      //=> 5

Funny we should mention `Mathematics`. If you recall, JavaScript provides a global environment that contains some existing objects that have handy functions you can use. One of them is called `Math`, and it contains functions for `abs`, `max`, `min`, and many others. Since it is always available, you can use it in any environment provided you don't shadow `Math`.

    Math.abs(-5)
      //=> 5
      
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