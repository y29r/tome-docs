[CollectionService]: https://create.roblox.com/docs/reference/engine/classes/CollectionService
[CollectionService:GetInstanceRemovedSignal]: https://create.roblox.com/docs/reference/engine/classes/CollectionService#GetInstanceRemovedSignal

# Tagging

## About
Tagging is a term used in Tome to refer to automatically disposing of Instances via a [CollectionService] tag.

---

## How does it work?
Tags are useful in the sense that once an Instance gets a tag, and later down that same Instance gets destroyed by most methods e.g. `#!luau Instance:Destroy`, `#!luau Instance:Remove` and `#!luau Instance.Parent = nil`.

Then the [CollectionService:GetInstanceRemovedSignal] will pick up on the object being destroyed, which from there the Tome can dispose of it from its own objects dictionary.

---

## Why use it?
This is especially useful for objects that may get prematurely destroyed by the void, or external scripts that don't use a garbage collection module.

Most garbage collection modules will **eventually** remove the object from memory if they're added, so it's generally case specific, and also how long you're okay with keeping that Instance in-memory for.

Long round-based games such as races may benefit from this with vehicles that can get destroyed incorrectly. Resulting in vehicles remaining in memory until the round collector frees them.