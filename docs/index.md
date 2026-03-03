# Introduction

## About

Tome is a garbage collection library that offers a variety of wrappers for properly disposing of objects at run time.

## FAQ

### Why even use a garbage collector like Tome?
In some cases traditional clean up can turn into more boiler plate expected. Take this example:
```luau linenums="1"
--!strict

local Vehicle = {}
Vehicle.__index = Vehicle

function Vehicle.new(model)
	local self = setmetatable({
		__model = model,
		__instances = {},
		__engineThread = nil,
	}, Vehicle)
end

function Vehicle:startEngine()
	local engineParticles = Instance.new("ParticleEmitter")
	-- ... particle properties
	engineParticles.Parent = self.__model.Root
	
	table.insert(self.__instances, engineParticles)
	
	self.__engineThread = task.spawn(function()
		while true do
			engineParticles.Rate = self.__model.Root.AssemblyLinearVelocity.Magnitude / 10
			
			task.wait(0.1)
		end
	end)
end

function Vehicle:Destroy()
	for index, instance in self.__instances do
		instance:Destroy()
	end
	
	if self.__engineThread then
		pcall(task.cancel, self.__engineThread)
	end
end

return Vehicle
```

This code is fine because it works without issue. But it lacks scalability. Later down the line you may need threads to run inside that vehicle class, which are then cleaned up when the vehicle should be destroyed.

This is where Tome comes in as a wrapper, simplifying everything:
```luau linenums="1"
--!strict

local Tome = require("@game/ReplicatedStorage/Tome/Tome")

local Vehicle = {}
Vehicle.__index = Vehicle

function Vehicle.new(model)
	local self = setmetatable({
		__model = model,
		__tome = Tome.new(), -- create a Tome object for every class object
	}, Vehicle)
end

function Vehicle:startEngine()
	local engineParticles = self.__tome:Add(Instance.new("ParticleEmitter"))
	engineParticles.Parent = self.__model.Root
	
	self.__tome:Spawn(function()
		while true do
			engineParticles.Rate = self.__model.Root.AssemblyLinearVelocity.Magnitude / 10
			
			task.wait(0.1)
		end
	end)
end

function Vehicle:Destroy()
	self.__tome:Destroy()
end

return Vehicle
```

### What advantages does it give over other libraries?
Tome offers mostly every feature that other garbage collection libraries have, with a lot more unique included. Alongside a similar API, Tome outperformed every library paired with its fastest add and destroy methods.