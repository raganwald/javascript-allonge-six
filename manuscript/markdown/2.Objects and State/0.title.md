# Stir the Allongé: Objects and State {#mutable}

![Life measured out by coffee spoons](images/coffee-spoons.jpg)

So far, we have discussed what many call "pure functional" programming, where every expression is necessarily [idempotent], because we have no way of changing state within a program using the tools we have examined.

We've also explored functions that rebind names within themselves as part of performing their calculations. And we briefly touched upon the notion of mutating an object as part of building it. But we have avoided objects that are meant to be changed, objects that model *state*.

[idempotent]: https://en.wikipedia.org/wiki/Idempotence

It's time to change *everything*.