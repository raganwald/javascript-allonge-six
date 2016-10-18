## Arrays and Destructuring Arguments {#arraysanddestructuring}

While we have mentioned arrays briefly, we haven't had a close look at them. Arrays are JavaScript's "native" representation of lists. Strings are important because they represent writing. Lists are important because they represent ordered collections of things, and ordered collections are a fundamental abstraction for making sense of reality.

### array literals

JavaScript has a literal syntax for creating an array: The `[` and `]` characters. We can create an empty array:

    []
      //=> []

We can create an array with one or more *elements* by placing them between the brackets and separating the items with commas. Whitespace is optional:

    [1]
      //=> [1]

    [2, 3, 4]
      //=> [2,3,4]

Any expression will work:

    [ 2,
      3,
      2 + 2
    ]
      //=> [2,3,4]

Including an expression denoting another array:

    [[[[[]]]]]

This is an array with one element that is an array with one element that is an array with one element that is an array with one element that is an empty array. Although that seems like something nobody would ever construct, many students have worked with almost the exact same thing when they explored various means of constructing arithmetic from Set Theory.

Any expression will do, including names:

    const wrap = (something) => [something];

    wrap("lunch")
      //=> ["lunch"]

Array literals are expressions, and arrays are *reference types*. We can see that each time an array literal is evaluated, we get a new, distinct array, even if it contains the exact same elements:

    [] === []
      //=> false

    [2 + 2] === [2 + 2]
      //=> false

    const array_of_one = () => [1];

    array_of_one() === array_of_one()
      //=> false

### element references

Array elements can be extracted using `[` and `]` as postfix operators. We pass an integer as an index of the element to extract:

    const oneTwoThree = ["one", "two", "three"];

    oneTwoThree[0]
      //=> 'one'

    oneTwoThree[1]
      //=> 'two'

    oneTwoThree[2]
      //=> 'three'

As we can see, JavaScript Arrays are [zero-based].

[zero-based]: https://en.wikipedia.org/wiki/Zero-based_numbering

We know that every array is its own unique entity, with its own unique reference. What about the contents of an array? Does it store references to the things we give it? Or copies of some kind?

{:lang="javascript"}
~~~~~~~~
const x = [],
      a = [x];

a[0] === x
  //=> true, arrays store references to the things you put in them.
~~~~~~~~

### destructuring arrays

There is another way to extract elements from arrays: *Destructuring*, a feature going back to Common Lisp, if not before. We saw how to construct an array literal using `[`, expressions, `,` and `]`. Here's an example of an array literal that uses a name:

    const wrap = (something) => [something];

Let's expand it to use a block and an extra name:

    const wrap = (something) => {
      const wrapped = [something];

      return wrapped;
    }

    wrap("package")
      //=> ["package"]

The line `const wrapped = [something];` is interesting. On the left hand is a name to be bound, and on the right hand is an array literal, a template for constructing an array, very much like a quasi-literal string.

In JavaScript, we can actually *reverse* the statement and place the template on the left and a value on the right:

    const unwrap = (wrapped) => {
      const [something] = wrapped;

      return something;
    }

    unwrap(["present"])
      //=> "present"

The statement `const [something] = wrapped;` *destructures* the array represented by `wrapped`, binding the value of its single element to the name `something`. We can do the same thing with more than one element:

    const surname = (name) => {
      const [first, last] = name;

      return last;
    }

    surname(["Reginald", "Braithwaite"])
      //=> "Braithwaite"

We could do the same thing with `(name) => name[1]`, but destructuring is code that resembles the data it consumes, a valuable coding style.

Destructuring can nest:

    const description = (nameAndOccupation) => {
      const [[first, last], occupation] = nameAndOccupation;

      return `${first} is a ${occupation}`;
    }

    description([["Reginald", "Braithwaite"], "programmer"])
      //=> "Reginald is a programmer"

### gathering

Sometimes we need to extract arrays from arrays. Here is the most common pattern: Extracting the head and gathering everything but the head from an array:

    const [car, ...cdr] = [1, 2, 3, 4, 5];

    car
      //=> 1
    cdr
      //=> [2, 3, 4, 5]

[`car` and `cdr`](https://en.wikipedia.org/wiki/CAR_and_CDR) are archaic terms that go back to an implementation of Lisp running on the IBM 704 computer. Some other languages call them `first` and `butFirst`, or `head` and `tail`. We will use a common convention and call variables we gather `rest`, but refer to the `...` operation as a "gather," following Kyle Simpson's example.[^getify]

[^getify]: Kyle Simpson is the author of [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/README.md#you-dont-know-js-book-series), available [here](http://search.oreilly.com/?q=you+don%27t+know+js+kyle+simpson)

Alas, the `...` notation does not provide a universal patten-matching capability. For example, we cannot write

    const [...butLast, last] = [1, 2, 3, 4, 5];
      //=> ERROR

    const [first, ..., last] = [1, 2, 3, 4, 5];
      //=> ERROR

Now, when we introduced destructuring, we saw that it is kind-of-sort-of the reverse of array literals. So if

    const wrapped = [something];

Then:

    const [unwrapped] = something;

What is the reverse of gathering? We know that:

    const [car, ...cdr] = [1, 2, 3, 4, 5];

What is the reverse? It would be:

    const cons = [car, ...cdr];

Let's try it:

{:lang="js"}
~~~~~~~~
const oneTwoThree = ["one", "two", "three"];

["zero", ...oneTwoThree]
  //=> ["zero","one","two","three"]
~~~~~~~~

It works! We can use `...` to place the elements of an array inside another array. We say that using `...` to destructure is gathering, and using it in a literal to insert elements is called "spreading."

### destructuring is not pattern matching

Some other languages have something called *pattern matching*, where you can write something like a destructuring assignment, and the language decides whether the "patterns" matches at all. If it does, assignments are made where appropriate.

In such a language, if you wrote something like:

    const [what] = [];

That match would fail because the array doesn't have an element to assign to `what`. But this is not how JavaScript works. JavaScript tries its best to assign things, and if there isn't something that fits, JavaScript binds `undefined` to the name. Therefore:

    const [what] = [];

    what
      //=> undefined

    const [which, what, who] = ["duck feet", "tiger tail"];

    who
      //=> undefined

And if there aren't any items to assign with `...`, JavaScript assigns an empty array:

    const [...they] = [];

    they
      //=> []

    const [which, what, ...they] = ["duck feet", "tiger tail"];

    they
      //=> []

From its very inception, JavaScript has striven to avoid catastrophic errors. As a result, it often coerces values, passes `undefined` around, or does whatever it can to keep executing without failing. This often means that we must write our own code to detect failure conditions, as we cannot rely on the language to point out when we are doing semantically meaningless things.

### destructuring and return values

Some languages support multiple return values: A function can return several things at once, like a value and an error code. This can easily be emulated in JavaScript with destructuring:

    const description = (nameAndOccupation) => {
      if (nameAndOccupation.length < 2) {
        return ["", "occupation missing"]
      }
      else {
        const [[first, last], occupation] = nameAndOccupation;

        return [`${first} is a ${occupation}`, "ok"];
      }
    }

    const [reg, status] = description([["Reginald", "Braithwaite"], "programmer"]);

    reg
      //=> "Reginald is a programmer"

    status
       //=> "ok"

### destructuring parameters

Consider the way we pass arguments to parameters:

    foo()
    bar("smaug")
    baz(1, 2, 3)

It is very much like an array literal. And consider how we bind values to parameter names:

    const foo = () => ...
    const bar = (name) => ...
    const baz = (a, b, c) => ...

It *looks* like destructuring. It acts like destructuring. There is only one difference: We have not tried gathering. Let's do that:

    const numbers = (...nums) => nums;

    numbers(1, 2, 3, 4, 5)
      //=> [1,2,3,4,5]

    const headAndTail = (head, ...tail) => [head, tail];

    headAndTail(1, 2, 3, 4, 5)
      //=> [1,[2,3,4,5]]


Gathering works with parameters! This is very useful indeed, and we'll see more of it in a moment.[^rest]

[^rest]: Gathering in parameters has a long history, and the usual terms are to call gathering "pattern matching" and to call a name that is bound to gathered values a "rest parameter." The term "rest" is perfectly compatible with gather: "Rest" is the noun, and "gather" is the verb. We *gather* the *rest* of the parameters.
