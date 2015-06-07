## Tap {#tap}

One of the most basic combinators is the "K Combinator," nicknamed the "Kestrel:"

    const K = (x) => (y) => x;

It has some surprising applications. One is when you want to do something with a value for side-effects, but keep the value around. Behold:

    const tap = (value) =>
      (fn) => (
        typeof(fn) === 'function' && fn(value),
        value
      )

`tap` is a traditional name borrowed from various Unix shell commands. It takes a value and returns a function that always returns the value, but if you pass it a function, it executes the function for side-effects. Let's see it in action as a poor-man's debugger:

    tap('espresso')((it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    

It's easy to turn off:

    tap('espresso')();
      //=> 'espresso'

Libraries like [Underscore] use a version of `tap` that is "uncurried:"

    _.tap('espresso', (it) =>
      console.log(`Our drink is '${it}'`) 
    );
      //=> Our drink is 'espresso'
           'espresso'
    
Let's enhance our recipe so that it works both ways:

    const tap = (value, fn) => {
      const curried = (fn) => (
          typeof(fn) === 'function' && fn(value),
          value
        );
      
      return fn === undefined
             ? curried
             : curried(fn);
    }

Now we can write:

    tap('espresso')((it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    
Or:

    tap('espresso', (it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    
And if we wish it to do nothing at all, We can write either `tap('espresso')()` or `tap('espresso', null)`

p.s. `tap` can do more than just act as a debugging aid. It's also useful for working with [object and instance methods](#tap-methods).

[Underscore]: http://underscorejs.org
