[Metaprops]: Metaprops.md
[Variables]: API.md/#variables
[Tome.TweenType]: API.md/#tometweentype
[Tome.FunctionType]: API.md/#tomefunctiontype
[Tome.ThreadType]: API.md/#tomethreadtype
[Tome.Guess]: API.md/#tomeguess

# Destroy Methods

Destroy methods refers to how an object is going to get destroyed. For example usually Instances may have a `#!luau "Destroy"` destroy method.

Every object type has it's own way of being *destroyed*.

## String Based
Providing a string for a destroy method is one of the most common way to destroy an object. Usually you will provide a string which will be used to index a method for the object. Take this example:

```luau
local part: BasePart = workspace.Part

part:Destroy() --> this is indexing the "Destroy" method
```

Which can also just be written as:

```luau
local part: BasePart = workspace.Part

part["Destroy"](part) --> this also destroys the part
```

Now Tome takes advantage of this and uses it to destroy Instances and custom classes.

## Symbol Based
Tome has a few [Variables] that can be used to asses what kind an object is. Depending on the symbol, the object will be destroyed differently. Below is the code Tome uses to destroy an object, given the object and destroy method:

```luau
return function(object: any, destroyMethod: Types.DestroyMethod) : ()
	if type(destroyMethod) == "function" then
		return destroyMethod(object)
	end
	
	if destroyMethod == FunctionType then
		object()
	elseif destroyMethod == TweenType then
		object:Cancel()
		object:Destroy()
	elseif destroyMethod == ThreadType then
		local threadStatus: "dead" | "normal" | "running" | "suspended" = coroutine.status(object)
		
		if threadStatus ~= THREAD_DEAD_STATE then
			pcall(task.cancel, object)
		end
	else
		local destroyFunction: ((object: any) -> ())? = object[destroyMethod]
		if destroyFunction then
			pcall(destroyFunction, object)
		end
	end
end
```

## What Destroy Methods are allowed?
### Strings & indexing
It will depend on the kind of object you are passing in, but generally any. For example if you are adding in an Instance, you will be able to use a string as a destroy method to index some method. Technically speaking you can even use a method like `#!luau Instance:GetDescendants` but that wouldn't really do anything. Usually it will be up to preference between removal methods such as `#!luau Instance:Destroy`, `#!luau Instance:Remove` and parenting to nil.

### Functions
Functions are also a common way of using a custom destroy method if you need something a bit more flexible. Like calling another method of a class before finally destroying it i.e. an explosion effect, or fade out.

One thing that's quite useful is using built in functions to dispose of certain objects. For example cancelling a thread with `#!luau task.cancel`, you may use this as a destroy method:
```luau
local tome: Tome.Tome = Tome.new()

local myThread: thread = task.spawn(function()
	
end)

tome:Add(myThread, task.cancel)
```

Tome will always pass in the object as a the first argument when using a function for a destroy method.

### Automatic determining
When an object that doesn't have a destroy method passed in is added into the Tome, Tome will try and *guess* what to use based on the type of object. For example if you give in an Instance, the destroy method will be `#!luau "Destroy"`.

Below are a list of object types to destroy methods.

- Instance: `#!luau "Destroy"`
- Tween: [Tome.TweenType]
- function: [Tome.FunctionType]
- thread: [Tome.ThreadType]
- RBXScriptConnection: `#!luau "Disconnect"`
- table:
	- table.Destroy: `#!luau "Destroy"`
	- table.Disconnect: `#!luau "Disconnect"`
	- table.destroy: `#!luau "destroy"`
	- table.__call: `#!luau table.__call`
	
These are the objects Tome is able to automatically assign destroy methods to.