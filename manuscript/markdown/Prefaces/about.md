## About JavaScript Allongé

JavaScript Allongé is a first and foremost, a book about *programming with functions*. It's written in JavaScript, because JavaScript hits the perfect sweet spot of being both widely used, and of having proper first-class functions with lexical scope. If those terms seem unfamiliar, don't worry: JavaScript Allongé takes great delight in explaining what they mean and why they matter.

*JavaScript Allongé* begins at the beginning, with values and expressions, and builds from there to discuss types, identity, functions, closures, scopes, collections, iterators, and many more subjects up to working with classes and instances.

It also provides recipes for using functions to write software that is simpler, cleaner, and less complicated than alternative approaches that are object-centric or code-centric. JavaScript idioms like function combinators and decorators leverage JavaScript's power to make code easier to read, modify, debug and refactor.

*JavaScript Allongé* teaches you how to handle complex code, and it also teaches you how to simplify code without dumbing it down. As a result, *JavaScript Allongé* is a rich read releasing many of JavaScript's subtleties, much like the Café Allongé beloved by coffee enthusiasts everywhere.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript

### why the "six" edition?

ECMAScript 2015 (formerly called ECMAScript 6 or "ES6"), is ushering in a very large number of improvements to the way programmers can write small, powerful components and combine them into larger, fully featured programs. Features like destructuring, block-structured variables, iterables, generators, and the class keyword are poised to make JavaScript programming more expressive.

Prior to ECMAScript 2015, JavaScript did not include many features that programmers have discovered are vital to writing great software. For example, JavaScript did not include block-structured variables. Over time, programmers discovered ways to roll their own versions of important features.

For example, block-structured languages allow us to write:

    for (int i = 0; i < array.length; ++i) {
      // ...
    }

And the variable `i` is scoped locally to the code within the braces. Prior to ECMAScript 2015, JavaScript did not support block-structuring, so programmers borrowed a trick from the Scheme programming language, and would write:

    var i;

    for (i = 0; i < array.length; ++i) {
      (function (i) {
        // ...
      })(i)
    }

To create the same scoping with an Immediately Invoked Function Expression, or "IIFE."

Likewise, many programming languages permit functions to have a variable number of arguments, and to collect the arguments into a single variable as an array. In Ruby, we can write:

    def foo (first, *rest)
      # ...
    end

Prior to ECMAScript 2015, JavaScript did not support collecting a variable number of arguments into a parameter, so programmers would take advantage of an awkward work-around and write things like:

    function foo () {
      var first = arguments[0],
          rest  = [].slice.call(arguments, 1);

      // ...
    }

The first edition of JavaScript Allongé explained these and many other patterns for writing flexible and composable programs in JavaScript, but the intention wasn't to explain how to work around JavaScript's missing features: The intention was to explain why the style of programming exemplified by the missing features is important.

Working around the missing features was a necessary evil.

But now, JavaScript is gaining many important features, in part because the governing body behind JavaScript has observed that programmers are constantly working around the same set of limitations. With ECMASCript 2015, we can write:

    for (let i = 0; i < array.length; ++i) {
      // ...
    }

And `i` is scoped to the for loop. We can also write:

    function foo (first, ...rest) {
      // ...
    }

And presto, `rest` collects the rest of the arguments without a lot of malarky involving slicing `arguments`. Not having to work around these kinds of missing features makes JavaScript Allongé a *better book*, because it can focus on the *why* to do something and *when* to do it, instead of on the how to make it work

JavaScript Allongé, The "Six" Edition packs all the goodness of JavaScript Allongé into a new, updated package that is relevant for programmers working with (or planning to work with) the latest version of JavaScript.

### that's nice. is that the only reason?

Actually, no.

If it were just a matter of updating the syntax, the original version of JavaScript Allongé could have simply iterated, slowly replacing old syntax with new. It would have continued to say much the same things, only with new syntax.

*But there's more to it than that*. The original JavaScript Allongé was not just written to teach JavaScript: It was written to describe certain ideas in programming: Working with small, independent entities that compose together to make bigger programs. Thus, the focus on things like writing decorators.

As noted above, JavaScript was chosen as the language for Allongé because it hit a sweet spot of having a large audience of programmers and having certain language features that happen to work well with this style of programming.

ECMAScript 2015 does more than simply update the language with some simpler syntax for a few things and help us avoid warts. It makes a number of interesting programming techniques easy to explain and easy to use. And these techniques dovetail nicely with Allongé's focus on composing entities and working with functions.

Thus, the "six" edition introduces [classes](#classes) and [mixins](#mixins). It introduces the notion of implementing private properties with [symbols](#symbols). It introduces [iterators and generators](#collections). But the common thread that runs through all these things is that since they are all simple objects and simple functions, we can use the same set of "programming with functions" techniques to build programs by composing small, flexible, and decoupled entities.

We just call some of those functions [constructors](#new), others [decorators](#decorators), others [functional mixins](#functional-mixins), and yet others, [policies](#policies).

Introducing so many new ideas did require a major rethink of the way the book was organized. And introducing these new ideas did add substantially to its bulk. But even so, in a way it is still explaining the exact same original idea that programs are built out of small, flexible functions composed together.
