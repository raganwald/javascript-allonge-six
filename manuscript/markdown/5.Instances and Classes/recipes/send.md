## Send {#send}

Previously, we saw that the recipe [bound](#bound) can be used to get a bound method from an instance. Unfortunately, invoking such methods is a little messy:

    mapWith(bound('eggs'))(inventories).map(
      boundmethod => boundmethod() 
    )
      //=> [ 36, 12, 42 ]

As we noted, it's ugly to write

    boundmethod => boundmethod()

So instead, we write a new recipe:

    const send = (methodName, ...args) =>
      (instance) => instance[methodName].apply(instance, args);

    mapWith(send('apples'))(inventories)
      //=> [ 0, 240, 24 ]
      
`send('apples')` works very much like `&:apples` in the Ruby programming language. You may ask, why retain `bound`? Well, sometimes we want the function but don't want to evaluate it immediately, such as when creating callbacks. `bound` does that well.