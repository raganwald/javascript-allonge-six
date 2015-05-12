## pluckWith {#pluck}

This pattern of combining [mapWith](#mapping) and [getWith](#getWith) is very frequent in JavaScript code. So much so, that we can take it up another level:

    const pluckWith = (attr) => mapWith(getWith(attr));
    
Or even better:

    const pluckWith = compose(mapWith, getWith);
    
And now we can write:

    const inventories = [
      { apples: 0, oranges: 144, eggs: 36 },
      { apples: 240, oranges: 54, eggs: 12 },
      { apples: 24, oranges: 12, eggs: 42 }
    ];
    
    pluckWith('eggs')(inventories)
      //=> [ 36, 12, 42 ]
      
Libraries like [Underscore] provide `pluck`, the flipped version of `pluckWith`:

    _.pluck(inventories, 'eggs')
      //=> [ 36, 12, 42 ]

Our recipe is terser when you want to name a function:

    const eggsByStore = pluckWith('eggs');
    
vs.

    const eggsByStore = (inventories) =>
      _.pluck(inventories, 'eggs');
    
And of course, if we have `pluck` we can use [flip](#flip) to derive `pluckWith`:

    const pluckWith = flip(_.pluck);

[Underscore]: http://underscorejs.org
