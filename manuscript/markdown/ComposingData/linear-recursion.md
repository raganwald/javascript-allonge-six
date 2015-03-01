## Self-Similarity {#linear-recursion}

> Recursion is the root of computation since it trades description for time.—Alan Perlis, [Epigrams in Programming](http://www.cs.yale.edu/homes/perlis-alan/quotes.html)

In [Arrays and Destructuring Arguments](#arraysanddestructuring), we worked with the basic idea that putting an array together with a literal array expression was the reverse or opposite of taking it apart with a destructuring assignment.

We saw that the basic idea that putting an array together with a literal array expression was the reverse or opposite of taking it apart with a destructuring assignment.

Let's be more specific. Some data structures, like lists, can obviously be seen as a collection of items. Some are empty, some have three items, some forty-two, some contain numbers, some contain strings, some a mixture of elements, there are all kinds of lists.

But we can also define a list by describing a rule for building lists. One of the simplest, and longest-standing in computer science, is to say that a list is:

0. Empty, or;
1. Consists of an element concatenated with a list .

Let's convert our rules to array literals. The first rule is simple: `[]` is a list. How about the second rule? We can express that using a spread. Given an element `e` and a list `list`, `[e, ...list]` is a list. We can test this manually by building up a list:

    []
    //=> []

    ["baz", ...[]]
    //=> ["baz"]

    ["bar", ...["baz"]]
    //=> ["bar","baz"]

    ["foo", ...["bar", "baz"]]
    //=> ["foo","bar","baz"]
  
Thanks to the parallel between array literals + spreads with destructuring + rests, we can also use the same rules to decompose lists:

    
    const [first, ...rest] = [];
    first
      //=> undefined
    rest
      //=> []:

    const [first, ...rest] = ["foo"];
    first
      //=> "foo"
    rest
      //=> []

    const [first, ...rest] = ["foo", "bar"];
    first
      //=> "foo"
    rest
      //=> ["bar"]

    const [first, ...rest] = ["foo", "bar", "baz"];
    first
      //=> "foo"
    rest
      //=> ["bar","baz"]
    

For the purpose of this exploration, we will presume the following:[^wellactually]

    
    const isEmpty = ([first, ...rest]) => first === undefined;

    isEmpty([])
      //=> true

    isEmpty([0])
      //=> false

    isEmpty([[]])
      //=> false
    
    
[^wellactually]: Well, actually, this does not work for arrays that contain `undefined` as a value, but we are not going to see that in our examples. A more robust implementation would be `(array) => array.length === 0`, but we are doing backflips to keep this within a very small and contrived playground.
    
Armed with our definition of an empty list and with what we've already learned, we can build a great many functions that operate on arrays. We know that we can get the length of an array using its `.length`. But as an exercise, how would we write a `length` function using just what we have already?

First, we pick what we call a *terminal case*. What is the length of an empty array? `0`. So let's start our function with the observation that if an array is empty, the length is `0`:

    
    const length = ([first, ...rest]) =>
      first === undefined
        ? 0
        : // ???
    
      
We need something for when the array isn't empty. If an array is not empty, and we break it into two pieces, `first` and `rest`, the length of our array is going to be `length(first) + length(rest)`. Well, the length of `first` is `1`, there's just one element at the front. But we don't know the length of `rest`. If only there was a function we could call... Like `length`!

    
    const length = ([first, ...rest]) =>
      first === undefined
        ? 0
        : 1 + length(rest);
    
    
Let's try it!

    
    length([])
      //=> 0
  
    length(["foo"])
      //=> 1
  
    length(["foo", "bar", "baz"])
      //=> 3
    
      
Our `length` function is *recursive*, it calls itself. This makes sense because our definition of a list is recursive, and if a list is self-similar, it is natural to create an algorithm that is also self-similar.

### linear recursion

"Recursion" sometimes seems like an elaborate party trick. There's even a joke about this:

> When promising students are trying to choose between pure mathematics and applied engineering, they are given a two-part aptitude test. In the first part, they are led to a laboratory bench and told to follow the instructions printed on the card. They find a bunsen burner, a sparker, a tap, an empty beaker, a stand, and a card with the instructions "boil water."

> Of course, all the students know what to do: They fill the beaker with water, place the stand on the burner and the beaker on the stand, then they turn the burner on and use the sparker to ignite the flame. After a bit the water boils, and they turn off the burner and are lead to a second bench.

> Once again, there is a card that reads, "boil water." But this time, the beaker is on the stand over the burner, as left behind by the previous student. The engineers light the burner immediately. Whereas the mathematicians take the beaker off the stand and empty it, thus reducing the situation to a problem they have already solved.

There is more to recursive solutions that simply functions that invoke themselves. Recursive algorithms follow the "divide and conquer" strategy for solving a problem:

0. Divide the problem into smaller problems
0. If a smaller problem is solvable, solve the small problem
0. If a smaller problem is not solvable, divide and conquer that problem
0. When all small problems have been solved, compose the solutions into one big solution

The big elements of divide and conquer are a method for decomposing a problem into smaller problems, a test for the smallest possible problem, and a means of putting the pieces back together. Our solutions are a little simpler in that we don't really break a problem down into multiple pieces, we break a piece off the problem that may or may not be solvable, and solve that before sticking it onto a solution for the rest of the problem.

This simpler form of "divide and conquer" is called *linear recursion*. It's very useful and simple to understand. Let's take another example. Sometimes we want to *flatten* an array, that is, an array of arrays needs to be turned into one array of elements that aren't arrays.[^unfold]

[^unfold]: `flatten` is a very simple [unfold](https://en.wikipedia.org/wiki/Anamorphism), a function that takes a seed value and turns it into an array. Unfolds can be thought of a "path" through a data structure, and flattening a tree is equivalent to a depth-first traverse.

We already know how to divide arrays into smaller pieces. How do we decide whether a smaller problem is solvable? We need a test for the terminal case. Happily, there is something along these lines provided for us:

    Array.isArray("foo")
      //=> false
      
    Array.isArray(["foo"])
      //=> true
      
The usual "terminal case" will be that flattening an empty array will produce an empty array. The next terminal case is that if an element isn't an array, we don't flatten it, and can put it together with the rest of our solution directly. Whereas if an element is an array, we'll flatten it and put it together with the rest of our solution.

So our first cut at a `flatten` function will look like this:

    const flatten = ([first, ...rest]) => {
      if (first === undefined) {
        return [];
      }
      else if (!Array.isArray(first)) {
        return [first, ...flatten(rest)];
      }
      else {
        return [...flatten(first), ...flatten(rest)];
      }
    }
    
    flatten(["foo", [3, 4, []]])
      //=> ["foo",3,4]
      
Once again, the solution directly displays the important elements: Dividing a problem into subproblems, detecting terminal cases, solving the terminal cases, and composing a solution from the solved portions.

### mapping {#mapping}

Another common problem is applying a function to every element of an array. JavaScript has a built-in function for this, but let's write our own using linear recursion.

If we want to square each number in a list, we could write:

    const squareAll = ([first, ...rest]) => first === undefined
                                                ? []
                                                : [first * first, ...squareAll(rest)];
                                                
    squareAll([1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]

And if we wanted to "truthify" each element in a list, we could write:

    const truthyAll = ([first, ...rest]) => first === undefined
                                                ? []
                                                : [!!first, ...truthyAll(rest)];

    truthyAll([null, true, 25, false, "foo"])
      //=> [false,true,true,false,true]
                                                
This specific case of linear recursion is called "mapping," and it is not necessary to constantly write out the same pattern again and again. Functions can take functions as arguments, so let's "extract" the thing to do to each element and separate it from the business of taking an array apart, doing the thing, and putting the array back together.

Given the signature:

    const mapWith = (fn, array) => // ...
    
We can write it out using a ternary operator. Even in this small function, we can identify the terminal condition, the piece being broken off, and recomposing the solution.

    const mapWith = (fn, [first, ...rest]) =>
      first === undefined
        ? []
        : [fn(first), ...mapWith(fn, rest)];
                                                  
    mapWith((x) => x * x, [1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
      
    mapWith((x) => !!x, [null, true, 25, false, "foo"])
      //=> [false,true,true,false,true]

### folding

With the exception of the `length` example at the beginning, our examples so far all involve rebuilding a solution using spreads.  But they needn't. A function to compute the sum of the squares of a list of numbers might look like this:

    const sumSquares = ([first, ...rest]) => first === undefined
                                             ? 0
                                             : first * first + sumSquares(rest);
                                             
    sumSquares([1, 2, 3, 4, 5])
      //=> 55

There are two differences between `sumSquares` and our maps above:

0. Given the terminal case of an empty list, we return a `0` instead of an empty list, and;
0. We catenate the square of each element to the result of applying `sumSquares` to the rest of the elements.

Let's rewrite `mapWith` so that we can use it to sum squares.

    const foldWith = (fn, terminalValue, [first, ...rest]) =>
      first === undefined
        ? terminalValue
        : fn(first, foldWith(fn, terminalValue, rest));
                                                           
And now we supply a function that does slightly more than our mapping functions:

    foldWith((number, rest) => number * number + rest, 0, [1, 2, 3, 4, 5])
      //=> 55

Our `foldWith` function is a generalization of our `mapWith` function. We can represent a map as a fold, we just need to supply the array rebuilding code:

    const squareAll = (array) => foldWith((first, rest) => [first * first, ...rest], [], array);
    
    squareAll([1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]

And if we like, we can write `mapWith` using `foldWith`:

    const mapWith = (fn, array) => foldWith((first, rest) => [fn(first), ...rest], [], array),
          squareAll = (array) => mapWith((x) => x * x, array);
    
    squareAll([1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
          
And to return to our first example, our version of `length` can be written as a fold:

    const length = (array) => foldWith((first, rest) => 1 + rest, 0, array);
    
    length([1, 2, 3, 4, 5])
      //=> 5
          
### summary

Linear recursion is a basic building block of algorithms. Its basic form parallels the way linear data structures like lists are constructed: This helps make it understandable. Its specialized cases of mapping and folding are especially useful and can be used to build other functions. And finally, while folding is a special case of linear recursion, mapping is a special case of folding.
    