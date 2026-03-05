
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

Works just like `#!luau Tome:Attach` however any amount of objects can be provided, and once all the objects are successfully connected, a single clean up function is returned, which when called, will free all the attachments at once.

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

Calls the [`#!luau RunService:BindToRenderStep`](https://create.roblox.com/docs/reference/engine/classes/RunService#BindToRenderStep) method. Tome adds a function inside of itself to unbind the render binding once the Tome is destroyed.

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

Works exactly the same as `#!luau Tome:Connect`, however only connecting to a Signal once, and adding it into the Tome. Can be a custom-made Signal or an RBXScriptSignal.

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

Calls `#!luau task.delay`, with the 1st argument being `'duration'`. Once that time has elapsed successfully, the thread will call `#!luau Tome:Destroy` internally.

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

Internally calls `#!luau Tome:DestroyAllObjects` and `#!luau Tome:DestroyAllPages`. During both states of destruction, the Tome will enter a destroying state, where most attempts at mutation e.g. `#!luau Tome:Add` will throw an error. This state is usually very short lived (<0.00001s)

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
	```luau linenums="1" hl_lines="5-5"
	local newTome: Tome.Tome = Tome.new()
	
	newTome:Add(workspace.Part)
	
	local newPage: Tome.Tome = newTome:AddPage("Test")
	newPage:Add(workspace.Part2)
	
	newTome:DestroyAllObjects() --> destroys only part 1, and doesn't destroy the Page
	```
	
---