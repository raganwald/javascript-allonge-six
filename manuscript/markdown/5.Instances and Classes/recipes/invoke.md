## Invoke {#invoke}

[Send](#send) is useful when invoking a function that's a member of an object (or of an instance's prototype). But we sometimes want to invoke a function that is designed to be executed within an object's context. This happens most often when we want  to "borrow" a method from one "class" and use it on another object.

It's not an unprecedented use case. The Ruby programming language has a handy feature called [instance_exec]. It lets you execute an arbitrary block of code in the context of any object. Does this sound familiar? JavaScript has this exact feature, we just call it `.apply` (or `.call` as the case may be). We can execute any function in the context of any arbitrary object.

[instance_exec]: http://www.ruby-doc.org/core-1.8.7/Object.html#method-i-instance_exec

The only trouble with `.apply` is that being a method, it doesn't compose nicely with other functions like combinators. So, we create a function that allows us to use it as a combinator:

    const invoke = (fn, ...args) =>
      instance => fn.apply(instance, args);

For example, let's say someone else's code gives you an array of objects that are in part, but not entirely like arrays. Something like:

    const data = [
      { 0: 'zero', 
        1: 'one', 
        2: 'two',
        length: 3},
      { 0: 'none',
        length: 1 },
      // ...
    ];
    
If they were arrays, and we wanted to copy them, we would use:

    mapWith(send('slice', 0))(data)
  
Because arrays have a `.send` method. But our quasi-arrays have no such thing. So... We want to borrow the `.slice` method from arrays, but have it work on our data. `invoke([].slice, 0)` does the trick:

    mapWith(invoke([].slice, 0))(data)
      //=> [
             ["zero","one","two"],
             ["none"],
             // ...
           ]

### instance eval

`invoke` is useful when you have the function and are looking for the instance. It can be written "the other way around," for when you have the instance and are looking for the function:

    const instanceEval = instance =>
      (fn, ...args) => fn.apply(instance, args);