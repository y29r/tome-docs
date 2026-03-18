[Debris]: https://create.roblox.com/docs/reference/engine/classes/Debris

# Schedular

## About
The Schedular is what Tome uses to dispose of objects on an in-order basis.

---

## How does it work?
Objects can be passed in with [Tome.schedule] or unscheduled with [Tome.unschedule]. When scheduling an object, it's usually given a life time. Tome will wait until that life time has elapsed and destroy the object.

The Schedular uses a priority queue to keep track of which objects to destroy first. This means upserts, downserts, pushes and pops are `O(log n)` time complexity.

---

## Why use it?
The Schedular was made as a more performant replacement for [Debris]. A Roblox staff member has [already mentioned](https://devforum.roblox.com/t/debris-maxitems-still-in-effect-despite-being-deprecated/2612863/3) that the service is *deprecated* and a `#!luau task.delay` solution is more performant.

As mentioned above, the Schedular doesn't use such an approach, and is far more flexible and performant. Moreover, `#!luau task.delay` poses an issue with retaining the execution after the thread has been closed, or script has been destroyed. The Schedular will continue to run even after the thread dies, or the script gets destroyed.