[Metaprops]: Metaprops.md
[Variables]: API.md/#variables

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