[Metaprops]: Metaprops.md

# Static

## About
Static is a `#!luau luau` file inside Tome that can be modified before run time.

---

## How does it work?
Static currently is only for [Metaprops] support. Before Tome creates a new Tome, it will check this Static file and use the values for its metaprops.

For example if you set `FastAdd = true` inside the Static file. Then all Tomes will by default use that metaprop.

---

## Why use it?
If you find yourself using certain metaprops a lot, you can set a specific metaprop to a default value to avoid situations like this:

```luau
local tome: Tome.Tome = Tome.new({
	Tagging = true,
	UseParentMetaprops = true,
})

local tome2: Tome.Tome = Tome.new({
	Tagging = true,
	UseParentMetaprops = true,
})

local tome3: Tome.Tome = Tome.new({
	Tagging = true,
	UseParentMetaprops = true,
})
```