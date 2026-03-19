[Tagging]: Tagging.md
[Tome:AddWithMiddleware]: API.md/#tomeaddwithmiddleware
[Tome:Instance]: API.md/#tomeinstance

# Metaprops

## About
Metaprops are short for meta properties. These properties affect how the Tome acts when doing certain operations. Such as adding, removing and destroying objects.

---

## Metaprops
There are a few metaprops that you can use to change the Tome:

### FastAdd
Normally Tome will do some sanity checks before adding an object into the Tome. Such as checking whether the object is a Tome within a Tome to prevent cyclic destroys, and updating tags when using [Tagging].

This metaprop will ignore those checks and skip past them. The reason you may want to do this is when you're absolutely sure you won't be running into these one-off cases, which will help save on performance.

### SpawnDestroy
Normally Tome will destroy objects inside the same loop thread. This behaviour can cause unexpected yielding and break logic. This metaprop aims to remove yielding entirely by creating a new thread for each individual object.

Most of the time you may only add one, or few yielding objects into the Tome. In this case you can save on some performance by using [Tome:AddWithMiddleware] to run the code in a new thread for only that object.

### Warnings
Sometimes issues can occur when adding or destroying objects. For example [Tome:Instance] take in a property dictionary as a second argument. Since Tome applies properties inside a safe call, you may not notice that a property gets applied due to a spelling mistake. Enabling this metaprop will allow you to see if something wasn't applied correctly.

This should be turned on in production enviornments as it can give good feedback to error analytics. In debugging it's common to not enable it.

### Tagging
Enables Tome [Tagging]

### UseParentMetaprops
With this metaprop, Pages created from the Tome will inherit the same metaprops.

---

## Why use metaprops?
Metaprops can signficantly help with performance in a lot of cases. A lot of the time you won't have to provide any. But if you find yourself using Tome for slightly more than just a basic garbage collector, then you may benefit from metaprops.