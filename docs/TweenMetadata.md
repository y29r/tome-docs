[Metaprops]: Metaprops.md
[Tome:Tween]: API.md/#tometween

# Tween Metadata

## About
Tween metadata is the 4th argument passed in [Tome:Tween].

---

## How does it work?
Tween metadata gives more functionality before and after adding the tween into the Tome. Below are the current keys you can add to edit how the tween will work:

### Play
When setting `#!luau Play = true` the tween will automaticall play without having to call `#!luau Tween:Play` after it is returned.

### RemoveOnComplete
When setting `#!luau RemoveOnComplete = true` the tween will be removed from the Tome after it has been completed. Completion is determined by the `#!luau Tween.Completed` signal.

---

## Why use it?
It can be helpful when writing code without needless variables. Take the example below:

```luau
local tome: Tome.Tome = Tome.new()

local tween: Tween = tome:Tween(workspace.Part, TweenInfo.new(1), {
	Size = Vector3.new(4, 4, 4),
})
tween:Play()
tween.Completed:Once(function()
	print("Tween has completed")
end)
```

There isn't inheritely wrong with this code, but it can be written shorter and less boilerplate-like. Especially when you're dealing with many tweens.

```luau
local tome: Tome.Tome = Tome.new()

tome:Tween(workspace.Part, TweenInfo.new(1), {
	Size = Vector3.new(4, 4, 4),
}, { Play = true, RemoveOnComplete = true }).Completed:Once(function()
	print("Tween has completed")
end)
```