# Decaffeinated: Impostors

![Decaf espresso](images/decaf-espresso.jpg)

Now that we've explored objects in some depth, it's time to acknowledge something that even small children know: Everything in JavaScript behaves like an object, everything in JavaScript behaves like an instance of a function, and therefore everything in JavaScript behaves as if it inherits some methods from a prototype and/or has some elements of its own.

For example:

    3.14159265.toPrecision(5)
      //=> '3.1415'
      
    'FORTRAN, SNOBOL, LISP, BASIC'.split(', ')
      //=> [ 'FORTRAN',
      #     'SNOBOL',
      #     'LISP',
      #     'BASIC' ]
      
    [ 'FORTRAN',
      'SNOBOL',
      'LISP',
      'BASIC' ].length
    //=> 4
    
Functions themselves are instances, and they have methods. For example, every function has a method `call`. `call`'s first argument is a *context*: When you invoke `.call` on a function, it invoked the function, setting `this` to the context. It passes the remainder of the arguments to the function. It seems like objects are everywhere in JavaScript!

You may have noticed that we use "weasel words" to describe how everything in JavaScript *behaves like* an object. Everything *behaves as if* it delegates behaviour to a prototype.

The full explanation is this: As you know, JavaScript has "value types" like `String`, `Number`, and `Boolean`. As noted in the first chapter, value types are also called *primitives*, and one consequence of the way JavaScript implements primitives is that they aren't objects. Which means they can be identical to other values of the same type with the same contents, but the consequence of certain design decisions is that value types don't actually have methods or prototypes.

So. Value types don't have methods or prototypes. And yet:

    "Spence Olham".split(' ')
      //=> ["Spence", "Olham"]

Somehow, when we write `"Spence Olham".split(' ')`, the string `"Spence Olham"` isn't an object, it doesn't have methods, but it does a damn fine job of impersonating an object with a `String` prototype. How does `"Spence Olham"` impersonate an object?

JavaScript pulls some legerdemain. When you do something that treats a value like an object, JavaScript checks to see whether the value actually is an object. If the value is actually a primitive,[^reminder] JavaScript temporarily makes an object that is a kinda-sorta copy of the primitive and that kinda-sorta copy has methods and you are temporarily fooled into thinking that `"Spence Olham"` has a `.split` method.

[^reminder]: Recall that Strings, Numbers, Booleans and so forth are value types and primitives. We're calling them primitives here.

These kinda-sorta copies are called String *instances* as opposed to String *primitives*. And the instances have methods, while the primitives do not. How does JavaScript make an instance out of a primitive? With `new`, of course.[^later] Let's try it:

[^later]: We'll read all about the `new` keyword in [COnstructors and `new`](#new).

    new String("Spence Olham")
      //=> "Spence Olham"
      
The string instance looks just like our string primitive. But does it behave like a string primitive? Not entirely:

    new String("Spence Olham") === "Spence Olham"
      //=> false
      
Aha! It's an object with its own identity, unlike string primitives that behave as if they have a canonical representation. If we didn't care about their identity, that wouldn't be a problem. But if we carelessly used a string instance where we thought we had a string primitive, we could run into a subtle bug:

    if (userName === "Spence Olham") {
      getMarried();
      goCamping()
    }
      
That code is not going to work as we expect should we accidentally bind `new String("Spence Olham")` to `userName` instead of the primitive `"Spence Olham"`.

This basic issue that instances have unique identities but primitives with the same contents have the same identities--is true of all primitive types, including numbers and booleans: If you create an instance of anything with `new`, it gets its own identity.

There are more pitfalls to beware. Consider the truthiness of string, number and boolean primitives:

    '' ? 'truthy' : 'falsy'
      //=> 'falsy'
    0 ? 'truthy' : 'falsy'
      //=> 'falsy'
    false ? 'truthy' : 'falsy'
      //=> 'falsy'
      
Compare this to their corresponding instances:

    new String('') ? 'truthy' : 'falsy'
      //=> 'truthy'
    new Number(0) ? 'truthy' : 'falsy'
      //=> 'truthy'
    new Boolean(false) ? 'truthy' : 'falsy'
      //=> 'truthy'
      
Our notion of "truthiness" and "falsiness" is that all instances are truthy, even string, number, and boolean instances corresponding to primitives that are falsy.

There is one sure cure for "JavaScript Impostor Syndrome." Just as `new PrimitiveType(...)` creates an instance that is an impostor of a primitive, `PrimitiveType(...)` creates an original, canonicalized primitive from a primitive or an instance of a primitive object.

For example:

    String(new String("Spence Olham")) === "Spence Olham"
      //=> true
      
Getting clever, we can write this:

    const original = function (unknown) {
      return unknown.constructor(unknown)
    }
        
    original(true) === true
      //=> true
    original(new Boolean(true)) === true
      //=> true
      
Of course, `original` will not work for your own creations unless you take great care to emulate the same behaviour. But it does work for strings, numbers, and booleans.