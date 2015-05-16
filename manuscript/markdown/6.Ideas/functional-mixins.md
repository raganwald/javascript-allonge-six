## Mixins {#functional-mixins}

In [Extending Classes with Mixins](#classes-and-mixins), we saw that you can emulate "mixins" using `Object.assign` on classes. We'll revisit this subject now and spend more time looking at mixing functionality into classes.

First, a quick recap: In JavaScript, a "class" is implemented as a constructor function and its prototype, whether you write it directly, or use the `class` keyword. Instances of the class are created by calling the constructor with `new`. They "inherit" shared behaviour from the constructor's `prototype` property.

One way to share behaviour scattered across multiple classes, or to untangle behaviour by factoring it out of an overweight prototype, is to extend a prototype with a mixin.

Here's another class of todo items:

    class Todo {
	  	constructor (name) {
      	this.name = name || 'Untitled';
      	this.done = false;
    	}
    	do () {
      	this.done = true;
				return this;
    	}
    	undo () {
      	this.done = false;
				return this;
			}
		}

And a "mixin:"

    const ColourCoded = {
      setColourRGB: function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
				return this;
      },
      getColourRGB: function () {
        return this.colourCode;
      }
    };
    
Mixing colour coding into our Todo prototype is straightforward:

    Object.assign(Todo.prototype, ColourCoded);
    
    new Todo('test')
			.setColourRGB(1, 2, 3)
			//=> {"name":"test","done":false,"colourCode":{"r":1,"g":2,"b":3}}
    
### what is a "mixin?"

Like "class," the word "mixin" means different things to different people. A Ruby user will talk about modules, for example. And a JavaScript user could in truth say that everything is an object and we're just extending one object (that happens to be a prototype) with the properties of another object (that just happens to contain some functions).

A simple definition that works for most purposes is to define a mixin as: *A collection of behaviour that can be added to a class's existing prototype*. `ColourCoded` above is such a mixin. If we had to actually assign a new prototype to the `Todo` class, that wouldn't be mixing functionality in, that would be replacing functionality.

### functional mixins

The mixin we have above works properly, but our little recipe had two distinct steps: Define the mixin and then extend the class prototype. Angus Croll pointed out that it's far more elegant to define a mixin as a function rather than an object. He calls this a [functional mixin][fm]. Here's our `ColourCoded` recast in functional form:

    const becomeColourCoded = (clazz) =>
      Object.assign(Todo.prototype, {
	      setColourRGB: function (r, g, b) {
	        this.colourCode = { r: r, g: g, b: b };
					return this;
	      },
	      getColourRGB: function () {
	        return this.colourCode;
	      }
			})
    
    becomeColourCoded(Todo);
		
The challenge with this approach is that it only works with constructor functions (and "classes," since they desugar to constructor functions). If we mix functionality directly into a target, we could mix functionality directly into an object if we so chose:

    const becomeColourCoded = (behaviour) =>
      Object.assign(behaviour, {
	      setColourRGB: function (r, g, b) {
	        this.colourCode = { r: r, g: g, b: b };
					return this;
	      },
	      getColourRGB: function () {
	        return this.colourCode;
	      }
			})
    
    becomeColourCoded(Todo.prototype);
		
Either way, we can make ourselves a convenience function (that also names the pattern):

		const fClassMixin = (mixin) =>
			clazz => Object.assign(clazz.prototype, mixin);
			
or:

		const fMixin = (mixin) =>
			behaviour => Object.assign(behaviour, mixin);
			
This allows us to define functional mixins neatly:

		const becomeColourCoded = fMixin({
      setColourRGB: function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
				return this;
      },
      getColourRGB: function () {
        return this.colourCode;
      }
		});
			
### taking flight

Twitter's [Flight] framework uses a variation on this technique that targets the mixin function's context:

    function asColourCoded () {
      return Object.assign(this, {
	      setColourRGB: function (r, g, b) {
	        this.colourCode = { r: r, g: g, b: b };
					return this;
	      },
	      getColourRGB: function () {
	        return this.colourCode;
	      }
			});
    };
    
    asColourCoded.call(Todo.prototype);
    
This approach has some subtle benefits: You can use mixins as methods, for example. It's possible to write a context-agnostic functional mixin:

    function colourCoded () {
      if (arguments[0] !== void 0) {
        return colourCoded.call(arguments[0]);
      }
      this.setColourRGB = fluent( function (r, g, b) {
        this.colourCode = { r: r, g: g, b: b };
      });
      
      this.getColourRGB = function () {
        return this.colourCode;
      };
      
      return this;
    };

Bueno!

[fm]: https://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/ "A fresh look at JavaScript Mixins"
[Flight]: http://flightjs.github.io/
