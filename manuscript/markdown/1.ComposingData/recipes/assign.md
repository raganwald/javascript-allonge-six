## Object.assign

It's very common to want to "extend" an object by assigning properties to it:

    const inventory = {
      apples: 12,
      oranges: 12
    };
    
    inventory.bananas = 54;
    inventory.pears = 24;

It's also common to want to assign the properties of one object to another:

[shallow copy]: https://en.wikipedia.org/wiki/Object_copy#Shallow_copy

      for (let fruit in shipment) {
        inventory[fruit] = shipment[fruit]
      }

Both needs can be met with `Object.assign`, a standard function. You can copy an object by extending an empty object:

    Object.assign({}, {
      apples: 12,
      oranges: 12
    })
      //=> { apples: 12, oranges: 12 }

You can extend one object with another:

    const inventory = {
      apples: 12,
      oranges: 12
    };
    
    const shipment = {
      bananas: 54,
      pears: 24
    }
    
    Object.assign(inventory, shipment)
      //=> { apples: 12,
      //     oranges: 12,
      //     bananas: 54,
      //     pears: 24 }
      
And when we discuss prototypes, we will use `Object.assign` to turn this:

    const Queue = function () {
      this.array = [];
      this.head = 0;
      this.tail = -1
    };
      
    Queue.prototype.pushTail = function (value) {
      // ...
    };
    Queue.prototype.pullHead = function () {
      // ...
    };
    Queue.prototype.isEmpty = function () {
      // ...
    }

Into this:

    const Queue = function () {
      Object.assign(this, {
        array: [],
        head: 0,
        tail: -1
      })
    };
      
    Object.assign(Queue.prototype, {
      pushTail (value) {
        // ...
      },
      pullHead () {
        // ...
      },
      isEmpty () {
        // ...
      }      
    });
    
Assigning properties from one object to another (also called "cloning" or "shallow copying") is a basic building block that we will later use to implement more advanced paradigms like mixins.