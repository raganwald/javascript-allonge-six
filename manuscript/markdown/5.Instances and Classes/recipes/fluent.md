## Fluent {#fluent}

Object and instance methods can be bifurcated into two classes: Those that query something, and those that update something. Most design philosophies arrange things such that update methods return the value being updated. For example:

    class Cake {
      setFlavour (flavour) { 
        return this.flavour = flavour 
      },
      setLayers (layers) { 
        return this.layers = layers 
      },
      bake () {
        // do some baking
      }
    }
    
    const cake = new Cake();
    cake.setFlavour('chocolate');
    cake.setLayers(3);
    cake.bake();

Having methods like `setFlavour` return the value being set mimics the behaviour of assignment, where `cake.flavour = 'chocolate'` is an expression that in addition to setting a property also evaluates to the value `'chocolate'`.

The [fluent] style presumes that most of the time when you perform an update, you are more interested in doing other things with the receiver than the values being passed as argument(s). Therefore, the rule is to return the receiver unless the method is a query:

    class Cake {
      setFlavour (flavour) { 
        this.flavour = flavour;
        return this;
      },
      setLayers (layers) { 
        this.layers = layers;
        return this;
      },
      bake () {
        // do some baking
        return this;
      }
    }

The code to work with cakes is now easier to read and less repetitive:

    const cake = new Cake().
                   setFlavour('chocolate').
                   setLayers(3).
                   bake();

For one-liners like setting a property, this is fine. But some functions are longer, and we want to signal the intent of the method at the top, not buried at the bottom. Normally this is done in the method's name, but fluent interfaces are rarely written to include methods like `setLayersAndReturnThis`.

[fluent]: https://en.wikipedia.org/wiki/Fluent_interface

When we write our own prototypes, the `fluent` method decorator solves this problem:

    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }

Now you can write methods like this:

    function Cake () {}

    Cake.prototype.setFlavour = fluent( function (flavour) { 
      this.flavour = flavour;
    });

It's obvious at a glance that this method is "fluent."

When we use the `class` keyword, we can decorate functions in a similar manner:

    class Cake {
      setFlavour (flavour) { 
        this.flavour = flavour;
      },
      setLayers (layers) { 
        this.layers = layers;
      },
      bake () {
        // do some baking
      }
    }
    Cake.prototype.setFlavour = fluent(Cake.prototype.setFlavour);
    Cake.prototype.setLayers = fluent(Cake.prototype.setLayers);
    Cake.prototype.bake = fluent(Cake.prototype.bake);
    
Or, we could write ourselves a slight variation:

    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }
    
    const fluentClass = (clazz, ...methodNames) {
      for (let methodName of methodNames) {
        clazz.prototype[methodName] = fluent(clazz.prototype[methodName]);
      }
      return clazz;
    }
    
Now we can simply write:

    fluentClass(Cake, 'setFlavour', 'setLayers', 'bake');
