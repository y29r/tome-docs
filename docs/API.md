[Tagging]: Tagging.md
[Tome:AddFromDictionary]: API.md/#tomeaddfromdictionary
[Tome:Attach]: API.md/#tomeattach
[Tome:Add]: API.md/#tomeadd
[Tome:Destroy]: API.md/#tomedestroy
[Tome:Connect]: API.md/#tomeconnect
[Tome:DestroyAllObjects]: API.md/#tomedestroyallobjects
[Tome:DestroyAllPages]: API.md/#tomedestroyallpages
[Tome:Contains]: API.md/#tomecontains
[Tome:Remove]: API.md/#tomeremove
[Instance.fromExisting]: https://create.roblox.com/docs/reference/engine/datatypes/Instance#fromExisting
[Tome:SetTag]: API.md/#tomesettag
[RunService:BindToRenderStep]: https://create.roblox.com/docs/reference/engine/classes/RunService#BindToRenderStep

# API
The official API for Tome giving basic examples for constructors, embedded systems and methods.

---

## Constructors
Tome uses a single constructor to create a Tome object

### `#!luau Tome.new`

!!! info "Arguments"
	1. `#!luau metaprops: Metaprops?` &mdash; The metaprops object to use when instantiating the Tome.

!!! tip "Returns"
	1. `#!luau tome: Tome` &mdash; Returns a new Tome object.

Creating a Tome is very easy.

=== "Basic Example"
	```luau linenums="1" hl_lines="1-1"
	local newTome = Tome.new()
	```

=== "Typed Example"
	```luau linenums="1" hl_lines="1-1"
	local newTome: Tome.Tome = Tome.new()
	```

---

## Functions
Functions in Tome can be used directly from the module without instantiation.

### `#!luau Tome.Is`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to check.

!!! tip "Returns"
	1. `#!luau isATome: boolean` &mdash; Whether the object is a Tome.

Returns whether the provided object is a Tome object. This function will check against the direct metatable, and `#!luau Tome:BindRenderStepped` method (to track earlier versions)

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome = Tome.new()
	
	print("#1 Is this object a Tome? ", Tome.Is({})) -- false
	print("#2 Is this object a Tome? ", Tome.Is(newTome)) -- true
	print("#3 Is this object a Tome? ", Tome.Is()) -- false
	```

---

## Variables
Get access to Tome internals to speed up certain methods manually.

### `#!luau Tome.FunctionType`

Providing this as a DestroyMethod for methods like `#!luau Tome:Add` can significantly speed up adding functions into the Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome = Tome.new()
	
	newTome:Add(function()
		
	end, Tome.FunctionType) -- Tome will skip finding a DestroyMethod and use this instead.
	```

---

### `#!luau Tome.ThreadType`

Providing this as a DestroyMethod for methods like `#!luau Tome:Add` can significantly speed up adding threads into the Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome = Tome.new()
	
	local myThread: thread = task.spawn(function()
		
	end)
	
	newTome:Add(myThread, Tome.ThreadType) -- Tome will skip finding a DestroyMethod and use this instead.
	```

---

### `#!luau Tome.TweenType`

Providing this as a DestroyMethod for methods like `#!luau Tome:Add` can significantly speed up adding tweens into the Tome.

This DestroyMethod will first call `#!luau Tween:Cancel` and then `#!luau Tween:Destroy`.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome = Tome.new()
	
	local tween: Tween = TweenService:Create(...)
	
	newTome:Add(tween, Tome.TweenType) -- Tome will skip finding a DestroyMethod and use this instead.
	```

---

### `#!luau Tome.Guess`

This is usually only used in pair with [Tome:AddFromDictionary]. This is used as a value in the dictionary to let Tome know that it should calculate the DestroyMethod.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-4"
	local newTome = Tome.new()
	
	local objects: {[any]: Tome.DestroyMethod} = {
		[workspace.Part] = Tome.Guess,
	}
	
	newTome:AddFromDictionary(objects)
	```

---

## Methods
Tome offers a variety of methods to use on a Tome object

---

### `#!luau Tome:Add`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to add into the Tome. Usually an `Instance`, `RBXScriptConnection` or a class.
	2. `#!luau destroyMethod: DestroyMethod?` &mdash; Destroy method to use instead of Tome finding the destroy method.

!!! tip "Returns"
	1. `#!luau object: any` &mdash; Returns the same object provided in argument #1.

The primary method of adding an object into the Tome. 
```luau linenums="1" hl_lines="3-3"
local newTome: Tome.Tome = Tome.new()

newTome:Add(workspace.Part) -- adds the part into the Tome

newTome:Destroy()
```

#### Optimization & Finer control
Now if you need finer control over how to destroy the object, you can use the second argument.
When using the second argument, Tome will skip over finding the destroy method and use what's provided instead.

=== "Custom destroy method"
	```luau linenums="1" hl_lines="3-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part, function(part: BasePart)
		print("Going to destroy this part!", part:GetFullName())
		
		part:Destroy()
	end)
	
	newTome:Destroy()
	```

=== "Optimization approach"
	Internally Tome will conclude `#!luau "Destroy"` as the destroy method for Instances. Tome is always unaware of the type of object you provide, so it has to come up with the destroy method at run time. Giving Tome a destroy method improves write speed signficantly in a lot of cases:
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part, "Destroy")
	
	newTome:Destroy()
	```

---

### `#!luau Tome:AddTuple`

!!! info "Arguments"
	1. `#!luau Tuple: ...any` &mdash; Any number of tuple objects to add into the Tome.

!!! tip "Returns"
	1. `#!luau Tuple: ...any` &mdash; Returns the same tuple (and retaining order) provided.

For adding any tuple amount of objects into the Tome. The objects will be returned in the same order they were provided in.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:AddTuple(workspace.Part, workspace.Part2, workspace.Part3) -- adds all 3 parts into the Tome
	
	newTome:Destroy()
	```

=== "Return Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	-- variabalize both parts
	local part: BasePart, part2: BasePart = newTome:AddTuple(workspace.Part, workspace.Part2)
	
	newTome:Destroy()
	```


---

### `#!luau Tome:AddFromArray`

!!! info "Arguments"
	1. `#!luau array: {any}` &mdash; The array of objects to add into the Tome.

!!! tip "Returns"
	1. `#!luau array: {any}` &mdash; Returns the same array that was provided in argument #1.

For adding an array of objects into the Tome. Useful in cases where it's preferable to call this instead of iterating through the array manually.
```luau linenums="1" hl_lines="5-5"
local newTome: Tome.Tome = Tome.new()

local myParts: {Part} = {workspace.Part, workspace.Part2, workspace.Part3}

newTome:AddFromArray(myParts) -- adds all 3 parts from 'myParts'

newTome:Destroy()
```

---

### `#!luau Tome:AddFromDictionary`

!!! info "Arguments"
	1. `#!luau dictionary: {[any]: DestroyMethod | Tome.Guess}` &mdash; The dictionary of objects to add into the Tome.

!!! tip "Returns"
	1. `#!luau dictionary: {[any]: DestroyMethod | Tome.Guess}` &mdash; Returns the same dictionary that was provided in argument #1.

In some cases you may want to optimize adding objects into the Tome. One way you can do this is by having a dictionary on a:
```luau linenums="1"
[key]: object
[value]: DestroyMethod
```
basis.

In some scenarios you may not know what destroy method to put as the value in the pair, this is where `#!luau Tome.Guess` comes in. Using this as the value will let Tome know to find the destroy method for the object.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-12"
	local newTome: Tome.Tome = Tome.new()
	
	local myParts: {[BasePart]: Tome.DestroyMethod} = {
		[workspace.Part] = "Destroy",
		[workspace.Part2] = function(part: BasePart)
			print("Going to destroy part2!")
			
			part:Destroy()
		end,
	}
	
	newTome:AddFromDictionary(myParts)
	
	newTome:Destroy()
	```

=== "Extended Example"
	Imagine having to add hundreds of newly created objects into the Tome. Adding them one by one with `#!luau Tome:Add` will work fine, however adding them as a dictionary will improve performance significantly:
	```luau linenums="1" hl_lines="5-11"
	local newTome: Tome.Tome = Tome.new()
	
	local myParts: {[BasePart]: Tome.DestroyMethod} = {}
	
	for index: number = 1, 1_000 do
		local myPart: BasePart = Instance.new("Part")
		myPart.Position = Vector3.new(0, 20, 0)
		myPart.Parent = workspace
		
		myParts[myPart] = "Destroy"
	end
	
	newTome:AddFromDictionary(myParts)
	
	newTome:Destroy()
	```

---

### `#!luau Tome:AddPromise`

!!! info "Arguments"
	1. `#!luau promise: Promise` &mdash; The Promise to add into the Tome.

!!! tip "Returns"
	1. `#!luau promise: Promise` &mdash; Returns the same Promise that was provided in argument #1.

Adds in a standard Promise object into the Tome. The Promise must have `#!luau :cancel`, `#!luau :finally` and `#!luau :getStatus` as methods. Moreover, `#!luau :getStatus` must return `#!luau "started"` as a status when provided, otherwise an error will be thrown.
```luau linenums="1" hl_lines="5-5"
local newTome: Tome.Tome = Tome.new()

local newPromise: Promise.Promise = Promise.new(function()
	return -- ... some yielding code
end)

newTome:AddPromise(newPromise)

newTome:Destroy() -- now when the Tome is destroyed, the Promise above will get cancelled
```

---

### `#!luau Tome:AddPage`

!!! info "Arguments"
	1. `#!luau name: string?` &mdash; Providing a name will open up the ability to use other methods such as `#!luau Tome:GetPage`. Internally helps with debugging as well.
	2. `#!luau metaprops: Metaprops?` &mdash; Providing metaprops will apply them to the new Page when constructing.

!!! tip "Returns"
	1. `#!luau page: Tome` &mdash; Returns a new Page (Tome)

Instantiates a new Page (alias for Tome when referring to a Tome within a Tome) Unlike a regular object, a Page gets added to a seperate table internally, which allows for faster querying, and gives you control over how Pages are handled.

When the Tome that created the Page is destroyed, the Page will also destroy; cleaning up all the objects inside of it, and any sub-Pages.

??? warning "Pages are not released on destruction"
	When a Page is added into the Tome, Tome doesn't see it as just another regular object, Tome will keep the Page alive until manually removed with `Tome:RipPage` or `Tome:RipPages`. Not freeing the Page will **not** result in a memory leak, at some point once the Tome is done being used, luau's garbage collector will automatically free it.
	
	If you absolutely need a Page to be removed on destruction, you can manually pass it into the Tome as though it were an object:
	```luau
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:Add(Tome.new()) -- will get cleaned up once Tome is destroyed
	```
	This is not recommended, as you lose control over time in more complex systems. One-off cases are usually safe.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("MyPage")
	
	newPage:Add(workspace.Part) -- adds the Part into the Page
	
	newTome:Destroy() -- now when the Tome is destroyed, the Page gets destroyed, which also includes the Part
	```
	
=== "Page Clean-up Example"
	```luau linenums="1"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("MyPage")
	
	newPage:Add(workspace.Part)
	
	newTome:Destroy()
	
	print(newTome:GetPage("MyPage")) --> 'newPage'
	
	newTome:RipPage("MyPage")
	
	print(newTome:GetPage("MyPage")) --> nil
	```

---

### `#!luau Tome:Attach`

!!! info "Arguments"
	1. `#!luau object: Tome | Instance | RBXScriptSignal | {Connect: () -> ()}` &mdash; The object that the Tome will attach to.

!!! tip "Returns"
	1. `#!luau attachment: Attachment` &mdash; Returns an attachment which can be cleaned up to unattach the attachment.

Attaches the Tome to one of the few object types. Once attached, when either triggers are fired, the Tome will destroy itself.

!!! important ""
	Attachments are not cleaned up when the Tome is destroyed. For example, if a Signal is provided, and the Signal gets fired n times, the Tome will also be destroyed n times until the Signal is either destroyed, or the attachment is unattached.

	It's also important to note that if an Instance that doesn't have a parent is attached, no attachment will be made and nothing is returned.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-4"
	local newTome: Tome.Tome = Tome.new()

	local part: BasePart = workspace.Part
	newTome:Attach(newTome)

	newTome:Add(function()
		print("Tome has been destroyed")
	end)

	part:Destroy() -- destroying the part will now destroy the Tome
	```

=== "Attachment Clean-up Example"
	In this example, an attachment is stored in the variable `attachmentCleanup`. This attachment can be "cleaned up" at any time, and once cleaned, will prevent the object from invoking `#!luau Tome:Destroy` again.
	```luau linenums="1" hl_lines="8-8"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function()
		print("Tome destroyed")
	end)
	
	local signal: Signal.Signal = Signal.new()
	local attachmentCleanup: Tome.Attachment? = newTome:Attach(signal) -- attach the Signal
	
	signal:Fire() --> "Tome destroyed (1x)"
	signal:Fire() --> "Tome destroyed (2x)"
	signal:Fire() --> "Tome destroyed (3x)"
	
	if type(attachmentCleanup) == "table" then
		attachmentCleanup:Disconnect() -- disconnect the attachment
	end
	
	signal:Fire() --> nothing happens
	```

---

### `#!luau Tome:AttachTuple`

!!! info "Arguments"
	1. `#!luau Tuple: ...(Tome | Instance | RBXScriptSignal | {Connect: () -> ()})` &mdash; The tuple of objects that the Tome will attach to.

!!! tip "Returns"
	1. `#!luau cleanUp: () -> ()` &mdash; The clean up function.
	2. `#!luau attachments: {Attachment}` &mdash; The array of attachments (shouldn't be mutated)

Works just like [Tome:Attach] however any amount of objects can be provided, and once all the objects are successfully connected, a single clean up function is returned, which when called, will free all the attachments at once.

!!! important ""
	It's important to note that this method internally calls `#!luau Tome:Attach`. The same warning(s) apply from `#!luau Tome:Attach`, to here.

=== "Basic Example"
	```luau linenums="1" hl_lines="9-9"
	local newTome: Tome.Tome = Tome.new()
	newTome:OnDestroy(function()
		print("Tome destroyed")
	end)

	local part: BasePart = workspace.Part
	local part2: BasePart = workspace.Part2

	local cleanUp: () -> () = newTome:AttachTuple(part, part2)

	part:Destroy() --> "Tome destroyed (1x)"
	part2:Destroy() --> "Tome destroyed (2x)"
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="10-10"
	local newTome: Tome.Tome = Tome.new()
	newTome:OnDestroy(function()
		print("Tome destroyed")
	end)
	
	local part: BasePart = workspace.Part
	local part2: BasePart = workspace.Part2
	local part3: BasePart = workspace.Part3
	
	local cleanUp: () -> () = newTome:AttachTuple(part, part2, part3)
	
	part:Destroy() --> "Tome destroyed (1x)"
	part2:Destroy() --> "Tome destroyed (2x)"
	
	cleanUp()
	
	part3:Destroy() --> nothing happens
	```
	
---

### `#!luau Tome:BindRenderStepped`

!!! info "Arguments"
	1. `#!luau name: string` &mdash; The name of the render step binding.
	2. `#!luau renderPriority: number` &mdash; The name of the render step binding.
	3. `#!luau listener: (deltaTime: number) -> ()` &mdash; The name of the render step binding.

!!! tip "Returns"
	1. `#!luau unbind: () -> ()` &mdash; The unbind function (manual clean up)

Calls the [RunService:BindToRenderStep] method. Tome adds a function inside of itself to unbind the render binding once the Tome is destroyed.

Optionally you can call the `#!luau unbind()` function returned to manually clean up the binding.

=== "Basic Example"
	In this example, the binding will output the delta time each frame until 2 seconds have elapsed.
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	local cleanUp: Tome.CallableRenderStepBinding = newTome:BindRenderStepped("test-binding", 201, function(deltaTime: number)
		print(deltaTime)
	end)
	
	task.wait(2)
	
	cleanUp()
	```
	
---

### `#!luau Tome:CanDestroy`

!!! tip "Returns"
	1. `#!luau canDestroy: boolean` &mdash; Whether the Tome can be destroyed at this moment in time.

Returns whether the Tome can be destroyed in this current moment of time. You only need to call this before using methods that throw an error when attempting to mutate the Tome during its destroy life cycle.

=== "Basic Example"
	In this example, the Tome is running with `#!luau SpawnDestroy = false`, hence the `#!luau task.defer`. Adding a yielding function will temporarily halt the destroy thread, keeping the Tome in a destroying state.
	```luau linenums="1" hl_lines="4-4"
	local newTome: Tome.Tome = Tome.new()
	
	task.defer(function()
		print(newTome:CanDestroy())
	end)
	
	newTome:Add(function()
		task.wait(2)
	end)
	
	newTome:Destroy()
	```

---

### `#!luau Tome:Clone`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to clone
	2. `#!luau extendFunctionName: string?` &mdash; The method to call on the object to clone it.

!!! tip "Returns"
	1. `#!luau clonedObject: any` &mdash; The cloned object.

Clones the provided object using the `#!luau object:Clone()` as the clone method (unless overridden by the 2nd argument)

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	local clonedPart: BasePart = newTome:Clone(workspace.Part)
	
	newTome:Destroy() -- Destroys the cloned part
	```
	
---

### `#!luau Tome:Connect`

!!! info "Arguments"
	1. `#!luau Signal: Signal` &mdash; The Signal to connect to.
	2. `#!luau listener: (...any) -> ()` &mdash; The listener function to connect with.

!!! tip "Returns"
	1. `#!luau Connection: {Disconnect: () -> ()} | RBXScriptConnection` &mdash; The Connection object.

Connects to a Signal, and adding it into the Tome. Can be a custom-made Signal or an RBXScriptSignal.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Connect(workspace.ChildAdded, function(child: Instance)
		print(child:GetFullName())
	end)
	```
	
---

### `#!luau Tome:Once`

!!! info "Arguments"
	1. `#!luau Signal: Signal` &mdash; The Signal to connect to.
	2. `#!luau listener: (...any) -> ()` &mdash; The listener function to connect with.

!!! tip "Returns"
	1. `#!luau Connection: {Disconnect: () -> ()} | RBXScriptConnection` &mdash; The Connection object.

Works exactly the same as [Tome:Connect], however only connecting to a Signal once, and adding it into the Tome. Can be a custom-made Signal or an RBXScriptSignal.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Once(workspace.ChildAdded, function(child: Instance)
		print(child:GetFullName())
	end)
	```
	
---

### `#!luau Tome:Construct`

!!! info "Arguments"
	1. `#!luau class: {new: () -> ()} | () -> ()` &mdash; The class to construct.
	2. `#!luau Tuple: ...any` &mdash; The arguments to pass into the constructor.

!!! tip "Returns"
	1. `#!luau Class: any` &mdash; The constructed class.

Constructs a class containing a `#!luau Class.new()` constructor function. Optionally you can pass in the constructor function if your class uses another name for construction.

Any amount of arguments can be passed in after the class/constructor, which will be passed into it. After construction, the class object is added into the Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="10-10"
	local newTome: Tome.Tome = Tome.new()
	
	-- simulate a custom class
	local Class = {
		new = function()
			return {}
		end,
	}
	
	local newClass: any = newTome:Construct(Class)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="13-13"
	local newTome: Tome.Tome = Tome.new()
	
	-- simulate a custom class
	local Class = {
		create = function(name: string, health: number)
			return {
				name = name,
				health = health,
			}
		end,
	}
	
	local newClass: any = newTome:Construct(Class.create, "Tea", 100)
	```
	
---

### `#!luau Tome:Delay`

!!! info "Arguments"
	1. `#!luau duration: number` &mdash; The amount of time to delay.
	2. `#!luau listener: (...any) -> ...any` &mdash; The listener function to call after the delay.
	3. `#!luau Tuple: ...any` &mdash; The params to pass into the listener.

!!! tip "Returns"
	1. `#!luau thread: thread` &mdash; The delayed thread.

Calls `#!luau task.delay`, adds the delayed thread into the Tome, and returns it.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Delay(2.0, function()
		print("Prints after 2 seconds")
	end)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Delay(2.0, function(argument: number)
		print("The argument passed: ", argument)
	end, 25)
	```
	
---

### `#!luau Tome:extend`

Duplicate of `#!luau Tome:AddPage`

---

### `#!luau Tome:Extend`

Duplicate of `#!luau Tome:AddPage`

---

### `#!luau Tome:FastAdd`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to add into the Tome.
	2. `#!luau destroyMethod: DestroyMethod?` &mdash; Destroy method to use instead of Tome finding the destroy method.
	
!!! tip "Returns"
	1. `#!luau object: object` &mdash; The same object passed in.

The same as [Tome:Add] but executes without the majority of features in Tome.
This method will skip over sanity checks like whether the Tome is currently being destroyed, tagging, and recursion mistakes with nested Tomes.

If you prioritize speed over features, then using this will benefit you.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:FastAdd(workspace.Part) -- adds the Part, just faster
	
	newTome:Destroy() --> destroys the part as normal
	```

---

### `#!luau Tome:Spawn`

!!! info "Arguments"
	1. `#!luau listener: (...any) -> ...any` &mdash; The listener function to call when spawning.
	2. `#!luau Tuple: ...any` &mdash; The params to pass into the listener.
	
!!! tip "Returns"
	1. `#!luau thread: thread` &mdash; The spawned thread.

Calls `#!luau task.spawn`, adds the spawned thread into the Tome, and returns it.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Spawn(function()
		print("Running in a spawned thread")
	end)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Spawn(function(argument: number)
		print("The argument passed: ", argument)
	end, 25)
	```
	
---

### `#!luau Tome:Defer`

!!! info "Arguments"
	1. `#!luau listener: (...any) -> ...any` &mdash; The listener function to call after the defer.
	2. `#!luau Tuple: ...any` &mdash; The params to pass into the listener.
	
!!! tip "Returns"
	1. `#!luau thread: thread` &mdash; The deferred thread.

Calls `#!luau task.defer`, adds the deferred thread into the Tome, and returns it.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Defer(function()
		print("Running in a spawned thread")
	end)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Defer(function(argument: number)
		print("The argument passed: ", argument)
	end, 25)
	```
	
---

### `#!luau Tome:DelayDestroy`

!!! info "Arguments"
	1. `#!luau duration: number` &mdash; The amount of time (in seconds) to delay the destroy.
	
!!! tip "Returns"
	1. `#!luau thread: thread` &mdash; The thread responsible for destroying the Tome.

Calls `#!luau task.delay`, with the 1st argument being `'duration'`. Once that time has elapsed successfully, the thread will call [Tome:Destroy] internally.

=== "Basic Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(function()
		print("Tome destroyed")
	end)
	
	newTome:DelayDestroy(2.0)
	```
	
---

### `#!luau Tome:Destroy`

!!! info "Arguments"
	1. `#!luau Tuple: ...any` &mdash; Any amount of params to pass into the `#!luau Tome:OnDestroy` callbacks.

Internally calls [Tome:DestroyAllObjects] and [Tome:DestroyAllPages]. During both states of destruction, the Tome will enter a destroying state, where most attempts at mutation e.g. [Tome:Add] will throw an error. This state is usually very short lived (<0.00001s)

=== "Basic Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(function()
		print("Tome destroyed")
	end)
	
	newTome:Destroy()
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function(argument: string)
		print(argument)
	end)
	
	newTome:Destroy("Hello, world")
	```
	
---

### `#!luau Tome:DestroyAllObjects`

!!! info "Arguments"
	1. `#!luau __ignoreDestroyingProperty: boolean?` &mdash; Internal argument to skip over checking whether the Tome is currently destroying.

Destroys all the objects inside the Tome. Doesn't destroy Pages.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	newTome:DestroyAllObjects() --> destroys the part
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="8-8"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	newPage:Add(workspace.Part2)
	
	newTome:DestroyAllObjects() --> destroys only part 1, and doesn't destroy the Page
	```
	
---

### `#!luau Tome:DestroyAllPages`

!!! info "Arguments"
	1. `#!luau __ignoreDestroyingProperty: boolean?` &mdash; Internal argument to skip over checking whether the Tome is currently destroying.

Destroys all the Pages inside the Tome. Doesn't destroy objects.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	newPage:Add(workspace.Part)
	
	newTome:DestroyAllPages() --> destroys the Page "test" along with the part
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="8-8"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	newPage:Add(workspace.Part2)
	
	newTome:DestroyAllPages() --> destroys only the Page "test" alongside part 2
	```
	
---

### `#!luau Tome:DestroyObject`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to destroy.

Destroys the given object, only if it exists inside the Tome. Destroying it will remove it from the Tome as well.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	newTome:DestroyObject(part) --> destroys the part
	```
	
---

### `#!luau Tome:DestroyTuple`

!!! info "Arguments"
	1. `#!luau Tuple: ...any` &mdash; Any amount of objects to destroy.

Destroys the given object(s), only if they exist inside the Tome. Destroying them will remove them from the Tome as well.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart, part2: BasePart = newTome:AddTuple(workspace.Part, workspace.Part2)
	
	newTome:DestroyTuple(part, part2) --> destroys "part" and "part2"
	```
	
---

### `#!luau Tome:DestroyObjectsWithTag`

!!! info "Arguments"
	1. `#!luau tag: string` &mdash; The tag to use when querying Tome.

Destroys all objects with the given tag. This only works for Instances and the objects must be inside the Tome to be destroyed.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:AddTuple(workspace.Part)
	part:AddTag("Test")
	
	newTome:DestroyObjectsWithTag("Test") --> destroys "part"
	```
	
---

### `#!luau Tome:DestroyObjectsOfType`

!!! info "Arguments"
	1. `#!luau typeName: string` &mdash; The type of object(s) to destroy.

Destroys all objects that match the provided type. During querying Tome will search for the following in these types of objects
```luau
Instance: Instance:IsA(type)
table: table.__type == type
```

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	newTome:DestroyObjectsOfType("BasePart") --> destroys "part"
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	local signal: Signal.Signal = newTome:Signal()
	
	newTome:DestroyObjectsOfType("Signal") --> destroys only "signal"
	```

---

### `#!luau Tome:Contains`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to check.
	
!!! tip "Returns"
	1. `#!luau exists: boolean` &mdash; Whether the object exists.

Returns whether the provided object exists inside the Tome

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	print(newTome:Contains(part)) --> true
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	print(newTome:Contains(workspace.Part2)) --> false
	```

---

### `#!luau Tome:Has`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to check.
	
!!! tip "Returns"
	1. `#!luau exists: boolean` &mdash; Whether the object exists.

The exact same as [Tome:Contains].

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	print(newTome:Has(part)) --> true
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	print(newTome:Has(workspace.Part2)) --> false
	```

---

### `#!luau Tome:FromExisting`

!!! info "Arguments"
	1. `#!luau instance: Instance` &mdash; The object to Instance against.
	2. `#!luau destroyMethod: DestroyMethod?` &mdash; An optional override for the destroy method to use.
	
!!! tip "Returns"
	1. `#!luau object: Instance` &mdash; The instantiated instance created from the 1st argument.

Creates an Instance from an existing one. See [Instance.fromExisting] for more information.

If the Instance is created successfully, the Instance will be added into the Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:FromExisting(workspace.Part) -- creates a clone
	part.Parent = workspace
	
	newTome:Destroy()
	```

---

### `#!luau Tome:GetObjects`

!!! tip "Returns"
	1. `#!luau objects: {[any]: DestroyMethod}` &mdash; The objects.

Returns the internal dictionary Tome uses to store objects. This table shouldn't be mutated freely, but if you know what you're doing, feel free.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	print(newTome:GetObjects()) --> {[Instance(Part,FFFFF)] = "Destroy"}
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	local objects: {[any]: Tome.DestroyMethod} = newTome:GetObjects()
	
	objects[workspace.Part] = "Destroy"
	```

---

### `#!luau Tome:GetObjectsWithTag`

!!! info "Arguments"
	1. `#!luau tag: string` &mdash; The tag to use.

!!! tip "Returns"
	1. `#!luau objects: {any}` &mdash; The tagged objects.

Returns objects that have the provided tag. Only works for Instances

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	part:AddTag("Test")
	
	print(newTome:GetObjectsWithTag("Test")) --> {Instance(Part, FFFFF)}
	```

---

### `#!luau Tome:GetObjectsOfType`

!!! info "Arguments"
	1. `#!luau objectType: string` &mdash; The object type to query with.

!!! tip "Returns"
	1. `#!luau objects: {any}` &mdash; The objects that match the type provided.

Returns objects that have the provided type.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	part:AddTag("Test")
	
	print(newTome:GetObjectsOfType("BasePart")) --> {Instance(Part, FFFFF)}
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(function()
		
	end)
	
	print(newTome:GetObjectsOfType("function")) --> {Function(FFFFF)}
	```

---

### `#!luau Tome:GetPage`

!!! info "Arguments"
	1. `#!luau name: string` &mdash; The name of the Page to get.

!!! tip "Returns"
	1. `#!luau page: Tome?` &mdash; The Page that was fetched.

Returns a Page within the Tome from a given name. If the Page with the name doesn't exist, then nil is returned.

=== "Basic Example"
	```luau linenums="1" hl_lines="10-10"
	local newTome: Tome.Tome = Tome.new()
	
	-- pretend we lose access to our Page "Test" due to being inside a deep scope
	do
		newTome:AddPage("Test")
	end
	
	-- now we can fetch it
	
	local myPage: Tome.Tome? = newTome:GetPage("Test")
	
	print(myPage) --> Tome(Page("Test"))
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	local myPage: Tome.Tome? = newTome:GetPage("Test")
	
	print(myPage) --> nil, because we never created a Page
	```

---

### `#!luau Tome:GetParent`

!!! tip "Returns"
	1. `#!luau parent: Tome?` &mdash; The parent of the Tome.

Returns the parent Tome of the Tome that had this method invoked from. A Tome that has a parent is also referred to as a 'Page'.

If a parent doesn't exist, nil is returned.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local myPage: Tome.Tome = newTome:AddPage("Test")
	
	print(myPage:GetParent()) --> Tome (newTome)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	print(newTome:GetParent()) --> nil, because this Tome doesn't have a parent
	```

---

### `#!luau Tome:GetTag`

!!! tip "Returns"
	1. `#!luau tag: string?` &mdash; The tag Tome uses for [Tagging].

Returns the tag the Tome uses for tracking Instances when [Tagging] is enabled. If [Tome:SetTag] was not called to alter the tag, then a standard GUID is usually returned.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new({
		Tagging = true,
	})
	
	print(myPage:GetTag()) --> "9d2a0ed3-21db-43aa-8d47-3e06614ef6ae"
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new() -- create without Tagging enabled
	
	print(myPage:GetTag()) --> nil, because tagging is disabled
	```

---

### `#!luau Tome:GivePage`

!!! info "Arguments"
	1. `#!luau pageName: string` &mdash; The name of the Page to give.
	2. `#!luau newParent: Tome` &mdash; The new parent of the Page.

Gives a Page from the Tome that was invoked, to another Tome. If the Page doesn't exist, nothing will happen.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	local newTome2: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	
	newTome:GivePage("Test", newTome2)
	```
	
---

### `#!luau Tome:HookRunServiceSignal`

!!! info "Arguments"
	1. `#!luau signalName: RunServiceSignalName` &mdash; The name of the Signal to hook to.
	2. `#!luau listener: (deltaTime: number) -> () | (time: number, deltaTime: number) -> ()` &mdash; The listener function to call every step.

!!! tip "Returns"
	1. `#!luau connection: RBXScriptConnection` &mdash; The connection between the Signal and listener.

Hooks to a RunService Signal, which can be any one of the following:
```luau
"RenderStepped"
"Heartbeat"
"PostSimulation"
"PreAnimation"
"PreRender"
"PreSimulation"
"Stepped"
```

When the RBXScriptConnection is created, it will be added to the Tome; disconnecting it once the Tome is destroyed.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-6"
	local newTome: Tome.Tome = Tome.new()
	
	-- hooks to the RunService.RenderStepped Signal
	newTome:HookRunServiceSignal("RenderStepped", function(deltaTime: number)
		print(deltaTime)
	end)
	```
	
=== "Extended Example"
	```luau linenums="1" hl_lines="3-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:HookRunServiceSignal("RenderStepped", function(deltaTime: number)
		print(deltaTime)
	end)
	
	task.wait(1)
	
	newTome:Destroy() -- will :Disconnect the RBXScriptConnection made above
	```

=== "Extended Example 2"
	```luau linenums="1" hl_lines="4-6"
	local newTome: Tome.Tome = Tome.new()
	
	-- RunService.Stepped is different from the others, in the sense that it responds with 2 number arguments
	newTome:HookRunServiceSignal("Stepped", function(timeElapsed: number, deltaTime: number)
		print(timeElapsed, deltaTime)
	end)
	
	task.wait(1)
	
	newTome:Destroy() -- will :Disconnect the RBXScriptConnection made above
	```

---

### `#!luau Tome:Instance`

!!! info "Arguments"
	1. `#!luau instanceName: string` &mdash; The name of the Instance to create.
	2. `#!luau properties: {[string]: any}` &mdash; The dictionary of properties to apply to the Instance.
	3. `#!luau destroyMethod: DestroyMethod?` &mdash; Optional DestroyMethod to use instead of `#!luau "Destroy".

!!! tip "Returns"
	1. `#!luau instance: Instance` &mdash; The created Instance.

Creates a new Instance from a given name. The Instance will automatically be added into the Tome.

Optionally you can provide properties to apply to the Instance before returning it.

!!! note ""
	Properties are applied within a safe call. This means if you incorrectly apply a property, the error will be supressed. To avoid this, ensure you create the Tome with the metaprop `#!luau Warnings` set to true.

Optionally you can also provide a custom destroy method.

??? important "DEV-TODO"
	Currently there's no guarantee that the property `#!luau properties.Parent` will be set after all the other properties. This is important because setting properties after parenting is slower than setting them during creation.
	
	This will be changed in the near future.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Instance("Part", {
		Position = Vector3.new(0, 2, 0),
	})
	part.Parent = workspace -- setting the Parent out here
	
	newTome:DelayDestroy(2.0)
	```

---

### `#!luau Tome:IsDestroying`

!!! tip "Returns"
	1. `#!luau destroying: boolean` &mdash; Whether the Tome is currently destroying.

Returns whether the Tome is currently being destroyed.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-6"
	local newTome: Tome.Tome = Tome.new()
	
	-- fake adding something into the Tome from another block of code
	task.delay(1, function()
		print(newTome:IsDestroying()) --> true
	end)
	
	newTome:Add(function()
		task.wait(5) --> halts the Tome, keeping it in a destroying state
	end)
	
	newTome:Destroy()
	```

---

### `#!luau Tome:Move`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to move from the Tome.
	2. `#!luau tome: Tome` &mdash; The Tome to move the object into.

!!! tip "Returns"
	1. `#!luau object: object` &mdash; The object object passed in.

Moves the provided object into another Tome. Removing it from the Tome that this method was called from as well.

!!! note ""
	During the move, the object will keep the exact same destroy method inside the new Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	local newTome2: Tome.Tome = Tome.new()
	
	newTome:Move(workspace.Part, newTome2)
	
	print(newTome:Contains(workspace.Part)) --> false
	print(newTome2:Contains(workspace.Part)) --> true
	```

---

### `#!luau Tome:Parent`

!!! info "Arguments"
	1. `#!luau tome: Tome` &mdash; The new parent to live under.

!!! tip "Returns"
	1. `#!luau tome: Tome` &mdash; Returns the Tome this method was called from.

Parents the Tome to another Tome. If the Tome already has a parent, then it will unparent from it first.

!!! note ""
	If the Tome doesn't have a name, the Tome will **not** be parented. To name a Tome you can do the following:
	```luau linenums="1" hl_lines="2-2"
	local newTome: Tome.Tome = Tome.new()
	newTome:Rename("Drink water")
	```
	
	All Tomes that are created with `#!luau Tome:AddPage, Tome:extend, Tome:Extend` will come with a standard GUID name.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-4"
	local newTome: Tome.Tome = Tome.new()
	
	local newTome2: Tome.Tome = Tome.new()
	newTome2:Parent(newTome) --> parents 'newTome2' under 'newTome'
	```

---

### `#!luau Tome:Remove`

!!! info "Arguments"
	1. `#!luau object: any` &mdash; The object to remove from the Tome.

!!! tip "Returns"
	1. `#!luau object: object` &mdash; The same object that was passed in.

Safely removes the provided object from the Tome, if it exists inside it.
Removing an object does **not** destroy it.

!!! note ""
	If the Tome has [Tagging] enabled and the object is an Instance, the object will have the Tome's tag removed as well.

=== "Basic Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	print(newTome:Contains(workspace.Part)) --> true
	
	newTome:Remove(part)
	
	print(newTome:Contains(workspace.Part)) --> false
	```

---

### `#!luau Tome:RemoveTuple`

!!! info "Arguments"
	1. `#!luau Tuple: ...any` &mdash; The objects to remove from the Tome.

!!! tip "Returns"
	1. `#!luau Tuple: ...object` &mdash; The same objects that were passed in.

The same as [Tome:Remove] with the one change of being able to provide a tuple of objects, instead of just one.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	local part2: BasePart = newTome:Add(workspace.Part2)
	
	newTome:RemoveTuple(part, part2)
	```

---

### `#!luau Tome:RemoveFromArray`

!!! info "Arguments"
	1. `#!luau arrayOfObjects: {any}` &mdash; The objects to remove from the Tome.

!!! tip "Returns"
	1. `#!luau arrayOfObjects: {any}` &mdash; The same objects that were passed in.

The same as [Tome:Remove] with the one change of being able to provide an array of objects, instead of just one.

=== "Basic Example"
	```luau linenums="1" hl_lines="8-8"
	local newTome: Tome.Tome = Tome.new()
	
	local array: {BasePart} = {
		newTome:Add(workspace.Part),
		newTome:Add(workspace.Part2),
	}
	
	newTome:RemoveArray(array)
	```

---

### `#!luau Tome:RemoveObjectsWithTag`

!!! info "Arguments"
	1. `#!luau tag: string` &mdash; The tag to use for getting the objects to remove.

Removes all objects from the Tome that have the specified tag. This only works for Instances.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	part:AddTag("IShouldBeRemoved")
	
	newTome:RemoveObjectsWithTag("IShouldBeRemoved")
	```

---

### `#!luau Tome:RemoveObjectsOfType`

!!! info "Arguments"
	1. `#!luau type: string` &mdash; The type to use for getting the objects to remove.

Removes all objects from the Tome that are of the specified type.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	local part: BasePart = newTome:Add(workspace.Part)
	
	newTome:RemoveObjectsOfType("BasePart")
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="7-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(function()
		
	end)
	
	newTome:RemoveObjectsOfType("function")
	```
	
=== "Extended Example 2"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(Tome.new())
	
	newTome:RemoveObjectsOfType("Tome")
	```

---

### `#!luau Tome:Rename`

!!! info "Arguments"
	1. `#!luau name: string` &mdash; The new name for the Tome.

Renames the Tome internally. This is usually used for debugging. But can be used for other purposes too.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	print(newTome) --> {pages = {["Test"] = {...}}}
	
	newPage:Rename("Test")
	
	print(newTome) --> {pages = {["Test2"] = {...}}}
	```

---

### `#!luau Tome:RipPage`

!!! info "Arguments"
	1. `#!luau nameOrPage: string | Tome` &mdash; The name of a Page/Tome or the Page/Tome itself.
	
!!! tip "Returns"
	1. `#!luau tome: Tome` &mdash; The Tome itself (for chaining purposes)

Removes the Page from the Tome permanently. Destroying itself before being removed.

!!! warning ""
	If a Page doesn't exist, an error will be thrown. To avoid this, you can call `#!luau Tome:GetPage` before attempting to remove a Page.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-4"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage()
	newTome:RipPage(newPage) --> destroys, and removes the newPage from newTome
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	local newPage2: Tome.Tome = newTome:AddPage("Test2")
	
	newTome:RipPage(newPage):RipPage(newPage2)
	```

---

### `#!luau Tome:RipPages`

Destroys **all** Pages inside the Tome, and removes them.

=== "Basic Example"
	```luau linenums="1" hl_lines="6-6"
	local newTome: Tome.Tome = Tome.new()
	
	local newPage: Tome.Tome = newTome:AddPage()
	local newPage2: Tome.Tome = newTome:AddPage()
	
	newTome:RipPages()
	```

---

### `#!luau Tome:SetTag`

!!! info "Arguments"
	1. `#!luau tag: string` &mdash; The new tag to apply.
	
!!! tip "Returns"
	1. `#!luau tome: Tome` &mdash; The Tome itself (for chaining purposes)

Sets a new tag for the Tome. This will remove the previous tag, which means all other objects that were tagged, will have their tag replaced with the new one. This will have to add and remove the tags for instances, which can be slow when used frequently, in mass.

This is mainly used for debugging, or very case-specific situations with [Tagging].

=== "Basic Example"
	```luau linenums="1" hl_lines="4-9"
	local newTome: Tome.Tome = Tome.new({
		Tagging = true,
	})
	newTome:SetTag("Test")
	
	local part: BasePart = newTome:Add(workspace.Part)
	print(part:HasTag("Test")) --> true
	
	newTome:SetTag("Test2")
	
	print(part:HasTag("Test")) --> false
	print(part:HasTag("Test2")) --> true
	
	```

---

### `#!luau Tome:Signal`

!!! tip "Returns"
	1. `#!luau signal: Signal` &mdash; The created Signal

Creates a standard Signal implementation, and adds it to the Tome. This is from earlier versions of Tome.

=== "Basic Example"
	```luau linenums="1" hl_lines="3-3"
	local newTome: Tome.Tome = Tome.new()
	
	local signal: Tome.Signal = newTome:Signal()
	
	signal:Connect(function(argument: string)
		print(argument)
	end)
	
	signal:Fire("Hello, world!") --> "Hello, world!"
	```

---

### `#!luau Tome:UnbindRenderStepped`

!!! info "Arguments"
	1. `#!luau name: string` &mdash; The name of the binding.

Manually unbinds the render step RunService binding.

=== "Basic Example"
	```luau linenums="1" hl_lines="9-9"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:BindRenderStepped("Test", 201, function(deltaTime: number)
		print(deltaTime)
	end)
	
	task.wait(1)
	
	newTome:UnbindRenderStepped("Test")
	```

---

### `#!luau Tome:Tween`

!!! info "Arguments"
	1. `#!luau instance: Instance` &mdash; The instance to tween.
	2. `#!luau tweenInfo: TweenInfo` &mdash; The tween info to use to mutate the instance.
	3. `#!luau propertyTable: {[string]: any}` &mdash; The properties to tween to.
	4. `#!luau metadata: Metadata?` &mdash; The metadata to mutate how to tween works.

!!! tip "Returns"
	1. `#!luau tween: Tween` &mdash; The created tween.

Creates a Tween and adds it into the Tome. This method comes with some extra benefits, such as metadata:

Metadata changes how the tween works. See [Link... once I make it] for more information.

=== "Basic Example"
	```luau linenums="1" hl_lines="9-9"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Tween(
		workspace.Part,
		TweenInfo.new(2),
		{ Size = Vector3.new(2, 2, 2) }
	):Play()
	```
	
=== "Extended Example"
	In this example, once the tween completes, the Tome gets destroyed. This is useful for creating timed events without creating new threads.
	
	```luau linenums="1" hl_lines="3-8"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Tween(
		workspace.Part,
		TweenInfo.new(2),
		{ Size = Vector3.new(2, 2, 2) },
		{ Play = true }
	).Completed:Once(newTome:WrapDestroy())
	```
	
---

### `#!luau Tome:Table`

!!! info "Arguments"
	1. `#!luau table: {[any]: any}?` &mdash; The table to use instead of creating a new one.

!!! tip "Returns"
	1. `#!luau table: table | {[any]: any}` &mdash; The same table passed in, or the created table.

Adds (or creates) a table into the Tome. The table will have its destroy method as `#!luau table.clear`. Once the Tome gets destroyed, the table will clear itself.

This is useful in semi-round systems where you may track players within a sub-round, and need to clear them after the sub-round ends.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-6"
	local newTome: Tome.Tome = Tome.new()
	
	local parts: {BasePart} = newTome:Table({
		workspace.Part,
		workspace.Part2
	})
	```

---

### `#!luau Tome:OnDestroy`

!!! info "Arguments"
	1. `#!luau callback: (...any) -> ()` &mdash; The callback to add once the Tome destroys.
	2. `#!luau onDestroyParams: OnDestroyParams` &mdash; The params to mutate how the callback works.

!!! tip "Returns"
	1. `#!luau cleanUp: () -> ()` &mdash; A clean up function to remove the callback.

Adds a callback function into the Tome that listens for when the Tome gets destroyed. The callback will recieve the same arguments that were passed in [Tome:Destroy]. This can act as a sort of Signal.

Optionally you can provide OnDestroyParams that affect when and how the callback gets called.

Currently the supported parameters are:
`Synchronous`: Will call the callback using `task.spawn` instead of directly calling it. This is useful if you know the callback will yield.
`Deferred`: Will call the callback using `task.defer`. This takes priority over `Synchronous`, but `Synchronous` must be defined as true.
`RemoveOnDestroy`: Will call the callback, and then remove it from the Tome. In cases like this, it's better to use [Tome:Add] which does the same thing, while being more inline with Tome.

!!! note ""
	By default, `Synchronous` is set to true. This is to prevent hard yielding. If for whatever reason you need the Tome callback thread to yield, you can manually set it to false.

=== "Basic Example"
	```luau linenums="1" hl_lines="4-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function()
		print("Hello, world!")
	end)
	
	newTome:Destroy() --> "Hello, world!"
	```

=== "Extended Example"
	```luau linenums="1" hl_lines="4-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function(myArgument: string)
		print(myArgument)
	end)
	
	newTome:Destroy("Hello, world!") --> "Hello, world!"
	```

=== "Extended Example 2"
	In this example, the callback will be deferred via `!#luau task.defer`.
	
	```luau linenums="1" hl_lines="4-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function(myArgument: string)
		print(myArgument)
	end, {
		Deferred = true,
	})
	
	newTome:Destroy("Hello, world!") --> "Hello, world!" (a tiny bit later)
	```

=== "Extended Example 3"
	In this example, nothing happens because the callback was removed after it was called once. Hence the second time we call [Tome:Destroy], nothing happens.
	
	```luau linenums="1" hl_lines="4-7"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:OnDestroy(function(myArgument: string)
		print(myArgument)
	end, {
		RemoveOnDestroy = true,
	})
	
	newTome:Destroy("Hello, world!") --> "Hello, world!"
	newTome:Destroy("Hello, world!") --> 
	```
	
---

### `#!luau Tome:Yield`

!!! info "Arguments"
	1. `#!luau thread: thread?` &mdash; The thread to yield.

Adds the provided thread into the Tome (or the current thread of a thread isn't provided) and uses `#!luau coroutine.resume` as the destroy method.

The current thread will then yield. Keep in mind, if the main thread is not resumed, you may encounter issues.

This will result in the thread resuming once the Tome is destroyed.

=== "Basic Example"
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:DelayDestroy(2.0)
	
	newTome:Yield() -- stops the current thread
	
	print("Hello, world!") -- prints after ~2 seconds
	```
