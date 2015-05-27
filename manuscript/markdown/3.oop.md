# The Coffee Factory: "Object-Oriented Programming"

Programming with objects and classes began in Norway in the late 1960s with the [Simula] programming language. Its creators, Ole-Johan Dahl and Kristen Nygaard, did not use those words to describe what would eventually become the dominant paradigm in computing.

A decade later, Dr. Alan Kay coined the phrase "Object-Oriented Programming" along with co-creating the [Smalltalk] programming language. He has famously said that to him, "OOP" was objects communicating with each other using messages, and that other languages copied the things that didn't matter from Smalltalk, and ignored the things he thought did matter.

[Simula]: https://en.wikipedia.org/wiki/Simula
[Smalltalk]: https://en.wikipedia.org/wiki/Smalltalk

Since that time, languages have either bolted object-ish ideas on top of their existing paradigms (like [Object Pascal] and [OCaml]), baked them in alongside other paradigms (like JavaScript), or embraced objects wholeheartedly.

[Object Pascal]: https://en.wikipedia.org/wiki/Object_Pascal
[OCaml]: https://en.wikipedia.org/wiki/OCaml
[C++]: https://en.wikipedia.org/wiki/C%2B%2B

That being said, there really is no one definition of "object-oriented." For one thing, there is no one definition of "object."

### objects

Some languages, like Smalltalk and [Ruby], treat an object as a fully encapsulated entity. There is no access to an object's private state, all you can do is invoke one of its methods. Other languages, like Java, permit objects to access each other's  state.

[Ruby]: https://en.wikipedia.org/wiki/Ruby_(programming_language)

Some languages (again, like Java) have very rigid objects and classes, it is impossible or awkward to add new methods or properties to objects at run time. Some are flexible about adding methods and properties at run time. And yet other languages treat objects as dictionaries, where properties and even methods can be added, modified, or removed with abandon.

So we can see that the concept of "object" is flexible across languages.

### classes

The concept of "class" is also flexible across languages. Object-oriented languages do not uniformly agree on whether classes are necessary, much less how they work. For example, The Common Lisp Object System defines behaviour with classes, and it also defines behaviour with generic functions. The [Self] and [NewtonScript] languages have prototypes instead of classes.

So some "OO" languages have objects, but not classes.

[Self]: https://en.wikipedia.org/wiki/Self_(programming_language)
[NewtonScript]: https://en.wikipedia.org/wiki/NewtonScript

C++ has classes, but they are not "first-class entities." You can't assign a class to a variable or pass it to a function. You can, however, manipulate the constructors for classes, the functions that make new objects. But you can't manipulate those constructors to change the behaviour of objects that have already been constructed, instance behaviour is early-bound by default.

Ruby has classes, and they're first-class entities. You can ask an object for its class, you can put a class in a variable, pass it to a method, or return it from a method, just like every other entity in the language. Classes in Ruby and Smalltalk even have their own class, they are instances of `Class`![^meta] Instance behaviour is late-bound and open for extension.[^mp]

[^mp]: Abuse of this feature by extending the behaviour of built-in classes is a controversial topic.

[^meta]: If the class of a class is `Class`, what class is the class of `Class`? In Ruby, `Class.class == Class`. In Smalltalk, it is `MetaClass`, which opens up the possibility for changing the way classes behave in a deep way.

### constructors

Some languages allow programs to construct objects independently, others (notably those that are heavily class-centric) require that objects always be constructed by their classes. Some languages allow any function or method to be used as a constructor, others require a special syntax or declaration for constructors.

### prototypes are not classes

Prototypical languages like Self and NewtonScript eschew classes altogether, using *prototypes* to define common behaviour for a set of objects. The difference between a prototype and a class is similar to the difference between a model home and a blueprint for a home.

You can say to a builder, "make me a home just like that model home," and the builder makes you a home that has a lot in common with the model home. You then decorate your home with additional personalization. But the model home is, itself, a home. Although you may choose to keep it empty, you could in principle move a family into it. This is different than asking a builder to make you a home based on a blueprint. The blueprint may specify the features of the home, but it isn't a home. It could never be used as a home.

Prototypes are like model homes, and classes are like blueprints. Classes are not like the objects they describe.[^wellactually]

[^wellactually]: Well, *actually*, the difference between prototypes and classes is like the difference between model homes and blueprints. But prototypes are not like model homes. In actual fact, the relationship between an object and its prototype is one of *delegation*. So if a model home had a kitchen, and you asked the builder to make you a home using the model as a prototype, you could customize your own kitchen. But if you didn't want to have your own custom kitchen, you would just use the model home's kitchen to do all your own cooking. The relationship between a model home and a house is sometimes described as [concatenative inheritance](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a), and JavaScript lets you do that too.

### "object-oriented programming" can mean almost anything

From this whirlwind tour of "object-oriented programming," we can see that the ideas behind "object-oriented programming" have some common roots in the history of programming languages, but each language implements its own particular flavour in its own particular way.

Thus, when we talk about "objects" and "prototypes" and "classes" in JavaScript, we're talking about objects, prototypes, and classes *as implemented in JavaScript*. And we must keep in mind that other languages can have a radically different take on these ideas.

### the javascript approach

JavaScript has objects, and by default, those objects are dictionaries. By default, objects directly manipulate each other's state. Methods can be added to, or removed from objects at run time. 

JavaScript has optional prototypes. Prototypes are objects in the same sense that model homes are homes. 

In JavaScript, object and array literals construct objects that delegate behaviour to the standard library's object prototype and array prototype, respectively. JavaScript also supports using `Object.create` to construct objects with or without a prototype, and `new` to construct objects using a constructor function.

Using prototypes and constructor functions, JavaScript programs can emulate many of the features of classes in other languages. JavaScript also has a `class` keyword that provides syntactic sugar for writing constructor functions and prototypes in a declarative fashion.

By default, a JavaScript class is a constructor composed with an object as its associated prototype. This can be denoted with the `class` keyword, by working with a function's default `.prototype` property, or by composing functions and objects independently.

JavaScript classes are constructors, but they are more than C++ constructors, in that manipulation of their prototype extends or modifies the behaviour of the instances they create. JavaScript classes take a minimalist approach to OO in the same sense that JavaScript objects take a minimal approach to OO. For example, behaviour can be mixed into an object, a prototype, or a class using the exact same mechanism, because objects, prototypes, and a constructor's prototype are all objects that are open to extension.

In sum, JavaScript is not exactly like any other object-oriented programming language, and its classes aren't like any other language that features classes, but then again, neither is any other object-oriented programming language, and neither are any other classes.