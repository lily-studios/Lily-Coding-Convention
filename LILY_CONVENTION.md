````
# Lily Studio Coding Convention

> **Official coding standard for Lily Studio Roblox and Luau development.**

The Lily Studio coding convention exists so every Lily project follows the same structure, naming style, control-flow rules, type-safety expectations, performance habits, and architectural patterns. The goal is not to make every file look identical, but to make every file feel familiar enough that another Lily Studio developer can open it, understand it quickly, and continue working without having to learn a different style each time.

Lily code should always favor **clarity, consistency, predictable behavior, simple control flow, strong typing, clean ownership, and easy cleanup**. Code should be written for the next developer who needs to read it, debug it, extend it, or safely replace part of it later.

> ### Core principle
>
> **Clear first. Compact second. Clever never.**

> [!IMPORTANT]
> **Lily Studio code should be direct, controlled, predictable, typed, efficient, and fully owned throughout its lifecycle.** Important behavior should never depend on hidden setup, accidental timing, unclear ownership, or uncontrolled background work.

---

## Table of Contents

- **Foundation**
  - [1. Purpose](#1-purpose)
  - [2. General Principles](#2-general-principles)
  - [3. Preserve Existing Behavior](#3-preserve-existing-behavior)
  - [4. Controlled and Predictable Behavior](#4-controlled-and-predictable-behavior)
- **Architecture and Dependencies**
  - [5. ModuleScript-First Architecture](#5-modulescript-first-architecture)
  - [6. Lily Packages Only](#6-lily-packages-only)
  - [7. Composition Over Inheritance](#7-composition-over-inheritance)
  - [8. Generic Solutions](#8-generic-solutions)
- **State, Change, and Lifecycle**
  - [9. State Ownership](#9-state-ownership)
  - [10. Public State](#10-public-state)
  - [11. Attributes Over ValueObjects](#11-attributes-over-valueobjects)
  - [12. Event-Driven Code](#12-event-driven-code)
  - [13. Cleanup and Lifecycle](#13-cleanup-and-lifecycle)
- **Functions and Control Flow**
  - [14. Naming](#14-naming)
  - [15. Functions](#15-functions)
  - [16. Function Arguments](#16-function-arguments)
  - [17. Prefer Table-Driven Design](#17-prefer-table-driven-design)
  - [18. Guard Clauses](#18-guard-clauses)
  - [19. No Conditional Nesting](#19-no-conditional-nesting)
  - [20. Avoid `else` and `elseif`](#20-avoid-else-and-elseif)
  - [21. Positive Conditions](#21-positive-conditions)
  - [22. Avoid Boolean Expression Control Flow](#22-avoid-boolean-expression-control-flow)
  - [23. Iteration](#23-iteration)
- **Types, Documentation, and Failures**
  - [24. Type Checking](#24-type-checking)
  - [25. Comments and Documentation](#25-comments-and-documentation)
  - [26. Error Handling](#26-error-handling)
- **File Layout and Source Organization**
  - [27. File Organization](#27-file-organization)
  - [28. Alphabetical Top-Level Order](#28-alphabetical-top-level-order)
  - [29. Lily Separators](#29-lily-separators)
  - [30. Formatting](#30-formatting)
- **Runtime Infrastructure and User Interface**
  - [31. Script-Created Runtime Infrastructure](#31-script-created-runtime-infrastructure)
- **Performance and Scale**
  - [32. Performance](#32-performance)
  - [33. Luau Local and Register Limits](#33-luau-local-and-register-limits)
- **Reference Examples**
  - [34. Common Lily Anti-Patterns](#34-common-lily-anti-patterns)
  - [35. Example Lily Function](#35-example-lily-function)
  - [36. Example Lily Module](#36-example-lily-module)
  - [37. Final Standard](#37-final-standard)

---

## Foundation

These sections define the basic expectations that apply to every Lily Studio file. They establish what Lily code is trying to achieve before architecture or implementation details are considered.

### 1. Purpose

This document defines the shared engineering standard for Lily Studio projects and should guide new development, pull-request reviews, refactors, shared packages, interface work, runtime systems, and long-term maintenance. Its purpose is to give every Lily developer the same expectations for how code is structured, named, documented, controlled, and maintained.

The convention applies to **all Lily Studio Luau code**, not one specific game, feature, or system. Examples use neutral names such as `foo`, `bar`, `baz`, `object`, `data`, and `value` so that the rules stay general and can be applied anywhere in the Lily codebase.

The convention should help Lily developers answer the same questions in the same way:

- How should files be organized?
- How should functions be named?
- How should invalid states be handled?
- How should state changes be observed?
- Where should logic live?
- When should a script be used instead of a module?
- How should UI be created?
- How should external dependencies be handled?
- How should code be typed and cleaned up?

The goal is to reduce style differences between developers and make Lily projects easier to maintain as the codebase grows.

---

### 2. General Principles

Lily code should be straightforward to follow from top to bottom, with each section serving a defined purpose and each function having a defined responsibility. A developer reading unfamiliar Lily code should be able to understand the execution path without untangling nested branches, hidden state changes, indirect behavior, or compact expression tricks.

Lily code should normally be:

- **clear**, because the meaning should be obvious without extra explanation
- **compact**, because unnecessary lines and repeated logic make files more difficult to maintain safely
- **block-organized**, because related validation, resolution, state changes, and output should remain visually grouped instead of being scattered across unnecessary helpers
- **type-safe**, because mistakes should be caught before runtime whenever possible
- **event-driven**, because code should react when something changes instead of repeatedly checking for change
- **modular**, because features should be separated into focused reusable modules
- **predictable**, because similar systems should use similar patterns
- **cleanable**, because every connection, object, task, and runtime state should have a clear **lifecycle**
- **performant**, because repeated minor runtime costs can become large in a large Roblox project
- **consistent**, because the same idea should be written the same way throughout Lily Studio

Lily code should not try to impress the reader with unusual patterns. The best Lily code should feel clear, direct, and reliable enough to trust during maintenance.

---

### 3. Preserve Existing Behavior

When existing Lily code is cleaned up, reorganized, optimized, documented, or given stronger types, its runtime behavior must remain unchanged unless the work explicitly includes a behavior change. A refactor should improve structure, readability, maintainability, or performance while preserving the established contract of the system.

Do not silently change:

- remote names
- payload fields
- public method names
- return values
- default values
- timing behavior
- state behavior
- **cleanup** behavior
- input behavior
- networking behavior
- module APIs
- object ownership
- initialization order

**Example**

If existing code sends:

```lua
remote:FireServer({
	value = value,
	enabled = enabled,
})
```

a style **cleanup** should not silently change it to:

```lua
remote:FireServer({
	amount = value,
	active = enabled,
})
```

The second version may look cleaner, but it changes the API and can break code that depends on the original field names.

> **Rule:** Refactoring improves structure and readability, while behavior changes must remain explicit and intentional.

---

### 4. Controlled and Predictable Behavior

Lily systems should behave in a controlled, predictable, and intentional way so that important behavior never depends on guesswork. A developer should be able to trace where a value originates, which owner is allowed to change it, what actions follow that change, and how the system eventually returns to a known clean state.

The code should not depend on accidental timing, hidden state, unexplained fallbacks, or behavior that only behaves correctly because several unrelated parts happen to run in a certain order.

> **Main rule:** Lily should define what will happen, why it will happen, and which owner is responsible for each part of the execution path.

#### Explicit state changes

State should change through clear functions or owners instead of being modified from unrelated places.

**Preferred**

```lua
local function setFooValue(fooContext: FooContext, value: number)
	if fooContext.value == value then return end

	fooContext.value = value
	updateFoo(fooContext)
end
```

The function shows exactly when the value changes and what happens afterward.

**Avoid**

```lua
fooContext.value = value
```

when many unrelated files can change the same value without going through a shared owner.

#### Explicit defaults

Defaults should be defined clearly instead of depending on missing data or accidental engine behavior.

**Preferred**

```lua
local defaultFoo = 1
```

A developer should not need to guess what happens when a value is not provided.

#### Explicit execution order

When order matters, the code should make that order clear.

```lua
updateState(fooContext)
updateView(fooContext)
notifyListeners(fooContext)
```

If a later step depends on an earlier step, that relationship should be visible in the function and explained when the reason is not obvious.

**Avoid hidden side effects**

A function should not silently change unrelated state.

The function name, API, and structure should make important side effects easy to discover.

If `setFooValue()` also destroys objects, sends network data, starts tasks, and changes unrelated configuration, the function is doing too much and should be separated.

**Avoid accidental timing dependencies**

Do not design expected runtime behavior around assumptions such as:

- another script will probably run first
- a value will probably exist by the next frame
- a connection will probably be ready in time
- two independent callbacks will probably execute in the expected order

If an order or dependency matters, Lily should control it directly through initialization, modules, signals, or explicit state.

#### Random behavior must still be controlled

Randomness is allowed only when the feature intentionally requires it. The system should still control when randomness is used, what range is allowed, and how the result affects state.

Do not use randomness as a replacement for missing logic or uncertain behavior.

#### No unknown ownership

**Every important value should have an explicitly identified owner with responsibility for mutation, access, synchronization, and cleanup.**

A developer should be able to answer:

- who creates it
- who may change it
- who reads it
- who cleans it up
- what its default is
- what happens when it changes

If those answers are unclear, the ownership model should be improved.

#### Predictable failure behavior

Failures should also be controlled. A function should clearly return, warn, or fail according to the type of problem instead of leaving the system in a half-updated state.

> **Rule:** Lily avoids unknown state, hidden behavior, accidental timing, and uncontrolled side effects. Important behavior must be explicit and owned.

---

## Architecture and Dependencies

Once the basic expectations are clear, the next concern is structure. Lily systems should be created from focused modules with clear dependencies, controlled ownership, and reusable code that remains inside the Lily ecosystem.

### 5. ModuleScript-First Architecture

Lily Studio uses **ModuleScripts for the large majority of implementation code**, while normal `Script` and `LocalScript` files are kept small and limited to clear entry points. Feature logic belongs in modules because modules provide stronger boundaries, better reuse, easier testing, clearer typing, and a more controlled dependency structure.

> **Main rule:** Lily systems live in modules. Scripts are small entry points.

---

#### 5.1 Most Logic Belongs in ModuleScripts

Feature logic, controllers, state owners, utilities, runtime systems, UI builders, data handling, and reusable behavior should normally be placed in **ModuleScripts**.

**Preferred structure**

```text
Foo
├── fooController
├── fooEngine
├── fooIndex
├── fooRuntime
└── fooView
```

The exact names will depend on the feature, but the important part is that the behavior lives in focused modules rather than one large script.

---

#### 5.2 Scripts Should Mostly Start Systems

A normal `Script` or `LocalScript` should usually perform a limited amount of startup work and then hand control to modules.

**Preferred**

```lua
local foo = require(source:WaitForChild("foo"))

foo.start()
```

The entry script should not contain hundreds of lines of business logic when that logic can live in modules.

---

#### 5.3 Use Few Entry Scripts

Lily projects should avoid scattering many independent scripts throughout the hierarchy because each additional script creates another startup path, another **lifecycle** to understand, and another place where hidden behavior can begin.

A smaller number of intentional entry scripts makes execution order and ownership easier to understand.

Good uses for scripts include:

- server bootstrap
- client bootstrap
- environment-specific startup
- top-level initialization that cannot be represented as a reusable module

Everything below that level should normally be delegated to **ModuleScripts**.

---

#### 5.4 Modules Should Have Clear APIs

A module should expose only the functions or values that outside code actually needs.

**Preferred**

```lua
local module = {}

function module.start()
end

function module.destroy()
end

return module
```

Private helpers should remain local and should not be exported only because they exist.

---

#### 5.5 Avoid Script-to-Script Dependency Chains

One script should not depend on another script running first when a shared module can own the behavior instead.

Prefer:

```text
bootstrap
    ↓
module
    ↓
shared state / behavior
```

instead of:

```text
script A
    ↓
script B
    ↓
script C
```

A module-first architecture gives Lily Studio a clearer dependency graph and makes systems easier to move or replace later.

---

#### 5.6 Prefer Folder-Oriented Architecture

Lily Studio uses folders extensively to keep source organization explicit, scalable, and easy to navigate. Related modules, systems, features, packages, configuration, and runtime responsibilities should normally be grouped into clearly named folders instead of accumulating in large flat directories.

> **Architecture rule:** Lily prefers a structured folder hierarchy when folders make ownership, responsibility, or dependency boundaries clearer.
A folder should represent a meaningful architectural boundary such as:

- a feature
- a system
- a package
- a runtime domain
- a group of related modules
- shared utilities with one responsibility
- client-specific implementation
- server-specific implementation
- shared implementation
- configuration belonging to one system
- internal implementation behind a public module

**Preferred**

```text
source
├── client
│   ├── controllers
│   ├── interfaces
│   └── systems
├── server
│   ├── controllers
│   └── systems
├── shared
│   ├── packages
│   ├── types
│   └── utilities
└── runtime
    ├── state
    └── services
```

The exact hierarchy depends on the system, but the structure should communicate where code belongs before a developer opens a file.

---

#### 5.7 Prefer Multiple Focused Folders Over One Large Flat Directory

When a directory begins containing unrelated modules or too many responsibilities, Lily should separate those responsibilities into focused folders.

**Avoid**

```text
source
├── fooController
├── fooState
├── fooTypes
├── barController
├── barState
├── barTypes
├── bazController
├── bazState
└── bazTypes
```

**Preferred**

```text
source
├── foo
│   ├── controller
│   ├── state
│   └── types
├── bar
│   ├── controller
│   ├── state
│   └── types
└── baz
    ├── controller
    ├── state
    └── types
```

The second structure keeps related implementation physically grouped and reduces ambiguity as the codebase grows.

---

#### 5.8 Folder Names Must Be Descriptive

Folder names follow the same naming philosophy as code identifiers: they should communicate their responsibility without unexplained abbreviations.

**Preferred**

```text
controllers
configuration
interfaces
packages
runtime
services
systems
utilities
```

**Avoid**

```text
ctrl
cfg
intf
pkg
sys
util
```

> **Rule:** A developer should be able to infer what belongs in a folder from its name alone.

---

#### 5.9 Folders Should Reflect Ownership and Dependency Boundaries

Folder structure should reinforce the architecture rather than merely divide files visually.

When practical:

- client-only modules belong under a client boundary
- server-only modules belong under a server boundary
- shared modules belong under a shared boundary
- internal package implementation remains inside that package
- feature-specific helpers remain with the feature that owns them
- reusable behavior moves into an appropriate Lily-owned package
- configuration remains near the system that owns it unless it is genuinely shared

This reduces accidental cross-system dependencies and makes ownership easier to identify.

---

#### 5.10 Do Not Create Meaningless Folder Depth

Lily prefers substantial folder organization, but every folder must have a purpose.

**Avoid**

```text
source
└── systems
    └── runtime
        └── internal
            └── modules
                └── helpers
                    └── foo
```

when those levels do not represent real architectural boundaries.

Excessive depth increases navigation cost without improving ownership or clarity.

> **Rule:** Use as many folders as the architecture benefits from, but every level of the hierarchy must represent a meaningful distinction.

---

#### 5.11 Keep Folder Organization Predictable Across Similar Systems

Systems that serve similar architectural roles should normally use similar folder structures.

For example, if one feature stores its controller, state, types, and internal modules together, another feature with the same architectural shape should follow the same pattern unless there is a specific reason not to.

Consistency allows developers to navigate unfamiliar Lily systems by recognizing the structure before reading implementation details.

---

#### 5.12 Source Folders and Runtime Folders Follow Different Ownership Rules

Folders used to organize source code may be authored directly as part of the project structure.

Folders that exist because a Lily system requires them at runtime follow the **Script-Created Runtime Infrastructure** standard and should normally be created, configured, owned, and cleaned up through code.

> **Rule:** Source folders organize the codebase. Runtime folders are runtime infrastructure and follow explicit lifecycle ownership.

---

#### 5.13 Folder Structure Is Part of the Architecture

Folder layout should be considered during system design, not only after implementation becomes difficult to navigate.

A well-structured Lily system should make the following apparent from its hierarchy:

- where the system begins
- which modules belong together
- which code is client, server, or shared
- which modules are public
- which modules are internal
- where state is owned
- where reusable packages live
- where configuration belongs
- which boundaries should not be crossed casually

> **Final folder rule:** **Lily uses folders extensively because physical source organization is part of architectural clarity, ownership, scalability, and maintainability.**

### 6. Lily Packages Only

Lily Studio code should depend on **Lily-owned packages and modules** rather than packages maintained by outside organizations. Shared behavior that becomes part of Lily's architecture should remain inside Lily's package structure so its API, update process, compatibility, review standards, and long-term maintenance stay under Lily Studio's control.

> **Main rule:** Lily code uses **Lily packages**. Do not add outside organization packages as project dependencies.

---

#### 6.1 Prefer Lily-Owned Dependencies

When reusable functionality is needed, first use an existing Lily package or create a Lily-owned package for that responsibility.

**Preferred**

```lua
local foo = require(lilyPackages:WaitForChild("foo"))
```

The package should be maintained as part of the Lily codebase and follow the same coding convention described in this document.

---

#### 6.2 Do Not Add Third-Party Organization Packages

Do not add packages owned by unrelated organizations only because they already solve a small problem. External packages introduce another API style, another update schedule, another ownership boundary, and another source of changes that Lily Studio does not control.

Lily should not depend on outside package organizations for standard project architecture.

---

#### 6.3 Roblox APIs Are Still Normal Dependencies

This rule does not apply to Roblox's built-in APIs, services, types, enums, or engine functionality. Roblox itself is the platform and does not count as an outside organization package under this convention.

---

#### 6.4 Build Shared Lily packages When Reuse Is Real

If the same behavior is needed across several Lily systems, move the behavior into a focused Lily package instead of copying it between projects or importing an outside package.

A Lily package should:

- have one clearly defined responsibility
- expose a focused public API
- use `--!strict` only when the package is intentionally maintained with strict type checking
- follow Lily naming and formatting
- have a clear **lifecycle** if it creates runtime state
- avoid hidden global state
- remain straightforward to replace or update

> **Rule:** Reusable shared code should become a Lily package, not an uncontrolled outside dependency.

---

### 7. Composition Over Inheritance

Lily prefers composition, where small and focused systems work together through clear APIs, instead of relying on deep inheritance trees that spread behavior across several parent-child layers.

**Generic example**

```text
Foo
+ FooController
+ BarEngine
+ BazController
```

Composition is usually easier to understand, replace, test, reuse, and clean up than a long inheritance chain.

> **Rule:** Prefer focused parts working together over deep class hierarchies.

---

### 8. Generic Solutions

Repeated behavior should be generalized when multiple systems genuinely perform the same operation and can share one clear implementation without hiding important differences between them.

**Good example**

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

Do not force unrelated systems into one generic abstraction only because a few lines look similar. Generalization should reduce real duplication without hiding the meaning of the systems involved.

> **Rule:** Generalize repeated behavior, not unrelated concepts.

---

## State, Change, and Lifecycle

After the architecture is defined, state should have one owner, changes should happen through clear paths, and everything created by the system should have a matching **cleanup path**.

### 9. State Ownership

Predictable behavior begins with clearly identified ownership. Important state should have one identifiable owner, and that owner should define the approved paths through which the state can be read, changed, synchronized, and eventually cleaned up.

**Preferred**

```lua
local fooContexts = {}

fooContexts[fooKey] = {
	connections = {},
	destroyed = false,
	value = 0,
}
```

This makes the lifetime and ownership of the state clear.

---

#### 9.1 Avoid Unnecessary Global State

Global state makes dependencies difficult to follow and increases the chance that unrelated systems accidentally affect each other.

State should usually belong to:

- a module
- an object
- a context
- a controller
- a runtime instance
- another clearly identified owner

---

#### 9.2 Keep Related State Together

Values that describe the same feature should normally be stored together instead of being spread across unrelated top-level tables.

> **Rule:** The code that owns a value should also control the normal ways that value changes.

---

### 10. Public State

Public state should be controlled whenever direct mutation could bypass validation, skip required side effects, break an invariant, or leave the owning system in a state it was not designed to handle.

**Example**

```lua
function controller:getValue(): number
	return self.value
end

function controller:setValue(value: number)
	self.value = math.max(value, 0)
end
```

Do not create getters and setters for every private field automatically. Use them when the state actually has rules that should be enforced through an API.

---

### 11. Attributes Over ValueObjects

Lily prefers **Instance Attributes** for lightweight metadata and state that naturally belongs to an Instance, because **Attributes** keep lightweight data attached to its owner without adding unnecessary child objects to the hierarchy.

**Preferred**

```lua
object:SetAttribute("foo", true)
```

instead of creating a separate ValueObject:

```lua
local foo = Instance.new("BoolValue")
foo.Name = "foo"
foo.Value = true
foo.Parent = object
```

---

#### 11.1 Use Attributes for Simple Instance-Owned Data

**Attributes** are the normal choice for standalone values such as:

- booleans
- numbers
- strings
- identifiers
- modes
- small configuration values
- lightweight metadata
- simple state that belongs directly to an Instance

**Example**

```lua
object:SetAttribute("bar", 5)
object:SetAttribute("baz", "Foo")
object:SetAttribute("foo", true)
```

---

#### 11.2 React to Attribute Changes

Lily should use the Attribute's change signal instead of repeatedly reading it in a loop.

```lua
object:GetAttributeChangedSignal("foo"):Connect(function()
	updateFoo(object:GetAttribute("foo"))
end)
```

---

#### 11.3 Do Not Use ValueObjects Only for Simple Metadata

Lily normally avoids creating:

- `BoolValue`
- `NumberValue`
- `StringValue`
- `IntValue`
- `ObjectValue`

when the object exists only to store a simple piece of metadata that can be represented by an Attribute.

Complex runtime state should still live in Luau when a table, object, module, or context is the better owner.

> **Rule:** Use **Attributes** for lightweight Instance metadata and Luau state for complex runtime data.

---

### 12. Event-Driven Code

Lily should react at the moment state changes instead of continuously checking whether a change has occurred. Roblox already exposes signals for many common state transitions, so Lily should connect to the correct change source and run only when there is actual work to perform.

> **Main rule:** Lily responds to state transitions directly instead of repeatedly polling to determine whether a transition occurred.

---

#### 12.1 Do Not Poll for Normal State Changes

**Avoid**

```lua
while true do
	if object.Enabled ~= previousValue then
		previousValue = object.Enabled
		updateFoo(object.Enabled)
	end

	task.wait()
end
```

**Preferred**

```lua
object:GetPropertyChangedSignal("Enabled"):Connect(function()
	updateFoo(object.Enabled)
end)
```

---

#### 12.2 Use the Signal Closest to the Change

##### Property changes

```lua
object:GetPropertyChangedSignal("Enabled"):Connect(function()
	updateFoo(object.Enabled)
end)
```

##### Attribute changes

```lua
object:GetAttributeChangedSignal("foo"):Connect(function()
	updateFoo(object:GetAttribute("foo"))
end)
```

##### Value changes

```lua
valueObject.Changed:Connect(function(value)
	updateFoo(value)
end)
```

##### Remote events

```lua
remote.OnClientEvent:Connect(function(arguments)
	updateFoo(arguments)
end)
```

##### Input

```lua
userInputService.InputBegan:Connect(onInputBegan)
userInputService.InputEnded:Connect(onInputEnded)
```

---

#### 12.3 Do Not Use `Heartbeat` as a General Change Detector

`RunService.Heartbeat` is Lily's only continuous runtime loop mechanism, but it should still be used only when work genuinely needs to advance over time. It must not be used to repeatedly check whether ordinary state changed when an event or direct state update can represent that change.

**Avoid**

```lua
runService.Heartbeat:Connect(function()
	if value ~= previousValue then
		previousValue = value
		updateFoo(value)
	end
end)
```

Call the update where the state changes, or subscribe to the correct signal instead.

---

#### 12.4 Continuous Runtime Loops Use Only `RunService.Heartbeat`

When a Lily feature genuinely requires continuous frame-based work, the loop must be driven by `RunService.Heartbeat`.

> **Hard rule:** All continuous or repeating Lily runtime loops use `RunService.Heartbeat`. Lily does not use `while`, `repeat`, `RenderStepped`, `Stepped`, or `task.wait()` loops for continuous runtime work.
**Preferred**

```lua
local heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
	updateFoo(deltaTime)
end)
```

The connection must belong to a clear owner and must be disconnected when that owner is destroyed.

If the work only needs to happen when state changes, Lily should still use an event, signal, callback, or direct state update instead of `Heartbeat`.

> **Rule:** Event-driven behavior handles changes. `RunService.Heartbeat` handles only genuinely continuous runtime work.

---

### 13. Cleanup and Lifecycle

Everything Lily creates must have a clearly defined **lifecycle** and **cleanup path**. If a system creates a connection, runtime object, interface object, task, cache entry, context, or other owned resource, that same ownership model must also define when and how the resource is released.

This includes:

- `RBXScriptConnection`
- created Instances
- UI objects
- runtime contexts
- cached tables
- temporary objects
- scheduled tasks
- references that could prevent garbage collection

**Preferred**

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

When a context is destroyed:

```lua
fooContexts[fooKey] = nil
```

Setup should not continuously stack duplicate connections, duplicate UI, duplicate runtime objects, or duplicate tasks.

> **Rule:** If Lily creates something, Lily should know how to remove it.

---

With ownership established, the implementation should remain straightforward to follow at the function level. Lily favors descriptive names, one clearly defined responsibility, flat control flow, and direct iteration.

#### 13.1 Table cleanup Is Part of the lifecycle

Lily Studio uses tables extensively for runtime contexts, mappings, configuration, state, caches, collections, ownership records, and shared data structures, which makes table **cleanup** especially important. A table that is no longer needed should not continue holding references to objects, connections, callbacks, Instances, or other tables after its owner has been destroyed.

> **Hard rule:** If a Lily-owned table is part of runtime state, its contents must be released when that runtime state is destroyed.
A table can keep other objects alive even after those objects are no longer visible or useful. For that reason, cleaning up a Lily system means more than destroying Instances or disconnecting events; the tables that owned those references must also stop retaining them.

---

#### 13.2 Clear Owned Runtime Tables

When a table is fully owned by one runtime context and is no longer needed, clear it during **cleanup**.

**Preferred**

```lua
local function destroyFoo(fooContext)
	table.clear(fooContext.connections)
	table.clear(fooContext.data)
	table.clear(fooContext.objects)
end
```

If the table itself is stored by another owner, remove that reference as well.

```lua
fooContexts[fooKey] = nil
```

This allows Luau's garbage collector to reclaim the table and anything that is no longer referenced elsewhere.

---

#### 13.3 Remove Entries as Soon as They Become Invalid

Lily should not wait until a large shutdown operation to remove obviously stale entries from long-lived tables.

**Preferred**

```lua
fooContexts[fooKey] = nil
```

as soon as that context is permanently destroyed.

The same rule applies to caches, registries, mappings, lookup tables, and other long-lived collections.

> **Rule:** Stale table entries should be removed at the moment they stop being valid.

---

#### 13.4 Nested Tables Must Be Cleaned Intentionally

Clearing only the outer table is not always enough when nested tables have their own **lifecycle**, active connections, tasks, or references.

For example, if each entry owns connections, those connections must be disconnected before the table is cleared.

**Preferred**

```lua
for _, bar in fooContext.bars do
	for _, connection in bar.connections do
		connection:Disconnect()
	end

	table.clear(bar.connections)
end

table.clear(fooContext.bars)
```

The **cleanup** order should follow ownership: release the resources owned by each nested entry first, then remove the entries themselves.

---

#### 13.5 Disconnect Before Clearing Connection Tables

A connection table should never simply be cleared while the connections are still active.

**Avoid**

```lua
table.clear(fooContext.connections)
```

when the stored `RBXScriptConnection` objects are still connected.

**Preferred**

```lua
for _, connection in fooContext.connections do
	connection:Disconnect()
end

table.clear(fooContext.connections)
```

Removing the Lua reference does not disconnect the Roblox connection, so both parts of the **lifecycle** must be handled.

---

#### 13.6 Destroy Owned Instances Before Releasing Their References

If a table owns Instances that should no longer exist, destroy the Instances before clearing the table that references them.

**Preferred**

```lua
for _, object in fooContext.objects do
	object:Destroy()
end

table.clear(fooContext.objects)
```

If another system owns the Instance, Lily should only remove its own reference and must not destroy an object it does not own.

> **Rule:** **cleanup** follows ownership. Destroy what the table owns, release what it only references.

---

#### 13.7 Stop Background Work Before Clearing Its State

A table should not be cleared while a background task, callback, or recurring runtime path can still access it.

The normal **cleanup** order should be controlled:

```text
stop recurring work
    ↓
disconnect events
    ↓
destroy owned Instances
    ↓
clear nested runtime tables
    ↓
remove owner from registries
    ↓
release final references
```

This prevents callbacks from reading partially destroyed state or recreating references during **cleanup**.

---

#### 13.8 Do Not Keep Destroyed Owners in Registries

Long-lived registries are especially important because one stale entry can keep an entire runtime tree alive.

**Avoid**

```lua
fooContexts[fooKey] = fooContext
```

remaining after `fooContext` has already been destroyed.

**Preferred**

```lua
fooContexts[fooKey] = nil
```

during the same controlled destruction path.

A Lily registry should contain only currently valid owners.

---

#### 13.9 Caches Need an Invalidation and cleanup Rule

A cache should never grow indefinitely simply because values were useful once.

Every Lily cache should have a clear answer for:

- who owns the cache
- what creates an entry
- when an entry becomes invalid
- who removes the entry
- whether the cache has a maximum lifetime
- whether the cache is cleared when its owner is destroyed

If a cache has no invalidation or **cleanup** rule, it is not fully controlled.

---

#### 13.10 Avoid Retaining Large Objects Through Closures

Callbacks and closures can keep tables alive when they capture a large owner or runtime context.

When a connection or task is destroyed, the callback that captured the state should no longer have a path that retains the owner alive.

This is another reason Lily requires controlled connection **cleanup** and controlled background-task **cleanup**.

> **Rule:** A destroyed owner should not remain reachable only because an old callback still references it.

---

#### 13.11 Reuse Tables Only When Ownership Is Clear

Reusing a table can reduce allocation in hot paths, but reuse is only safe when the table has one clear owner and its previous contents are completely reset before reuse.

**Preferred**

```lua
table.clear(fooData)
```

before the same owned table is repopulated.

Do not reuse a table that may still be referenced by another system, callback, or consumer expecting the previous contents.

Optimization never overrides ownership.

---

#### 13.12 Do Not Replace cleanup With Garbage Collection Assumptions

Luau's garbage collector can reclaim unreachable tables, but Lily should not depend on garbage collection to solve ownership mistakes.

The code must first make unused state unreachable by:

- disconnecting active connections
- stopping tasks
- destroying owned Instances
- removing registry entries
- clearing owned tables
- removing final references

The garbage collector can only reclaim data after Lily has correctly released its references.

> **Hard rule:** Garbage collection is the final memory-recovery mechanism, not a substitute for **cleanup**.

---

#### 13.13 Table cleanup Should Be Easy to Explain

Because Lily uses many tables, the **cleanup path** should be just as understandable as the setup path.

A developer should be able to explain:

> "This owner stops its recurring work, disconnects its connections, destroys its owned objects, clears its tables, removes itself from the registry, and then releases the final reference."

If **cleanup** cannot be explained clearly, the ownership model is probably too complicated and should be simplified.

---

#### 13.14 cleanup Should Be Symmetrical With Setup

Every major setup action should have a corresponding **cleanup** action.

| Setup | **cleanup** |
| --- | --- |
| create connection | disconnect connection |
| create Instance | destroy owned Instance |
| insert registry entry | remove registry entry |
| create runtime table | clear/release runtime table |
| start background work | stop background work |
| create UI | destroy UI |
| cache owned data | invalidate/clear cache |

This symmetry makes memory behavior easier to reason about and reduces the chance that a resource is forgotten.

> **Final cleanup rule:** Lily **cleanup** is not optional housekeeping. It is part of the runtime design, especially because Lily relies heavily on tables to own and connect system state.

## Functions and Control Flow

### 14. Naming

Naming is one of the most important standards in Lily Studio.

A developer should be able to understand the role, ownership, and purpose of a value from its name before needing to inspect its implementation.

> **Hard rule:** All Lily-owned code identifiers use `camelCase`.

This includes:

- variables
- functions
- methods
- parameters
- callbacks
- table fields
- module fields
- public API names
- types
- constants
- configuration values
- state fields
- cache names
- connection names
- helper names

Lily does not switch to `PascalCase`, `snake_case`, or `SCREAMING_SNAKE_CASE` for different categories of internal identifiers.

---

#### 14.1 Use `camelCase` for Everything Lily Owns

**Preferred**

```lua
local profileToken
local selectedItems
local currentValue
local maximumAttempts

type profileContext = {
	name: string,
	isEnabled: boolean,
}

local function updateProfileState()
end

function module.getProfile()
end
```

**Avoid**

```lua
local ProfileToken
local selected_items
local CURRENT_VALUE
local MAXIMUM_ATTEMPTS

type profileContext = {
	Name: string,
	IsEnabled: boolean,
}

local function UpdateProfileState()
end

function module.GetProfile()
end
```

> **Rule:** Do not change casing conventions based on whether an identifier is a variable, function, type, constant, module field, or public method. Lily-owned identifiers remain `camelCase`.

---

#### 14.2 Use Full and Descriptive Words

Naming should favor clarity over shortness.

Do not abbreviate Lily-owned identifiers merely to reduce character count.

**Preferred**

```lua
local connection
local controller
local selectedItems
local currentValue
local profileContext
local remoteFolder
```

**Avoid**

```lua
local conn
local ctrl
local sel
local val
local ctx
local rf
```

> **Hard rule:** Lily Studio does not abbreviate meaningful internal identifiers. Use the complete descriptive name.

Short names are acceptable only when the meaning is universally obvious inside a very small scope.

**Acceptable**

```lua
for index, value in values do
end

local x = position.X
local y = position.Y
```

Do not shorten a meaningful name simply because it appears often.

---

#### 14.3 Names Must Communicate Meaning and Ownership

Avoid vague names when a more specific name is available.

**Preferred**

```lua
local profileConnections = {}
local pendingRequests = {}
local selectedItems = {}
local accessPermissions = {}
```

**Avoid**

```lua
local data = {}
local stuff = {}
local things = {}
local temp = {}
```

Generic names such as `data`, `value`, `result`, or `context` are acceptable when the surrounding scope already makes their meaning immediately obvious.

A name should answer as many of these questions as reasonably possible:

1. What does this value represent?
2. Who owns it?
3. What is it used for?
4. Is it state, configuration, a dependency, or a temporary value?

---

#### 14.4 Boolean Names Should Read Like Questions

Boolean values should clearly communicate true/false state.

**Preferred**

```lua
local isRunning
local hasAccess
local shouldUpdate
local wasCalled
local isFirstRun
```

These names make conditions read naturally:

```lua
if hasAccess then
end
```

Avoid vague boolean names when a boolean prefix makes the purpose clearer.

---

#### 14.5 Function Names Should Describe Actions

Functions should normally begin with a verb so the call site explains what is happening.

**Preferred**

```lua
updateState()
createProfile()
destroyContext()
sendRequest()
resolveValue()
releaseConnection()
```

**Avoid**

```lua
state()
profileThing()
valueData()
```

The function name should describe the operation rather than only naming the object it works with.

---

#### 14.6 Event Handlers Should Describe When They Run

Event callbacks should explain the event or timing that causes them to execute.

**Preferred**

```lua
onInputBegan()
onInputEnded()
onPlayerAdded()
onRemoteEvent()
onAttributeChanged()
```

This keeps event wiring predictable and easy to scan.

---

#### 14.7 Preserve Externally Owned Names

The `camelCase` rule applies to names Lily owns.

Do not rename Roblox API members, external protocol fields, serialized fields, existing required attributes, or other names whose spelling is controlled outside Lily.

**Examples**

```lua
player.UserId
part.CFrame
remoteEvent.OnServerEvent
```

If an external contract requires a specific field name, preserve that contract even when it does not match Lily naming.

> **Rule:** Lily-owned identifiers use `camelCase`. Externally owned names keep the spelling required by their owner.

---

#### 14.8 Naming Is Part of Code Quality

Naming is not cosmetic.

Poor names increase the amount of code a developer must inspect to understand behavior, ownership, state, and dependencies.

Before accepting a name, ask:

- is it `camelCase`?
- is it descriptive?
- is it unnecessarily abbreviated?
- does it communicate what the value actually represents?
- would the name still make sense at the call site?
- does it match the responsibility of the code?

> **Hard rule:** Do not sacrifice naming quality for shorter code. Clear naming is a core requirement of Lily Studio code.
### 15. Functions

Once naming is clear, functions should remain focused enough that their purpose can be understood without tracing several unrelated operations or hidden state changes. Each Lily function should have a defined responsibility, a visible execution path, and behavior that can be explained clearly in a short description.

**Preferred separation**

```lua
local function createFoo()
end

local function updateFoo()
end

local function destroyFoo()
end
```

A single function should not simultaneously handle validation, UI creation, networking, state mutation, **cleanup**, and unrelated runtime behavior unless those actions genuinely form one small operation.

---

#### 15.1 Create Helpers Only When They Add Meaning

A helper should reduce repeated logic, remove meaningful nesting, isolate a real responsibility, or make the main flow easier to read. Lily prefers compact block-based code and does not create helpers merely to relocate a few obvious lines.

##### Weak helper

```lua
local function setTrue(data)
	data.value = true
end
```

If this is used once and does not express a useful concept, it only adds another place to look.

##### Useful helper

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

This helper represents a real **lifecycle** operation and can be reused safely.

---

#### 15.2 Prefer Block-Based Function Organization

Lily Studio prefers code that is organized into **clear, compact execution blocks**. A function should read as a sequence of visible operations instead of being fragmented across excessive helpers, repetitive validation functions, or unnecessary type scaffolding.

> **Main rule:** Prefer organized blocks of related logic over excessive helper extraction. A helper should exist because it represents a real operation, not merely because several lines can be moved somewhere else.
A strong Lily function often reads in blocks such as:

```text
validate
    ↓
resolve
    ↓
prepare
    ↓
apply
    ↓
replicate
    ↓
return
```

Each block should have one understandable purpose and should remain visually separated from unrelated work.

**Preferred**

```lua
local function updateFoo(fooContext, arguments)
	if type(arguments) ~= "table" then return false end

	local fooName = arguments.fooName
	local enabled = arguments.enabled

	if type(fooName) ~= "string" then return false end
	if type(enabled) ~= "boolean" then return false end

	local fooData = fooContext.foos[fooName]
	if not fooData then return false end

	fooData.enabled = enabled
	updateFooView(fooData)

	return true
end
```

The function remains compact, while its validation, resolution, mutation, and return behavior remain visually distinct.

---

#### 15.3 Do Not Over-Extract Compact Logic

Lily does not split a straightforward operation into many small helpers only to reduce the number of lines in one function.

**Avoid**

```lua
local function getFooName(arguments)
	return arguments.fooName
end

local function getFooEnabled(arguments)
	return arguments.enabled
end

local function validateFooName(fooName)
	return type(fooName) == "string"
end
```

when those helpers exist only to support one small local operation.

**Preferred**

```lua
local fooName = arguments.fooName
local enabled = arguments.enabled

if type(fooName) ~= "string" then return false end
if type(enabled) ~= "boolean" then return false end
```

> **Rule:** Compact logic should remain local when keeping it together makes the execution path easier to read.

---

#### 15.4 Prefer Table-Driven Validation When Several Entries Share the Same Pattern

When several values, handlers, remotes, modes, or configuration entries follow the same validation pattern, Lily should prefer a compact table-driven block instead of manually validating every entry through a large repetitive function.

**Preferred**

```lua
local requiredNames = {
	"bar",
	"baz",
	"foo",
}

local resolved = {}

for _, name in requiredNames do
	local object = container:FindFirstChild(name)
	if not object then return nil end

	resolved[name] = object
end

return resolved
```

This is preferred over writing a separate local variable, guard, warning, and assignment block for every entry when the validation behavior is identical.

A table-driven validation block is especially appropriate when:

- every entry follows the same validation rule
- each key maps directly to one required object or handler
- additional entries may be added later
- a long repeated validation function would add no architectural meaning
- the resulting table becomes the controlled owner of the resolved values

> **Rule:** When validation is uniform, describe the requirements as data and process them through one clear block.

---

#### 15.5 Keep Helpers Focused and Substantial

A Lily helper should normally represent a meaningful operation such as:

- resolving a dependency group
- disconnecting owned connections
- validating one reusable data structure
- applying one state transition
- building one runtime object
- transforming one reusable value
- performing one lifecycle operation

Do not create helpers whose only purpose is to move two or three obvious lines out of an otherwise clear function.

> **Final block rule:** **Lily favors block-based code: related logic stays together, blocks are visually organized, helpers represent real operations, and tables replace repetitive validation when the behavior is uniform.**

---

### 16. Function Arguments

A function call should communicate enough intent that the reader can understand the operation without immediately opening the function definition to decode positional arguments or hidden behavior.

---

#### 16.1 Avoid Hidden Boolean Arguments

**Avoid**

```lua
updateFoo(foo, true, false)
```

The reader cannot easily tell what each boolean means.

**Preferred**

```lua
updateFoo(foo, {
	force = false,
	replicate = true,
})
```

Use an options table when several optional behaviors need names.

---

#### 16.2 Avoid Positional `nil`

**Avoid**

```lua
createFoo(name, nil, true)
```

**Preferred**

```lua
createFoo(name, {
	enabled = true,
})
```

Options tables should still be used only when they make the call clearer. Simple functions should remain simple.

---

### 17. Prefer Table-Driven Design

Lily Studio favors **tables for related data, configuration, mappings, handlers, and grouped runtime state** because tables keep connected information together and make systems easier to extend without adding scattered variables or repeated branching logic.

A table should be used when several values belong to the same concept, when several names map to related behavior, or when a system needs one clear structure that another function can read and process.

> **Main rule:** When several related values or behaviors clearly benefit from being grouped, Lily should usually represent them with a table. Simple standalone values may remain separate when that is clearer.

#### 17.1 Group Related Data Together

When several values describe the same object or concept, keep them together in one table instead of creating many separate variables that must remain synchronized manually.

**Preferred**

```lua
local fooData = {
	name = "Foo",
	enabled = true,
	value = 1,
}
```

**Also Allowed**

```lua
local fooName = "Foo"
local fooEnabled = true
local fooValue = 1
```

Separate variables are completely valid Lily code when each value is simple, local to the current scope, and does not need to travel through the system as one grouped object. A table should be introduced only when grouping the values gives the code clearer ownership, a reusable structure, a shared type, or a cleaner API.

> **Rule:** Lily likes **table-driven design**, but Lily does not force unrelated or standalone local values into tables.

---

#### 17.2 Use Tables for Mappings

Tables are preferred when one known value maps directly to another value or function.

**Preferred**

```lua
local handlers = {
	Bar = runBar,
	Baz = runBaz,
	Foo = runFoo,
}

local handler = handlers[mode]
if not handler then return end

handler()
```

This is usually clearer and easier to extend than a long `if` or `elseif` chain.

---

#### 17.3 Use Tables for Configuration

Related configuration should normally live in a structured table when the values describe one system or one operation.

**Preferred**

```lua
local fooConfig = {
	defaultValue = 1,
	maximumValue = 10,
	minimumValue = 0,
}
```

This retains the configuration grouped under one clear owner instead of spreading related settings across the file.

---

#### 17.4 Use Tables for Runtime Context

When a runtime system owns several related values, those values should usually be grouped under one context table.

**Preferred**

```lua
local fooContext = {
	connections = {},
	destroyed = false,
	value = 0,
}
```

The context becomes the owner of the state and makes the **lifecycle** easier to understand.

---

#### 17.5 Prefer Data-Driven Behavior Over Repeated Branches

If behavior can be described as data, Lily should normally prefer a **table-driven design** instead of repeating nearly identical code.

**Preferred**

```lua
local actions = {
	Bar = updateBar,
	Baz = updateBaz,
	Foo = updateFoo,
}
```

A **table-driven design** is especially useful when new entries may be added later because the system can often be extended by adding data instead of rewriting control flow.

---

#### 17.6 Do Not Use Tables Without a Reason

Lily prefers tables when they improve ownership, organization, extensibility, or readability, but a table should not be introduced only because tables are common.

**Preferred**

```lua
local value = 1
```

when only one independent value exists.

Do not wrap every single value inside a table when the table adds no structure or meaning.

> **Rule:** Lily uses tables heavily, but tables are not mandatory. Every table should represent a real grouping, mapping, collection, configuration, or owner, while simple standalone values may remain normal local variables.

---

#### 17.7 Keep Table Shapes Consistent

Tables that represent the same concept should use the same field names and structure throughout the codebase.

**Preferred**

```lua
local foo = {
	enabled = true,
	name = "Foo",
	value = 1,
}

local bar = {
	enabled = false,
	name = "Bar",
	value = 2,
}
```

Consistent table shapes make autocomplete stronger, types easier to define, and shared functions easier to reuse.

---

#### 17.8 Type Important Tables

Important or reusable tables should use named Luau types so their expected structure is explicit.

**Preferred**

```lua
type fooData = {
	enabled: boolean,
	name: string,
	value: number,
}

local fooData: FooData = {
	enabled = true,
	name = "Foo",
	value = 1,
}
```

This gives Lily the organizational benefits of tables without losing type safety.

---

#### 17.9 Keep Tables Focused

A table should represent one understandable concept. Do not turn one table into an unrelated collection of configuration, state, callbacks, temporary values, and unrelated objects simply because they can all be stored together.

If the table cannot be described clearly in a short sentence, its responsibilities should be separated.

> **Rule:** A Lily table should have a defined purpose, a predictable shape, and one understandable owner.

---

### 18. Guard Clauses

**guard clauses** are a standard Lily control-flow pattern because they keep the main execution path flat and make invalid or unsupported states visible near the top of the function. A guard should return early when continuing would be incorrect, unsafe, or unnecessary.

**Preferred**

```lua
local function updateFoo(fooContext)
	if not fooContext then return end
	if fooContext.destroyed then return end
	if not fooContext.foo then return end

	fooContext.foo.Enabled = true
end
```

**Avoid**

```lua
local function updateFoo(fooContext)
	if fooContext then
		if not fooContext.destroyed then
			if fooContext.foo then
				fooContext.foo.Enabled = true
			end
		end
	end
end
```

The preferred version makes every invalid condition visible at the beginning of the function, while the main behavior stays at the normal indentation level.

---

#### 18.1 Do Not Add Redundant Guards

**guard clauses** should protect real conditions, not repeat guarantees that were already established by the surrounding architecture.

**Avoid**

```lua
local function sendFoo(fooContext, value)
	if not fooContext then return end
	if not fooContext.remote then return end
	if not fooContext.remote.Parent then return end

	fooContext.remote:FireServer(value)
end
```

If `sendFoo()` is only called with a validated context, the clearer function is:

```lua
local function sendFoo(fooContext, value)
	fooContext.remote:FireServer(value)
end
```

> **Rule:** Lily uses necessary **guard clauses**, but removes checks that are already guaranteed elsewhere.

---

#### 18.2 Wrap Long Guard Conditions Cleanly

Short guard clauses should remain compact when they are easy to read.

**Preferred**

```lua
if not player then return false end
```

When one logical condition becomes too long for a clean single line, Lily should wrap the condition across multiple lines instead of forcing the entire expression onto one line.

**Avoid**

```lua
if not player or not player:IsA("Player") or type(accessType) ~= "string" or accessType == "" then return false end
```

**Preferred**

```lua
if not player
	or not player:IsA("Player")
	or type(accessType) ~= "string"
	or accessType == ""
then
	return false
end
```

Each continuation line should represent one clear part of the condition. Keep the logical operator at the beginning of the continuation line so the relationship between conditions is immediately visible.

Do not break a short condition across several lines only to satisfy a rigid formatting rule.

> **Rule:** Keep short guards on one line. When a guard becomes visually long, wrap each logical condition onto its own continuation line and place `then` and the guard action on clear separate lines.

---

### 19. No Conditional Nesting

Lily avoids **conditional nesting** because each additional level forces the reader to carry more conditions mentally while following the main path. Flat control flow retains decisions visible, reduces indentation, and creates later changes less likely to introduce hidden branches.

**Avoid**

```lua
if fooContext then
	if foo then
		updateFoo(foo)
	end
end
```

**Preferred**

```lua
if not fooContext then return end
if not foo then return end

updateFoo(foo)
```

The same rule applies inside loops, callbacks, event handlers, and public module methods.

---

#### 19.1 Use Helpers When Flat Code Becomes Too Large

If a function cannot stay flat without becoming difficult to read, move a meaningful decision into a focused helper rather than adding nested branches.

**Example**

```lua
local function canUpdateFoo(fooContext)
	if not fooContext then return false end
	if not fooContext.foo then return false end

	return fooContext.foo.Enabled
end

--————————————————————————————————————————————————————————————————————--

if not canUpdateFoo(fooContext) then return end

updateFoo(fooContext.foo)
```

The helper should still have one clearly defined responsibility and should not exist only to hide complexity.

> **Rule:** Lily uses **guard clauses**, `continue`, `break`, lookup tables, and focused helpers instead of nested conditionals.

---

#### 19.2 Minimize Nested Executable Code

Lily Studio strongly prefers flat executable code. Nesting should be minimized because each additional indentation level increases cognitive load, obscures execution paths, complicates cleanup, and makes ownership more difficult to reason about. Some nesting is unavoidable, so the goal is not absolute elimination; the goal is to keep every execution path as shallow and explicit as practical.

> **Core rule:** Lily code should use the least amount of executable nesting that is reasonably possible. Nesting is acceptable when removing it would reduce correctness, clarity, or maintainability.
This preference applies to:

- nested `if` statements
- nested loops
- nested callbacks
- nested anonymous functions
- nested event handlers
- deeply nested `pcall` or callback flows
- control flow embedded inside other control flow
- large blocks whose behavior depends on several indentation levels

**Avoid**

```lua
if foo then
	for _, bar in bars do
		if bar.enabled then
			bar.event:Connect(function()
				if baz then
					updateFoo(bar)
				end
			end)
		end
	end
end
```

**Preferred**

```lua
if not foo then return end
if not baz then return end

for _, bar in bars do
	if not bar.enabled then continue end

	connectBar(bar)
end
```

```lua
local function connectBar(bar)
	bar.event:Connect(function()
		updateFoo(bar)
	end)
end
```

The preferred structure separates responsibilities and keeps the main execution path visually flat.

---

#### 19.3 Extract Nested Responsibilities Into Focused Helpers

When a block begins requiring another layer of executable nesting, Lily should first consider moving that responsibility into a focused helper.

**Avoid**

```lua
for _, object in objects do
	if object.enabled then
		for _, value in object.values do
			if value > 0 then
				updateFoo(object, value)
			end
		end
	end
end
```

**Preferred**

```lua
local function updateObject(object)
	if not object.enabled then return end

	for _, value in object.values do
		if value <= 0 then continue end

		updateFoo(object, value)
	end
end

for _, object in objects do
	updateObject(object)
end
```

The purpose of extraction is not to create excessive numbers of helpers. It is to keep each execution path focused and understandable.

---

#### 19.4 Avoid Callback Pyramids

Nested callbacks make lifecycle ownership, error handling, and cleanup difficult to follow.

**Avoid**

```lua
foo.Event:Connect(function()
	bar.Event:Connect(function()
		baz.Event:Connect(function()
			updateFoo()
		end)
	end)
end)
```

Lily should instead create each connection through a clear owner and keep event wiring at one predictable level.

**Preferred**

```lua
fooConnection = foo.Event:Connect(onFoo)
barConnection = bar.Event:Connect(onBar)
bazConnection = baz.Event:Connect(onBaz)
```

```lua
local function onFoo()
	updateFoo()
end

local function onBar()
	updateBar()
end

local function onBaz()
	updateBaz()
end
```

---

#### 19.5 Avoid Nested Loops Whenever Responsibilities Can Be Separated

Nested loops are acceptable when the data relationship genuinely requires them and flattening would make the implementation less correct, less clear, or unnecessarily complex.

Before writing a nested loop, consider whether the operation should instead use:

- a precomputed lookup table
- an index
- a mapping
- cached relationships
- a focused helper
- a separate processing pass
- a more appropriate data structure

> **Rule:** Before accepting a nested loop, determine whether the data or execution path can be organized more clearly with a flatter structure. Keep the nesting when it is genuinely the clearest correct design.

---

#### 19.6 Indentation Depth Is a Design Signal

Increasing indentation should be treated as a design signal that responsibilities may be accumulating in one place.

A Lily function should normally have a shallow visual structure:

```text
validate
    ↓
resolve
    ↓
perform
    ↓
return
```

rather than:

```text
if
    ↓
    loop
        ↓
        if
            ↓
            callback
                ↓
                condition
```

When indentation begins increasing, reconsider the design before adding another nested block. If the additional nesting is still the clearest and most correct structure after that review, it is acceptable.

> **Rule:** Lily does not use indentation as the primary way to organize complexity. Complexity should be separated through architecture, helpers, data structures, and explicit ownership.

---

#### 19.7 Necessary Executable Nesting Is Allowed

Lily does not require developers to force every function into a completely flat shape when doing so would damage the implementation.

Executable nesting is acceptable when:

- the relationship is inherently hierarchical
- a nested loop directly represents the data relationship
- extracting the logic would make ownership less clear
- a helper would exist only to hide one trivial block
- flattening would duplicate work or state
- flattening would make control flow harder to understand
- the nested form is materially easier to verify for correctness

The standard is not **zero nesting**. The standard is **minimum necessary nesting**.

**Acceptable**

```lua
for _, foo in foos do
	for _, bar in foo.bars do
		updateBar(foo, bar)
	end
end
```

If the data model is naturally `foo -> bars`, this two-level iteration may be clearer than introducing indexes, temporary tables, or artificial helper functions purely to remove indentation.

> **Rule:** Do not flatten code mechanically. Prefer the shallowest structure that remains correct, readable, and faithful to the underlying data and ownership model.

---

#### 19.8 Structural Nesting Is Different From Executable Nesting

Some nesting is inherent to structured data and is not prohibited by this rule.

The following remain valid when they represent real structure:

- nested tables
- typed table shapes
- folder hierarchies
- configuration trees
- function-call arguments
- returned data structures

The restriction targets **nested executable behavior**, where one runtime path is embedded inside another.

> **Final rule:** Keep Lily execution paths as flat as reasonably possible. When nesting appears, first consider simplifying, extracting, reorganizing, or redesigning; retain the nesting only when it is genuinely necessary for correctness or clearer structure.

### 20. Avoid `else` and `elseif`

Lily prefers control flow that progresses downward in a direct and predictable path. `else` and `elseif` are avoided because they often introduce branch-heavy structures where an early return, guard clause, separate operation, or lookup table would express the same behavior more clearly.

---

#### 20.1 Avoid `else`

**Avoid**

```lua
if enabled then
	startFoo()
else
	stopFoo()
end
```

**Preferred**

```lua
if enabled then
	startFoo()
	return
end

stopFoo()
```

---

#### 20.2 Avoid `elseif`

**Avoid**

```lua
if mode == "Foo" then
	runFoo()
elseif mode == "Bar" then
	runBar()
elseif mode == "Baz" then
	runBaz()
end
```

**Preferred**

```lua
local handlers = {
	Bar = runBar,
	Baz = runBaz,
	Foo = runFoo,
}

local handler = handlers[mode]
if not handler then return end

handler()
```

A lookup table is not always required, but Lily should still avoid long conditional chains when a simpler structure exists.

---

### 21. Positive Conditions

Conditions and boolean names should normally be written in positive form because positive logic is easier to interpret at a glance and reduces the mental effort required to reason about inverted or double-negative conditions.

**Preferred**

```lua
if isRunning then
	updateFoo()
end
```

**Avoid**

```lua
if not isNotRunning then
	updateFoo()
end
```

The same rule applies to names:

```lua
local hasValue = value ~= nil
```

is usually clearer than:

```lua
local isMissingValue = value == nil
```

Negative names should only be used when the negative state is the actual concept being represented.

---

### 22. Avoid Boolean Expression Control Flow

Lily does not use chained `and/or` expressions as a substitute for explicit control flow, because the shorter expression often hides decision-making and becomes difficult to reason about when `false` or `nil` are valid values.

**Avoid**

```lua
local value = condition and foo or bar
```

Especially avoid long chains that choose between several values:

```lua
local result = firstCondition and foo
	or secondCondition and bar
	or baz
```

**Preferred**

```lua
if firstCondition then return foo end
if secondCondition then return bar end

return baz
```

The explicit version is easier to debug and behaves correctly when `false` or `nil` are valid values.

---

### 23. Iteration

Lily uses finite collection iteration when a collection genuinely needs to be processed, while avoiding loops whose only purpose is to wait, poll, or repeatedly ask whether ordinary state has changed.

---

#### 23.1 Use Generalized Luau Iteration

Lily does not use `pairs()` or `ipairs()`.

**Preferred**

```lua
for key, value in data do
end
```

```lua
for index, bar in bars do
end
```

**Avoid**

```lua
for key, value in pairs(data) do
end
```

```lua
for index, bar in ipairs(bars) do
end
```

> **Rule:** Use `for ... in table do` for Lily table iteration.

---

#### 23.2 Use `continue` to Keep Loop Bodies Flat

**Preferred**

```lua
for _, bar in bars do
	if not bar.Parent then continue end
	if bar:GetAttribute("disabled") then continue end

	updateBar(bar)
end
```

---

#### 23.3 Use `break` When the Work Is Complete

**Preferred**

```lua
for _, bar in bars do
	if bar.Name ~= fooName then continue end

	foundBar = bar
	break
end
```

A loop should not continue doing work after the required result has already been found.

---

## Types, Documentation, and Failures

Types and documentation should make an API easier to understand before it is used, while failure behavior should remain explicit and consistent with the rest of the code.

### 24. Type Checking

Type checking in Lily Studio is selective.

Not every file requires explicit Luau types, and not every file should use strict type checking.

> **Hard rule:** `--!strict` is used only when a file is intentionally written and maintained as strictly typed Luau.

Do not add `--!strict` to a file merely because it is production code, a ModuleScript, or part of Lily Studio.

---

#### 24.1 Strict Mode Is Only for Strictly Typed Files

A file using:

```lua
--!strict
```

must actually follow strict typing throughout the parts of the file that require it.

Use `--!strict` when the file benefits from strong compile-time contracts, such as:

- reusable libraries
- shared APIs
- engine internals
- complex reusable state
- important module contracts
- code where strict type checking materially improves correctness

Do not use `--!strict` as a decorative header.

**Preferred for a strictly typed file**

```lua
--!strict

type profileContext = {
	name: string,
	isEnabled: boolean,
}

local function getProfileName(context: profileContext): string
	return context.name
end
```

**Preferred for a file that does not need strict typing**

```lua
local function getProfileName(context)
	return context.name
end
```

**Avoid**

```lua
--!strict

local function getProfileName(context)
	return context.name
end
```

when the file is not actually being maintained as a strictly typed file.

> **Rule:** Strict mode and strict typing belong together.

---

#### 24.2 Explicit Typing Is Optional

Lily Studio does not require every variable, function parameter, return value, callback, or table to have an explicit type.

Use explicit types when they improve:

- correctness
- editor support
- readability
- reusable contracts
- shared module boundaries
- complex state definitions
- non-obvious data structures

Do not add types only to satisfy a formatting rule.

**Acceptable**

```lua
local contexts = {}

local function getContext(profileToken)
	return contexts[profileToken]
end
```

**Also acceptable when an explicit contract is useful**

```lua
type profileContext = {
	name: string,
	isEnabled: boolean,
}

local contexts: { [string]: profileContext } = {}

local function getContext(profileToken: string): profileContext?
	return contexts[profileToken]
end
```

The correct choice depends on the responsibility and complexity of the file.

> **Hard rule:** Do not make Lily code harder to read solely to increase the number of type annotations.

---

#### 24.3 Tables Do Not Always Need Explicit Types

Simple local tables may remain inferred.

**Acceptable**

```lua
local pendingRequests = {}
local connections = {}
local cachedValues = {}
```

Explicit table types are useful when the structure is important or reusable.

**Example**

```lua
type requestState = {
	isPending: boolean,
	requestId: string,
}

local requests: { [string]: requestState } = {}
```

Use the type because the structure matters, not because every table must have one.

---

#### 24.4 Derive Types From the Real Provider

When Lily code receives data from another Lily module, inspect the module that actually creates or returns that data before defining a consumer type.

Do not invent fields merely to make a broad generic structure.

For example, if a provider returns:

```lua
local entries = {
	foo = {
		kind = "primary",
		name = "Foo",
		isEnabled = true,
	},

	bar = {
		kind = "secondary",
		name = "Bar",
		count = 3,
	},
}
```

a typed consumer may define:

```lua
type primaryEntry = {
	kind: "primary",
	name: string,
	isEnabled: boolean,
}

type secondaryEntry = {
	kind: "secondary",
	name: string,
	count: number,
}

type entry =
	primaryEntry
	| secondaryEntry
```

Do not write an inaccurate broad structure such as:

```lua
type entry = {
	kind: string?,
	name: string?,
	isEnabled: boolean?,
	count: number?,
	value: number?,
	index: number?,
}
```

when the provider does not actually produce one shared shape containing those optional fields.

Before defining a receiving type:

1. inspect the provider
2. identify every valid returned shape
3. identify fields that are always present
4. identify fields that are genuinely optional
5. preserve meaningful literal values
6. use precise unions when multiple real variants exist
7. keep the consumer synchronized with the provider

If the consumer does not benefit from explicit typing, it may remain untyped.

> **Rule:** When types are used, they must describe the real contract rather than an invented approximation.

---

#### 24.5 Avoid Escape-Hatch Types

Lily Studio should not use broad escape-hatch types merely to silence the type checker.

Avoid:

```lua
any
```

and similarly broad placeholder typing when a real contract can be described.

If a value cannot be typed accurately without creating misleading or excessive complexity, reconsider whether that file should be strictly typed at all.

> **Rule:** Do not enable strict mode and then bypass it with inaccurate broad types.

---

#### 24.6 Runtime Validation Is Not the Same as Type Checking

Runtime validation and Luau type annotations solve different problems.

Use runtime checks when data can be invalid at runtime, especially when it comes from:

- clients
- remotes
- attributes
- JSON
- external modules
- user input
- dynamic Roblox hierarchy state

Example:

```lua
if type(profileName) ~= "string"
	or profileName == ""
then
	return false
end
```

A type annotation does not replace runtime validation when untrusted or dynamic data can still be invalid.

Likewise, do not add unnecessary runtime `type()` checks only because a local variable is not explicitly annotated.

> **Rule:** Validate values that can actually be wrong at runtime. Type values when static typing materially helps the code.

---

#### 24.7 Strictness Should Match the File

Lily files may reasonably fall into different categories:

**Untyped / inferred**

```lua
local function updateState(context, value)
	context.value = value
end
```

**Partially typed**

```lua
local function updateState(context, value: number)
	context.value = value
end
```

**Strictly typed**

```lua
--!strict

type stateContext = {
	value: number,
}

local function updateState(context: stateContext, value: number): ()
	context.value = value
end
```

All three can be valid Lily code.

The file should use the level of typing that best fits its responsibility.

> **Hard rule:** `--!strict` means the file is intentionally strict. If the file is not intentionally strict, do not add `--!strict`.
### 25. Comments and Documentation

Types describe the shape of an API, while comments should document the information that names and types cannot communicate by themselves. A Lily comment must have a defined purpose and should help another developer understand intent, constraints, ordering, side effects, ownership, or non-obvious behavior.

Comments should explain **why**, important behavior, unusual decisions, expectations, side effects, or API usage. They should not repeat simple code in plain English.

> **Main rule:** A Lily comment should write the code clearer to understand and maintain, not add noise around code that was already clear.

---

#### 25.1 Comments Must Be Proper and Useful

A comment should be complete enough to be understood by another developer who did not write the original code.

**Avoid**

```lua
-- update foo
updateFoo()
```

The function name already explains that the function updates `foo`, so the comment adds no useful information.

**Better**

```lua
-- Update the cached value before notifying listeners so every listener reads the new state.
updateFoo()
```

The second comment explains an important reason for the order of operations.

---

#### 25.2 Explain Why, Not What

Comments should usually explain why the code is written a certain way.

**Avoid**

```lua
-- Set foo to true.
foo = true
```

**Better**

```lua
-- Keep this enabled until cleanup so callbacks cannot recreate the object during destruction.
foo = true
```

The code already shows **what** happens. The comment explains **why** it happens.

---

#### 25.3 Comments Should Explain Non-Obvious Behavior

A comment is useful when the behavior would otherwise be surprising.

Good reasons for a comment include:

- a specific execution order is required
- a workaround exists because of a Roblox limitation
- a performance decision is not obvious
- **cleanup** must happen before another operation
- a value intentionally does not update immediately
- an API has an unusual requirement
- a function has an important side effect
- a temporary compatibility rule exists
- a section depends on behavior outside the current file

**Example**

```lua
-- Disconnect first so the callback cannot run again while the owner is being destroyed.
disconnectConnections(fooContext)
```

---

#### 25.4 Comments Must Stay Accurate

Incorrect comments are worse than missing comments because they give developers false information.

When behavior changes, any comment describing that behavior must be updated at the same time.

Do not leave comments that describe:

- old parameter names
- removed behavior
- old **state ownership**
- outdated workarounds
- old APIs
- behavior that no longer exists

> **Rule:** If the code changes, the related documentation must change with it.

---

#### 25.5 Use Documentation Comments for Important Functions

Important public functions, reusable helpers, shared package APIs, and functions with non-obvious parameters should use clear documentation comments when documentation improves the API.

Luau documentation comments can use `---` and tags such as `@param`, `@return`, and `@error` when those tags make the function easier to understand.

**Example**

```lua
--- Updates the stored value and notifies every registered listener.
--- @param fooContext FooContext The context that owns the value.
--- @param value number The new value that should be stored.
--- @return boolean True when the value changed.
local function updateFoo(fooContext: FooContext, value: number): boolean
	if fooContext.value == value then return false end

	fooContext.value = value
	notifyFoo(fooContext)

	return true
end
```

The documentation should explain the API, not duplicate the function body line by line.

---

#### 25.6 `@param` Should Add Meaning

`@param` is useful when a parameter needs more explanation than its type and name already provide.

##### Useful

```lua
--- @param timeout number Maximum number of seconds the operation may remain active.
```

##### Unnecessary

```lua
--- @param value number The value.
```

The second comment adds nothing because both the name and type already explain the parameter.

> **Rule:** Use `@param` when it adds context, limits, units, ownership, or behavior that is not obvious from the signature.

---

#### 25.7 Useful Documentation Tags

Use documentation tags only when they improve understanding.

Common useful tags include:

```text
@param
@return
@error
@within
```

Examples:

```lua
--- @param fooKey string Stable key used to identify the owner.
--- @param duration number Duration in seconds.
--- @return FooContext? The matching context, or nil when none exists.
```

Do not add tags only to make the comment block look larger or more formal.

---

#### 25.8 Public APIs Need Better Documentation Than Private Details

Shared Lily modules and **Lily packages** are used by other developers, so their public APIs should be especially clear.

A public function should make it straightforward to understand:

- what the function does
- what each important parameter means
- what it returns
- what state it changes
- whether it creates anything
- whether **cleanup** is required
- whether it can fail
- whether it has important side effects

Private helpers usually need less documentation when their name, types, and implementation already make the behavior obvious.

---

#### 25.9 Comments Are Not a Replacement for Clear Code

Do not keep confusing code and then try to explain the confusion with a large comment.

**Avoid**

```lua
-- This does several steps in a specific order because the data gets changed in multiple places
-- and then it checks whether bar exists before using baz, except when foo is active.
local function processData(...)
	-- complicated implementation
end
```

If the explanation is difficult because the code is difficult, improve the code first.

Prefer:

- clearer names
- smaller functions
- simpler **state ownership**
- fewer responsibilities
- **table-driven** organization for related data, mappings, handlers, configuration, and runtime state
- flat control flow
- focused helpers

Comments should support well-structured code, not compensate for poorly structured code.

---

#### 25.10 Keep Comments Professional

Comments are part of the Lily Studio codebase and should be written professionally.

Avoid:

- jokes that make behavior unclear
- temporary frustration comments
- insults
- vague notes such as `fix later`
- unexplained abbreviations
- comments written only for the original author
- comments that do not use normal grammar when a full explanation is needed

**Avoid**

```lua
-- idk why this breaks lol
```

**Better**

```lua
-- Roblox may return nil while the object is being reparented, so defer the lookup until the next task cycle.
```

---

#### 25.11 Use `TODO` Only When the Work Is Real and Specific

A `TODO` should explain exactly what remains to be done and why it is not being completed in the current change.

**Avoid**

```lua
-- TODO fix this
```

**Better**

```lua
-- TODO: Replace this compatibility path after the old data format is no longer supported.
```

Do not use `TODO` as permanent documentation for known broken behavior.

---

#### 24.12 Code Must Be Easy to Explain

A Lily developer should be able to explain a function in clear terms without needing a long technical speech.

If a function cannot be explained clearly, it is likely carrying too many responsibilities, hiding too much state, or using control flow that is too complicated.

> **Main rule:** If a function cannot be explained clearly in terms of its responsibility, inputs, state changes, and outputs, its design should be simplified or separated into more focused operations.
A good explanation should normally sound simple:

> "This function retrieves the current value, returns early when nothing changed, stores the new value, and notifies the listeners."

That explanation is short because the function has a clear responsibility.

A warning sign sounds more like:

> "This function sometimes changes the value, but it also creates objects, checks another system, sends data, updates several tables, and then does different **cleanup** depending on how it was called."

That function should probably be split into smaller operations.

---

#### 25.13 Functions Should Be Explainable From Their Structure

A clear function should make its behavior visible through:

- a descriptive name
- clear parameter names
- strong types
- necessary **guard clauses**
- no **conditional nesting**
- one main responsibility
- obvious state changes
- clear return behavior
- small focused helpers when needed

**Preferred**

```lua
local function setFooValue(fooContext: FooContext, value: number)
	if fooContext.value == value then return end

	fooContext.value = value
	updateFooView(fooContext)
	notifyFoo(fooContext)
end
```

This function is straightforward to explain:

> It ignores duplicate values, stores the new value, updates the view, and notifies listeners.

---

#### 25.14 Difficulty Explaining Code Is a Design Warning

If another Lily developer asks, "What does this function do?" and the answer requires explaining many unrelated systems, that is a sign that the function may need to be redesigned.

Possible improvements include:

1. split unrelated responsibilities into separate functions
2. move state to the system that owns it
3. replace nested logic with guards
4. give unclear variables better names
5. remove hidden side effects
6. create a focused helper for a real sub-operation
7. move reusable behavior into a module
8. make public behavior explicit through the function API

The goal is not to make every function tiny. The goal is to create every function understandable.

---

#### 25.15 Documentation Should Match the Level of Complexity

Simple code needs little documentation because the code itself should explain most of the behavior.

Complex public APIs may need more explanation because another developer cannot see the implementation from the call site.

A useful Lily documentation balance is:

| Code | Documentation |
| --- | --- |
| Simple private helper | Usually little or no comment |
| Non-obvious private helper | Short reason or behavior note |
| Shared public function | Clear documentation comment |
| Important Lily package API | Full purpose, parameters, return behavior, and important side effects |
| Workaround or unusual behavior | Explain why the unusual code is required |

This retains comments useful without covering every line with unnecessary text.

---

### 26. Error Handling

Lily should handle failures according to what the failure represents, separating expected recoverable conditions from invalid runtime input and from broken programming assumptions. The chosen response should leave the system in a known state and create the failure behavior straightforward to understand.

**Recoverable failure**

```lua
function module.start(fooKey: string): boolean
	if fooKey == "" then return false end

	return true
end
```

**Broken invariant**

```lua
assert(type(data) == "table", "Foo data must return a table.")
```

Use `warn()` when the system can continue but the problem is important enough to report.

Errors should not be used as explicit control flow.

---

## File Layout and Source Organization

The source file itself should also be predictable. Related declarations should stay together, top-level groups should follow the same order, and formatting should make large files quick to scan and navigate.

### 27. File Organization

Lily files should follow a consistent top-level structure so developers can quickly locate services, dependencies, state, types, private helpers, public APIs, and **cleanup** logic without learning a different layout for every file.

#### Recommended order

| Order | Section |
| ---: | --- |
| 1 | `--!strict` only when the file intentionally uses strict type checking |
| 2 | File description |
| 3 | Roblox services |
| 4 | Module table |
| 5 | Constants and configuration |
| 6 | State |
| 7 | Dependencies |
| 8 | Types, when used |
| 9 | Private helpers |
| 10 | Feature functions |
| 11 | Public module methods |
| 12 | `return module` |

**Example**

```lua

-- handles foo runtime

local replicatedStorage = game:GetService("ReplicatedStorage")
local userInputService = game:GetService("UserInputService")

--————————————————————————————————————————————————————————————————————--

local module = {}

--————————————————————————————————————————————————————————————————————--

local defaultFoo = 1

--————————————————————————————————————————————————————————————————————--

local fooContexts = {}

--————————————————————————————————————————————————————————————————————--

local source = replicatedStorage:WaitForChild("source")

--————————————————————————————————————————————————————————————————————--

type fooContext = {
	value: number,
}

--————————————————————————————————————————————————————————————————————--

local function getFoo(fooKey: string): FooContext?
	return fooContexts[fooKey]
end

--————————————————————————————————————————————————————————————————————--

function module.start()
end

--————————————————————————————————————————————————————————————————————--

return module
```

---

#### 27.1 Keep Top-Level Local Declarations at the Top

Lily Studio keeps top-level local declarations in the appropriate declaration sections near the top of the file.

> **Hard rule:** Do not introduce new top-level local variables, temporary startup tables, caches, or collections after the file has entered its function or runtime-execution sections.

Top-level locals should normally belong under one of these sections:

1. Roblox services
2. module table
3. constants and configuration
4. state
5. dependencies
6. types

After private functions begin, the file should remain focused on behavior rather than introducing new top-level state.

**Avoid**

```lua
local function watchFoo()
end

local initialFoos = container:GetChildren()
local initialBars = {}
local initialObjects = container:GetDescendants()

for _, object in initialObjects do
	-- startup work
end
```

The declarations appear after function definitions and make the file harder to scan because state and execution are mixed together.

**Preferred**

```lua
local initialFoos = container:GetChildren()

--————————————————————————————————————————————————————————————————————--

local function watchFoo()
end

--————————————————————————————————————————————————————————————————————--

for _, foo in initialFoos do
	watchFoo(foo)
end
```

When the temporary data does not need to remain top-level state, prefer removing the temporary table entirely and keeping startup execution direct.

**Preferred**

```lua
for _, foo in container:GetChildren() do
	if not foo:IsA("Folder") then continue end

	watchFoo(foo)
end
```

Do not create several top-level `initial...` tables only to make one startup pass more indirect.

> **Rule:** Declare top-level state at the top. Keep startup execution at the bottom compact, direct, and free of unnecessary temporary top-level locals.

---

#### 27.2 Do Not Mix Declarations With Startup Execution

Lily files should have a clear transition from declarations to functions to final startup execution.

A typical file should read in this order:

```text
services
    ↓
module
    ↓
configuration
    ↓
state
    ↓
dependencies
    ↓
types
    ↓
private functions
    ↓
public methods
    ↓
startup connections / one-time initialization
    ↓
return module
```

Once the file reaches startup execution, do not insert new helper functions or new top-level state below that point.

If startup logic becomes large enough that it needs several temporary tables or several passes, first consider whether the behavior belongs inside an existing focused helper.

> **Rule:** Lily files should not alternate between declarations, functions, new declarations, and execution. Each top-level section should appear once in a predictable order.

---

### 28. Alphabetical Top-Level Order

Declarations grouped near the top of a Lily file should be alphabetized **within their own logical section** whenever dependency order does not require otherwise. Consistent ordering makes files faster to scan, easier to compare in reviews, and less affected by personal ordering preferences.

The rule applies to grouped declarations such as:

- Roblox services
- required modules
- dependency references
- folder references
- related constants
- related configuration values
- other similar top-level declarations

Dependency order still takes priority when one declaration must exist before another can be created.

---

#### 28.1 Services

**Preferred**

```lua
local collectionService = game:GetService("CollectionService")
local httpService = game:GetService("HttpService")
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local tweenService = game:GetService("TweenService")
local userInputService = game:GetService("UserInputService")
```

---

#### 28.2 Required Modules

**Preferred**

```lua
local bar = require(source:WaitForChild("bar"))
local baz = require(source:WaitForChild("baz"))
local foo = require(source:WaitForChild("foo"))
```

---

#### 28.3 References

**Preferred**

```lua
local barFolder = source:WaitForChild("bar")
local bazFolder = source:WaitForChild("baz")
local fooFolder = source:WaitForChild("foo")
```

---

#### 28.4 Dependency Order Can Override Alphabetical Order

**Preferred when values depend on each other**

```lua
local maximumFoo = 10
local minimumFoo = 1
local fooRange = maximumFoo - minimumFoo
```

`fooRange` depends on the two values above it, so dependency order is more important than forcing every name into alphabetical order.

---

### 29. Lily Separators

Use the standard Lily separator between major file sections and top-level functions so large modules maintain a consistent visual rhythm and important boundaries remain easy to identify while scanning the file.

```lua
--————————————————————————————————————————————————————————————————————--
```

**Example**

```lua
local function foo()
end

--————————————————————————————————————————————————————————————————————--

local function bar()
end

--————————————————————————————————————————————————————————————————————--
```

The separator makes large files easier to scan and creates a consistent visual structure across Lily Studio projects.

---

### 30. Formatting

Lily formatting should remain compact enough to avoid unnecessary vertical space, while preserving enough visual structure that declarations, control flow, data tables, and function boundaries can be scanned quickly.

---

#### 30.1 Keep Related Lines Together

**Preferred**

```lua
local function setFoo(fooContext, value)
	fooContext.value = value
	updateFoo(fooContext)
end
```

**Avoid**

```lua
local function setFoo(fooContext, value)

	fooContext.value = value

	updateFoo(fooContext)

end
```

---

#### 30.2 Keep Simple Expressions on One Line

**Preferred**

```lua
local x = math.clamp((foo - bar) / baz, 0, 1)
```

Avoid breaking a simple expression across several lines when the one-line version remains straightforward to understand.

---

#### 30.3 Keep Large Tables Easy to Scan

**Preferred**

```lua
local data = {
	enabled = true,
	name = "Foo",
	value = 1,
}
```

Lily code should be compact, but not compressed to the point where the structure becomes hard to see.

---

#### 30.4 Use Parentheses in Mathematical Expressions

Lily Studio uses parentheses to make mathematical and compound expressions easier to read, organize, and evaluate. Even when Luau's operator precedence would already produce the correct result, parentheses should be added when they clarify how values are grouped or calculated.

> **Rule:** Math should be grouped with parentheses so the intended calculation is obvious without relying on the reader to remember operator precedence.
**Preferred**

```lua
local normalized = (value - minimumValue) / (maximumValue - minimumValue)
local result = (foo + bar) * baz
local offset = foo + (bar * baz)
local angle = (math.pi * 2) / count
```

**Avoid**

```lua
local normalized = value - minimumValue / maximumValue - minimumValue
local result = foo + bar * baz
local angle = math.pi * 2 / count
```

The preferred form makes each mathematical group visible and reduces ambiguity during review, refactoring, and debugging.

Parentheses are especially important when an expression combines:

- addition and subtraction
- multiplication and division
- modulo
- powers
- several function results
- normalization formulas
- ranges
- ratios
- interpolation calculations
- angle calculations
- chained arithmetic operations

**Preferred**

```lua
local range = maximumValue - minimumValue
local normalized = (value - minimumValue) / range
local result = minimumValue + (range * normalized)
```

For simple single operations, unnecessary parentheses are not required:

```lua
local total = foo + bar
local difference = foo - bar
local product = foo * bar
local ratio = foo / bar
```

> **Rule:** Use parentheses whenever they improve clarity, grouping, or the visual organization of an expression. Do not add them mechanically when the expression is already straightforward.

---

## Runtime Infrastructure and User Interface

Lily **runtime infrastructure** follows the same ownership rules as the rest of the codebase: required runtime objects are created through code, configured in source, owned by a clear system, and cleaned up through the same controlled **lifecycle**. User interfaces follow this rule as part of the broader **runtime infrastructure** standard.

### 31. Script-Created Runtime Infrastructure

Lily Studio **runtime infrastructure** should be created, configured, connected, and cleaned up through code so the complete structure of a system remains visible in **source control** and does not depend on hidden Studio setup. A developer should be able to recreate the runtime behavior of a Lily system from the codebase without manually rebuilding required networking objects, interface objects, folders, or other supporting Instances in Roblox Studio.

> **Main rule:** If an Instance exists because a Lily system needs it to function at runtime, Lily should normally create and own that Instance through code.
This rule applies to **runtime infrastructure** such as:

- `RemoteEvent`
- `RemoteFunction`
- `BindableEvent`
- `BindableFunction`
- runtime folders
- runtime configuration containers
- dynamically required helper Instances
- Lily-created UI
- runtime attachments or organizational objects
- temporary Instances
- generated runtime containers
- other Instances that exist because the code requires them

The purpose of this rule is to keep system behavior reproducible, controlled, and reviewable instead of relying on Studio objects that may be renamed, removed, duplicated, or configured differently without the code showing that change.

---

#### 31.1 Networking Objects Are Created Through Code

Lily should create its own networking objects instead of requiring developers to manually place `RemoteEvent` or `RemoteFunction` Instances in Studio.

**Preferred**

```lua
local remote = Instance.new("RemoteEvent")
remote.Name = "Foo"
remote.Parent = parent
```

When several networking objects are required, the system that owns them should create them during initialization and keep their names, parents, permissions, and **cleanup** behavior explicit.

**Avoid**

```text
ReplicatedStorage
└── FooRemotes
    ├── Bar
    └── Baz
```

when those objects must be manually created in Studio before the code can work.

A manually created runtime remote introduces hidden setup that is not represented by the implementation itself.

---

#### 31.2 Runtime Folders and Containers Are Created Through Code

Folders that exist only to organize or support Lily runtime systems should also be created by the owning code.

**Preferred**

```lua
local fooFolder = Instance.new("Folder")
fooFolder.Name = "Foo"
fooFolder.Parent = parent
```

Do not require developers to manually create runtime folders only because other Lily code expects a particular path to exist.

If a folder is part of the runtime architecture, its creation belongs to the runtime architecture.

---

#### 31.3 Bindables Are Created Through Code

Internal communication objects such as `BindableEvent` and `BindableFunction` should be created by the system that owns them rather than manually placed in Studio.

**Preferred**

```lua
local fooEvent = Instance.new("BindableEvent")
fooEvent.Name = "Foo"
fooEvent.Parent = parent
```

This retains ownership and **cleanup** obvious and prevents hidden dependencies between unrelated Studio objects.

---

#### 31.4 Lily UI Is Created Through Code

Lily Studio interfaces are created through code so their structure, properties, behavior, and **lifecycle** remain visible in **source control**. Lily should not depend on manually assembled Studio UI hierarchies for interfaces owned by Lily systems.

**Avoid**

```text
StarterGui
└── FooGui
    ├── Bar
    ├── Baz
    └── Foo
```

when the Lily runtime expects those objects to already exist.

**Preferred**

```lua
local fooFrame = interface.Frame(parent, {
	Name = "FooFrame",
	Position = UDim2.fromOffset(10, 10),
	Size = UDim2.fromOffset(300, 200),
})
```

The code that creates the interface should also own its important configuration, event connections, updates, and **cleanup**.

---

#### 31.5 Runtime Properties Are Configured Through Code

Creating an Instance through code is not enough if important properties still depend on manual Studio configuration. Runtime objects should be configured by the same code that creates them whenever those properties are required for correct behavior.

Important configuration may include:

- names
- parent relationships
- **Attributes**
- sizes
- positions
- visibility
- networking placement
- interface properties
- runtime flags
- ownership-related metadata
- other properties required by the system

**Preferred**

```lua
local foo = Instance.new("Folder")
foo.Name = "Foo"
foo:SetAttribute("enabled", true)
foo.Parent = parent
```

A Lily developer should not need to create the object in code and then remember to configure part of its required behavior manually in Studio.

---

#### 31.6 Runtime Objects Must Have Clear Ownership

Every script-created runtime Instance should have one clearly identified owner responsible for creating it, configuring it, using it, and removing it when the owning system is destroyed.

A developer should be able to answer:

- which module creates the object
- when the object is created
- where the object is parented
- which code may change it
- whether another system may reference it
- when the object is destroyed
- whether it is recreated during setup

If ownership is unclear, the runtime architecture should be simplified before more code depends on the object.

---

#### 31.7 Setup Must Not Create Duplicate Infrastructure

Initialization should be deterministic and safe to run according to the **lifecycle** defined by the owning system. Lily should not create duplicate remotes, duplicate folders, duplicate UI, duplicate bindables, or duplicate runtime containers because setup was called more than once.

The system should either reuse the valid object it already owns or cleanly replace the old object according to the intended **lifecycle**.

**Preferred pattern**

```lua
local foo = parent:FindFirstChild("Foo")
if foo then return foo end

foo = Instance.new("Folder")
foo.Name = "Foo"
foo.Parent = parent

return foo
```

This pattern is appropriate only when reuse is part of the intended ownership model. Lily should not add existence checks automatically when the architecture already guarantees one controlled creation path.

---

#### 31.8 runtime infrastructure Must Be Reproducible

A Lily project should not depend on a developer remembering a list of manual Studio steps before the code can function.

For **runtime infrastructure**, cloning the project and running the intended bootstrap path should be enough for Lily to create the objects it owns.

The codebase should define:

1. what is created
2. when it is created
3. where it is created
4. how it is configured
5. which system owns it
6. how it is cleaned up

This makes setup easier to review, easier to reproduce, and less likely to behave differently between development environments.

---

#### 31.9 Authored Content Is Different From runtime infrastructure

This rule does not mean every object in a Roblox experience must be generated through Luau.

Manually authored content may still exist when the object is genuinely part of the designed world or asset content rather than Lily **runtime infrastructure**. Examples may include a map, a model created by an artist, a physical environment object, or another asset whose purpose is visual or authored content.

The distinction is ownership:

> If Lily **needs the object because the system architecture requires it**, Lily should create it through code.

> If the object exists as **authored game or world content**, it may be created outside the runtime code when that is the intended content workflow.

This retains Lily systems fully controlled without forcing artistic or world-building content into unnecessary procedural creation.

---

#### 31.10 Do Not Hide Required Setup in Studio

A Lily module should not silently assume that a required runtime object has already been created by hand.

**Avoid**

```lua
local fooRemote = replicatedStorage:WaitForChild("FooRemote")
```

when `FooRemote` is Lily-owned **runtime infrastructure** and no Lily code creates it.

**Preferred**

```lua
local fooRemote = Instance.new("RemoteEvent")
fooRemote.Name = "FooRemote"
fooRemote.Parent = replicatedStorage
```

or a shared Lily-owned creation helper when several systems follow the same infrastructure pattern.

The important requirement is that the creation path remains inside Lily code.

---

#### 31.11 Use Lily packages for Shared Creation Patterns

If several Lily systems need the same creation behavior, that behavior should be placed in a Lily-owned module or package instead of copied between systems or delegated to an outside dependency.

For example, Lily may use shared internal helpers for:

- creating networking folders
- creating remotes
- building interfaces
- creating runtime containers
- registering **cleanup**
- applying standard **Attributes**
- establishing common ownership patterns

The helper should still keep the final ownership and **lifecycle** clear to the caller.

---

#### 31.12 Script-Created Infrastructure Must Be Cleanable

Creating **runtime infrastructure** through code also means Lily is responsible for removing or replacing it correctly.

A system that creates an Instance must define whether that Instance:

- lives for the entire server or client lifetime
- lives for one context
- lives for one runtime owner
- is temporary
- is replaced during reinitialization
- is destroyed when the owner is destroyed

**Example**

```lua
local function destroyFoo(fooContext)
	if not fooContext.foo then return end

	fooContext.foo:Destroy()
	fooContext.foo = nil
end
```

Creation without **lifecycle** ownership is incomplete architecture.

---

#### 31.13 source control Should Describe the Runtime Structure

One of the main reasons Lily creates infrastructure through code is that important runtime architecture should be reviewable from the repository.

A reviewer should be able to see changes to:

- remote names
- folder names
- **Attributes**
- interface structure
- object relationships
- default configuration
- ownership
- initialization
- **cleanup**

without needing to compare a separate manually edited Studio hierarchy.

> **Rule:** Lily **runtime infrastructure** belongs in source-controlled code, not hidden manual setup.

---

## Performance and Scale

Performance rules sit together because they describe how Lily controls repeated work, **CPU usage**, idle runtime behavior, memory ownership, **cleanup**, scaling behavior, and Luau runtime limits without sacrificing correctness or readability.

### 32. Performance

After correctness, ownership, and **lifecycle** are established, Lily should consider the cost of repeating the same work at scale. An operation that is insignificant during one-time setup can become expensive when it executes every frame, across large collections, or through many active systems at once.

Performance work should focus first on repeated work rather than one-time setup work.

---

#### 32.1 Cache Repeated References

Avoid repeatedly searching the hierarchy in hot code when a stable reference can be resolved once and reused.

---

#### 32.2 Avoid Unnecessary Per-Frame Allocation

Be careful about creating new tables, closures, arrays, temporary objects, or other garbage inside code that runs every frame.

---

#### 32.3 Prevent Duplicate Connections

Setup should not repeatedly connect the same events without cleaning the previous connections first.

---

#### 32.4 Avoid Repeated Runtime `WaitForChild`

Required references should normally be resolved during setup rather than repeatedly searched inside runtime update paths.

---

#### 32.5 Optimize Real Hot Paths

The most important areas include:

- frame updates
- scheduler work
- repeated object updates
- high-frequency callbacks
- frequently called networking paths
- large collection processing

Lily should still favor readable solutions and should not make ordinary code difficult to understand for a meaningless micro-optimization.

> **Rule:** Optimize repeated work first, while keeping the code clear enough to maintain.

---

#### 32.6 Never Resolve Hierarchy Inside Controlled Loops

Controlled loops, frame updates, schedulers, recurring callbacks, and other repeated runtime paths must never perform hierarchy discovery or wait for Instances to appear. Any object that the loop depends on should be resolved during setup, stored by the owning system, and reused directly while the loop is active.

> **Hard rule:** Never use `WaitForChild()` or similar hierarchy lookup work inside a controlled or repeated loop.
**Avoid**

```lua
runService.Heartbeat:Connect(function()
	local foo = parent:WaitForChild("Foo")
	updateFoo(foo)
end)
```

**Preferred**

```lua
local foo = parent:WaitForChild("Foo")

runService.Heartbeat:Connect(function()
	updateFoo(foo)
end)
```

The first version performs a hierarchy operation every time the callback runs and may also yield if the expected object is temporarily unavailable. The second version resolves the dependency once during setup, after which the hot path uses the cached reference directly.

This rule also applies to repeated use of hierarchy-discovery operations such as:

- `WaitForChild()`
- `FindFirstChild()`
- `FindFirstChildWhichIsA()`
- `FindFirstChildOfClass()`
- `GetChildren()`
- `GetDescendants()`
- repeated path traversal through `Parent` or child indexing
- repeated service or dependency resolution that can be completed during setup
- repeated `require()` calls for dependencies that should already be cached

These operations are not forbidden throughout Lily code. They are forbidden inside controlled repeated paths when the dependency can be resolved before the repeated work begins.

#### 32.7 Resolve Once, Then Reuse

The standard Lily pattern is:

```text
setup
    ↓
resolve dependencies
    ↓
validate required objects
    ↓
cache references
    ↓
start controlled runtime work
    ↓
reuse cached references
```

A repeated runtime path should work with data and references that are already available. It should not discover its own dependencies every time it runs.

**Preferred**

```lua
local fooContext = {
	foo = parent:WaitForChild("Foo"),
}

local function update()
	updateFoo(fooContext.foo)
end
```

This makes the runtime behavior more deterministic because the system either completes setup with the required dependency or does not begin the repeated work.

#### 32.8 Controlled Loops Must Not Yield

A controlled loop should not unexpectedly pause because one iteration is waiting for another object, dependency, or piece of setup to become available.

Avoid yielding operations inside controlled repeated work, including any operation whose purpose is to wait for setup that should already have completed.

If a dependency may genuinely appear later, Lily should handle that through a deliberate **event-driven** **lifecycle** rather than placing a wait inside the hot loop.

> **Rule:** Setup resolves dependencies. Controlled loops only perform the work they were created to perform.

#### 32.9 Background Loops Are Forbidden by Default

Lily Studio should not run background loops simply to keep a script active, repeatedly check state, refresh values, or perform work that can be triggered by an event, signal, callback, or direct state change.

> **Hard rule:** Lily does not use `while`, `repeat`, `task.wait()`, `RenderStepped`, or `Stepped` as runtime loop mechanisms.
**Avoid**

```lua
while true do
	updateFoo()
	task.wait()
end
```

```lua
repeat
	updateFoo()
	task.wait(.1)
until stopped
```

```lua
runService.RenderStepped:Connect(function()
	updateFoo()
end)
```

These patterns are not part of Lily's runtime loop standard.

---

#### 32.10 Necessary Continuous Work Uses `RunService.Heartbeat`

If a feature genuinely requires continuous or repeating runtime work, Lily uses `RunService.Heartbeat`.

**Preferred**

```lua
local heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
	updateFoo(deltaTime)
end)
```

> **Hard rule:** `RunService.Heartbeat` is the only approved Lily mechanism for continuous runtime loops.

---

#### 32.11 Prefer Event-Driven Work When Continuous Work Is Not Required

`Heartbeat` should not be used when an event can describe the change directly.

**Avoid**

```lua
runService.Heartbeat:Connect(function()
	local value = object:GetAttribute("foo")
	if value == previousValue then return end

	previousValue = value
	updateFoo(value)
end)
```

**Preferred**

```lua
object:GetAttributeChangedSignal("foo"):Connect(function()
	updateFoo(object:GetAttribute("foo"))
end)
```

---

#### 32.12 Every `Heartbeat` Loop Must Have a Clear Owner

Every `Heartbeat` connection must have a clearly identified owner responsible for starting it, storing the connection, and disconnecting it during cleanup.

**Preferred**

```lua
fooContext.heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
	updateFoo(fooContext, deltaTime)
end)
```

---

#### 32.13 Every `Heartbeat` Loop Must Stop With Its Owner

A `Heartbeat` connection must be disconnected when the context, object, controller, or runtime system that owns it is destroyed.

**Preferred**

```lua
local function destroyFoo(fooContext)
	if not fooContext.heartbeatConnection then return end

	fooContext.heartbeatConnection:Disconnect()
	fooContext.heartbeatConnection = nil
end
```

---

#### 32.14 Do Not Stack `Heartbeat` Loops

Setup, restart, or reinitialization logic must not create another `Heartbeat` connection while the existing one for the same owner is still active. If a loop must restart, disconnect the old connection before creating the replacement.

---

#### 32.15 `Heartbeat` Loops Must Never Resolve Dependencies

A controlled `Heartbeat` loop must work only on references and data that were resolved before the loop started.

It must never contain:

- `WaitForChild()`
- `FindFirstChild()`
- `FindFirstChildWhichIsA()`
- `FindFirstChildOfClass()`
- `GetChildren()`
- `GetDescendants()`
- repeated `require()`
- repeated dependency resolution
- unexpected yielding

**Preferred flow**

```text
setup
    ↓
resolve dependencies
    ↓
cache references
    ↓
connect Heartbeat
    ↓
perform controlled runtime work
    ↓
disconnect during cleanup
```

> **Rule:** Initialization resolves and validates dependencies before runtime begins; `Heartbeat` performs only the continuous work explicitly assigned to that runtime path.

---

#### 32.16 Necessary `Heartbeat` Loops Should Be Documented

When the reason for continuous work is not immediately obvious, the code should explain why `Heartbeat` is required.

**Example**

```lua
-- Heartbeat advances time-based state because progression must continue every frame.
fooContext.heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
	updateFoo(fooContext, deltaTime)
end)
```

> **Final rule:** Lily permits no uncontrolled recurring execution paths. Event-driven code handles changes, and every genuinely continuous runtime loop is an owned, cleanable `RunService.Heartbeat` connection.

#### 32.17 Optimization Is Part of the Design

Lily Studio does not treat optimization as something that is added only after a system begins to lag. Performance should be considered while the architecture is being designed so the normal implementation already avoids unnecessary work, excessive allocation, uncontrolled background activity, and resources that remain alive after their owner is gone.

> **Main rule:** Lily code should be designed to remain efficient, stable, and predictable as the amount of work increases.
Optimization should focus on reducing work that is repeated frequently, removing unnecessary allocations, preventing duplicate runtime behavior, and making sure every created resource has a controlled **lifecycle**.

---

#### 32.18 memory leaks Are Not Acceptable

Lily systems must not leave behind references, connections, tasks, Instances, tables, callbacks, or runtime contexts after the system that owns them has been destroyed.

Common causes of **memory leaks** include:

- `RBXScriptConnection` objects that are never disconnected
- tables that retain references to destroyed objects
- background tasks that continue after their owner is gone
- closures that retain large runtime objects alive
- duplicate runtime contexts
- UI that is recreated without destroying the previous UI
- cached objects that are never removed
- events that retain callbacks indefinitely
- objects that are removed from the hierarchy but still referenced by Luau state

A **cleanup path** should release every resource the owner created.

**Preferred**

```lua
local function destroyFoo(fooContext)
	for _, connection in fooContext.connections do
		connection:Disconnect()
	end

	table.clear(fooContext.connections)
	table.clear(fooContext.data)

	if fooContext.object then
		fooContext.object:Destroy()
		fooContext.object = nil
	end
end
```

The exact **cleanup** depends on the system, but the ownership rule remains the same.

> **Hard rule:** Lily should not leave memory behind after an owner is destroyed.

---

#### 32.19 Avoid High CPU usage

Lily code should not perform work more often than the feature requires.

High **CPU usage** is often caused by:

- unnecessary frame updates
- polling loops
- duplicate event connections
- repeated hierarchy searches
- repeated table construction in hot paths
- repeated `require()` calls
- processing unchanged state
- updating every object when only one object changed
- recalculating values that could be cached
- running the same operation from several systems at once

The preferred Lily approach is to perform work only when there is a reason to perform it.

**Preferred flow**

```text
state changes
    ↓
affected system is notified
    ↓
only required work runs
    ↓
cached state is updated
```

**Avoid**

```text
background loop
    ↓
check everything
    ↓
nothing changed
    ↓
repeat forever
```

> **Rule:** CPU time should be spent on actual work, not repeated checking.

---

#### 32.20 Avoid Unnecessary Allocation

Frequently executed code should avoid creating temporary tables, closures, Instances, arrays, or other short-lived objects unless the operation genuinely requires them.

**Avoid in a hot path**

```lua
local function updateFoo()
	local data = {
		value = currentValue,
	}

	applyFoo(data)
end
```

when the table can be reused or the value can be passed directly.

Every allocation is small by itself, but allocations repeated every frame or across large collections create more garbage for Luau to collect.

> **Rule:** Hot code should reuse stable data where practical instead of continuously creating temporary objects.

---

#### 32.21 Cache Expensive or Repeated Results

If a result is stable and used repeatedly, Lily should normally calculate or resolve it once and store it with the system that owns it.

Useful values to cache may include:

- resolved Instance references
- parsed configuration
- calculated constants
- lookup tables
- frequently used mappings
- reusable state
- compiled runtime data

Do not cache values whose correctness depends on changing data unless the cache has a clear invalidation path.

> **Rule:** Cache repeated work only when the cached value has clear ownership and a clear rule for becoming invalid.

---

#### 32.22 Avoid Duplicate Work

Lily should not allow several systems to perform the same expensive operation independently when the result can be calculated once and shared through a controlled owner.

Duplicate work may include:

- several loops reading the same state
- several callbacks rebuilding the same data
- several systems searching the same hierarchy
- several modules calculating the same derived value
- multiple network sends representing the same state change

When shared work is appropriate, one system should own the calculation and expose the result through a clear API or state update.

---

#### 32.23 Update Only What Changed

Lily should prefer targeted updates instead of rebuilding or recalculating an entire system when only one part changed.

**Preferred concept**

```lua
local function setFooValue(fooContext, value)
	if fooContext.value == value then return end

	fooContext.value = value
	updateFoo(fooContext)
end
```

The early return avoids unnecessary work when the requested state already matches the current state.

This pattern is especially important for:

- UI updates
- network replication
- large collections
- expensive calculations
- runtime state synchronization

> **Rule:** If the relevant state has not changed, Lily should avoid performing unnecessary runtime work.

---

#### 32.24 Performance Must Remain Predictable Under Scale

A system that behaves correctly well with one object but becomes disproportionately expensive with many objects should be reviewed before it becomes a production problem.

When writing code that may process many values or objects, consider:

- how often the function runs
- how many entries it processes
- whether the work grows linearly or worse
- whether temporary memory grows over time
- whether **cleanup** removes old state
- whether events can become duplicated
- whether repeated work can be shared or cached
- whether unchanged objects can be skipped

The goal is not to optimize imaginary problems. The goal is to avoid architecture that obviously becomes expensive when the same operation is repeated at scale.

---

#### 32.25 Performance Optimizations Must Preserve Correctness

Optimization must never make behavior unpredictable, unsafe, or difficult to maintain.

Do not remove required validation, **lifecycle** handling, ownership, or synchronization only to save a limited amount of CPU time.

A good optimization should normally do one or more of the following:

- reduce repeated work
- remove unnecessary allocation
- reduce hierarchy access
- reduce duplicate callbacks
- reuse stable references
- skip unchanged state
- improve **cleanup**
- reduce unnecessary networking
- simplify a hot path

while preserving the same intended behavior.

> **Rule:** Lily removes unnecessary runtime cost without compromising correctness, ownership, validation, or deterministic behavior.

---

#### 32.26 Every Resource Has a Lifetime

Memory and CPU ownership should be treated as part of the system **lifecycle**.

A Lily developer should be able to explain:

- what starts the work
- what retains the work alive
- what data the work owns
- what resources it allocates
- what event or function stops it
- what is removed during **cleanup**
- whether anything can continue after destruction

If those answers are unclear, the **lifecycle** is not controlled enough.

---

#### 32.27 Performance Problems Should Be Prevented, Not Hidden

Do not solve high CPU or memory usage by hiding symptoms while leaving the underlying repeated work or ownership problem in place.

Examples of weak fixes include:

- adding longer waits to an unnecessary polling loop
- suppressing warnings caused by duplicate initialization
- reducing update frequency when the update should be **event-driven**
- clearing one table while another reference still retains the same objects alive
- adding more checks around a system that should have one controlled owner

Lily Studio should correct the source of the waste whenever possible.

> **Rule:** Lily code should use only the CPU, memory, networking, and runtime work that the feature actually requires, and every allocated resource must have an intentional owner and **cleanup path**.

---

#### 32.28 Idle Systems Must Be Dormant

A Lily system that has no active work should not continue executing recurring runtime code merely because the system exists.

A system is **idle** when no current interaction, animation, transition, scheduled operation, active request, stream, simulation, or state change requires processing.

While idle, Lily should not continuously:

- execute frame callbacks
- poll state
- inspect input state
- rebuild or refresh UI
- write unchanged Instance properties
- resend unchanged network state
- recalculate unchanged derived values
- traverse the hierarchy looking for changes
- process empty queues
- recreate temporary tables or closures
- advance timers that do not currently serve active work
- run callbacks caused only by unrelated global activity

Passive event subscriptions are allowed when they wake only because a relevant external state actually changed. Merely keeping an owned `RBXScriptConnection` registered does not violate the idle rule when that signal is naturally quiet and scoped to the state the system owns.

> **Hard rule:** When a Lily feature has nothing to do, it must perform no self-generated or recurring runtime work.

**Preferred idle model**

```text
idle
    ↓
no recurring execution
    ↓
relevant event occurs
    ↓
perform required work once
    ↓
return to idle
```

---

#### 32.29 An Early Return Does Not Make a High-Frequency Callback Idle

A callback can still consume runtime time even when its first instruction immediately returns. If the signal itself fires frequently, leaving the connection active still wakes the Lua callback repeatedly.

**Avoid**

```lua
fooContext.heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
	if not fooContext.isActive then return end
	updateFoo(fooContext, deltaTime)
end)
```

The callback above still runs every Heartbeat while inactive.

The same problem can occur with frequently firing signals such as pointer movement, input-change streams, render or simulation updates, broad hierarchy changes, or other high-volume event sources.

**Preferred**

```lua
local function stopFooRuntime(fooContext)
	local connection = fooContext.heartbeatConnection
	if not connection then return end

	connection:Disconnect()
	fooContext.heartbeatConnection = nil
end

local function startFooRuntime(fooContext)
	if fooContext.heartbeatConnection then return end

	fooContext.heartbeatConnection = runService.Heartbeat:Connect(function(deltaTime)
		updateFoo(fooContext, deltaTime)
	end)
end
```

> **Rule:** If a high-frequency callback is unnecessary while inactive, disconnect it instead of keeping it alive behind a guard clause.

---

#### 32.30 Continuous Runtime Connections Exist Only While Continuous Work Is Active

A continuous connection should normally be created when the feature enters the state that requires continuous work and disconnected as soon as that state ends.

This applies to:

- `RunService.Heartbeat`
- drag or pointer movement tracking
- continuous input streams
- animation progression
- simulation stepping
- active interpolation
- continuously sampled controls
- queue processors that genuinely need repeated execution
- any other callback source that can fire repeatedly without a new meaningful state transition

The lifetime of the module or UI is not automatically the lifetime of its continuous work.

**Preferred flow**

```text
feature becomes active
    ↓
connect continuous runtime signal
    ↓
perform continuous work
    ↓
feature becomes inactive
    ↓
disconnect continuous runtime signal
    ↓
remain dormant
```

> **Hard rule:** Continuous work follows the active state that requires it, not merely the existence of the owner.

---

#### 32.31 High-Frequency Input Listeners Should Be Temporary or Centralized

Signals that can fire for every pointer movement, analog change, drag update, touch movement, camera movement, or similar high-frequency input should not remain independently connected across many inactive controls.

For interaction-specific work, Lily should normally:

1. connect the high-frequency listener when the interaction begins
2. store the connection under the interaction owner
3. process only the active interaction
4. disconnect the listener immediately when the interaction ends, is cancelled, loses focus, becomes hidden, or is destroyed

A permanent global input listener is acceptable only when the system genuinely needs to observe global input while idle, such as a centralized keybind router. In that case, one controlled owner should route the input instead of every feature creating its own global listener.

> **Rule:** Prefer one intentional input owner over many dormant features waking for unrelated user input.

---

#### 32.32 Broad Watchers Must Be Narrowed to the Smallest Useful Scope

A broad event source can wake a system because unrelated parts of the experience changed. Lily should subscribe to the closest event source that fully represents the dependency.

Broad watchers that require extra scrutiny include:

- `game.DescendantAdded`
- `game.DescendantRemoving`
- `workspace.DescendantAdded`
- `workspace.DescendantRemoving`
- `ReplicatedStorage.DescendantAdded`
- `ReplicatedStorage.DescendantRemoving`
- broad `Changed` signals when one property signal is available
- global input streams when one control owns the interaction
- collection or registry signals covering substantially more state than the feature needs

**Prefer** a direct child, attribute, property, tag, remote, or owner-specific signal whenever it can express the same transition.

If a broad watcher is genuinely necessary during discovery or recovery, it should be disconnected as soon as the required dependency is resolved unless continued observation is part of the feature's actual responsibility.

> **Rule:** Do not make a small feature wake because the entire game changed somewhere unrelated.

---

#### 32.33 Deferred and Scheduled Work Must Not Create Idle Churn

`task.defer()` and `task.delay()` are valid for one-shot scheduling when the operation genuinely needs deferred execution. They must not be used to create hidden polling, self-rescheduling refresh loops, or repeated idle wakeups.

Lily should:

- coalesce duplicate queued work
- keep at most one pending task when one result is sufficient
- cancel owned delayed work when it becomes invalid
- invalidate stale generations before their callbacks mutate state
- avoid self-rescheduling callbacks unless the feature genuinely requires recurring work
- stop recurring scheduled work when the active state ends

**Avoid**

```lua
local function refreshLater()
	task.delay(.1, function()
		refreshFoo()
		refreshLater()
	end)
end
```

when an event can trigger `refreshFoo()` directly.

> **Hard rule:** A Lily task must not repeatedly wake an otherwise idle system just to check whether work exists.

---

#### 32.34 Unchanged State Produces Zero Downstream Work

State comparison should happen before expensive or externally visible work begins.

When the resolved value has not changed, Lily should avoid unnecessary:

- property writes
- attribute writes
- UI refreshes
- layout rebuilds
- callbacks
- signal emission
- remote sends
- serialization
- cache invalidation
- table reconstruction
- derived-value recalculation
- runtime object updates

**Preferred**

```lua
local function setFooValue(fooContext, value)
	if fooContext.value == value then return false end

	fooContext.value = value
	updateFoo(fooContext)
	return true
end
```

If several derived values are involved, compare the resolved state or use a controlled dirty-state model before triggering downstream work.

> **Hard rule:** No state change means no replication, no callback, no property update, and no runtime refresh unless an external contract explicitly requires one.

---

#### 32.35 Separate Lifetime Connections From Active-Work Connections

Long-lived systems often need a small set of passive lifecycle listeners while only temporarily needing high-frequency runtime listeners. These responsibilities should not be mixed into one undifferentiated connection collection when separating them makes the active lifecycle clearer.

A useful ownership shape is:

```lua
local fooContext = {
	activeConnections = {},
	connections = {},
}
```

`connections` may own low-frequency lifecycle or state-change signals that remain valid for the owner lifetime. `activeConnections` owns listeners that exist only while a particular interaction or runtime mode is active.

When the active state ends:

```lua
disconnectConnections(fooContext.activeConnections)
```

When the owner is destroyed, both groups are disconnected.

> **Rule:** The active runtime lifecycle may be shorter than the owner lifecycle, and Lily should model that difference explicitly when it prevents idle work.

---

#### 32.36 Idle UI Must Not Keep Interaction Runtime Active

A UI object may remain allocated without needing its interaction runtime to remain active.

When an interface, modal, page, editor, tool, or control becomes hidden or inactive, Lily should stop any interaction-specific work that can still fire in the background. Depending on the feature, this may include:

- drag tracking
- pointer movement tracking
- temporary keyboard capture
- active fader or slider tracking
- continuous preview updates
- animation stepping
- selection marquee updates
- hover work that is irrelevant while hidden
- delayed interaction callbacks

Low-frequency lifecycle listeners may remain when they are needed to detect the next legitimate state transition.

> **Rule:** Hidden or inactive UI should not retain active interaction runtime merely because its Instances still exist.

---

#### 32.37 Idle Runtime Must Be Verified During Review

A performance review should test what a system does when nobody is using it, not only what it does under load.

For a feature that should be dormant, verify that leaving it untouched does not produce a continuing stream of:

- script callback executions
- frame-rate work
- remote traffic
- property or attribute writes
- UI rebuilds
- hierarchy scans
- queue processing
- temporary allocations
- growing tables
- repeated setup
- duplicate connections

Useful idle tests include:

- leave the feature untouched
- move the pointer somewhere unrelated
- generate unrelated keyboard or game input
- change unrelated Instances elsewhere in the hierarchy
- open and close the feature repeatedly
- run setup more than once
- destroy and recreate the owner
- leave the system inactive after prior use

An idle system may still wake for a legitimate relevant event. The requirement is that it must not create a **self-sustaining or unrelated recurring execution rate** when no work is required.

> **Final performance rule:** Lily code should be event-driven by default, dormant while idle, active only for the lifetime of necessary work, and should use only the CPU, memory, networking, and runtime work that the feature actually requires. Every active resource must have an intentional owner, activation condition, deactivation condition, and **cleanup path**.

### 33. Luau Local and Register Limits

Large Luau modules can reach the local/register limit when too many top-level locals and local functions are declared in a single chunk, so very large files should be organized with that compiler limit in mind before it becomes a production issue.

A common error looks similar to:

```text
Out of local registers
exceeded limit 200
```

For large modules, cold private helpers can be grouped under an internal table when doing so retains the file below Luau's limit.

**Example**

```lua
local internals = {}

function internals.bar()
end

function internals.foo()
end
```

Do not automatically move every helper into a table. Hot helpers may still be better as locals when performance or clarity benefits from it.

> **Rule:** Stay safely below Luau's local/register limit without creating unnecessary architecture.

---

## Reference Examples

The final sections bring the convention together with common anti-patterns and complete examples that show how the rules work when combined.

### 34. Common Lily Anti-Patterns

The following patterns should normally be removed during review because they weaken readability, ownership, predictability, or maintainability and often make future changes more likely to introduce bugs.

---

#### conditional nesting

```lua
if fooContext then
	if foo then
		updateFoo(foo)
	end
end
```

Use guards instead.

---

#### `else` and `elseif` chains

Long branching structures should normally become early returns, separate operations, or lookup tables.

---

#### Chained boolean control flow

```lua
local value = condition and foo or bar
```

Use explicit control flow.

---

#### Polling for normal changes

```lua
while true do
	if value ~= previousValue then
		updateFoo(value)
	end

	task.wait()
end
```

Use the change source instead.

---

#### Idle callbacks kept alive behind guards

```lua
runService.Heartbeat:Connect(function(deltaTime)
	if not isActive then return end
	updateFoo(deltaTime)
end)
```

If the callback is unnecessary while inactive, disconnect it instead of letting the signal wake Lua continuously.

The same rule applies to high-frequency input, pointer movement, broad watchers, scheduled refresh callbacks, and other recurring sources.

---

#### Broad listeners for narrow responsibilities

Do not subscribe a small feature to global input or hierarchy activity when a narrower owner-specific signal can represent the same change.

---

#### Hidden UI with active interaction runtime

Do not keep drag, pointer, preview, animation, or other interaction-specific runtime active after the relevant interface becomes hidden or inactive.

---

#### `pairs()` and `ipairs()`

```lua
for key, value in pairs(data) do
end
```

Use generalized Luau iteration.

---

#### Redundant guard checks

Do not repeat validations that the architecture already guarantees.

---

#### Meaningless abbreviations

```lua
local ctx
local mgr
local val
```

Use descriptive names.

---

#### Hidden boolean arguments

```lua
updateFoo(foo, true, false, true)
```

Use named options when several behaviors need to be selected.

---

#### Giant scripts

Do not place an entire feature inside one `Script` or `LocalScript` when the behavior belongs in focused **ModuleScripts**.

---

#### Outside organization packages

Do not add third-party package organizations as standard Lily dependencies. Shared code should be Lily-owned.

---

#### Manually built Lily UI

Do not depend on a hidden Studio UI hierarchy when Lily can create the interface from code.

---

#### ValueObjects used only as metadata

Use **Attributes** for lightweight Instance-owned data.

---

#### Stale state

Do not leave old connections, contexts, UI, tasks, caches, or runtime objects alive after their owner is destroyed.

---

#### Unnecessary `any`

Do not hide type errors with `any` when the real type can be represented or validated.

---

#### Unsafe casts

Do not force a value into a type that has not actually been established.

---

### 35. Example Lily Function

```lua

local function updateFooState(fooContext: FooContext, fooName: string, enabled: boolean)
	fooContext.activeFoos[fooName] = nil
	if enabled then fooContext.activeFoos[fooName] = true end

	for _, bar in fooContext.bars do
		local baz = bar.bazMap[fooName]
		if not baz then continue end
		if baz.enabled == enabled then continue end

		baz.enabled = enabled
		updateBaz(baz, enabled)
	end
end
```

This follows the Lily convention because the function has one defined responsibility, uses descriptive names, retains control flow flat, avoids `else` and `elseif`, uses generalized iteration, uses `continue`, and exposes clear types.

---

### 36. Example Lily Module

```lua

-- handles foo runtime

local replicatedStorage = game:GetService("ReplicatedStorage")
local userInputService = game:GetService("UserInputService")

--————————————————————————————————————————————————————————————————————--

local module = {}

--————————————————————————————————————————————————————————————————————--

type fooContext = {
	fooKey: string,
	remote: RemoteEvent,
}

--————————————————————————————————————————————————————————————————————--

local connections: { RBXScriptConnection } = {}
local fooContexts: { [string]: FooContext } = {}

--————————————————————————————————————————————————————————————————————--

local function connect(signal: RBXScriptSignal, callback: () -> ()): RBXScriptConnection
	local connection = signal:Connect(callback)
	connections[#connections + 1] = connection

	return connection
end

--————————————————————————————————————————————————————————————————————--

local function getFooContext(fooKey: string): FooContext?
	local fooContext = fooContexts[fooKey]
	if not fooContext then return end
	if not fooContext.remote.Parent then return end

	return fooContext
end

--————————————————————————————————————————————————————————————————————--

local function disconnect()
	for _, connection in connections do
		connection:Disconnect()
	end

	table.clear(connections)
end

--————————————————————————————————————————————————————————————————————--

function module.start(fooKey: string): boolean
	if fooKey == "" then return false end
	if getFooContext(fooKey) then return true end

	return true
end

--————————————————————————————————————————————————————————————————————--

function module.destroy(fooKey: string)
	if not fooContexts[fooKey] then return end

	fooContexts[fooKey] = nil
	if next(fooContexts) then return end

	disconnect()
end

--————————————————————————————————————————————————————————————————————--

return module
```

The important part is not the exact names in the example. The important part is the structure: strict typing, alphabetical top sections, flat control flow, focused helpers, clear **state ownership**, generalized iteration, predictable **lifecycle**, and a small public module API.

This convention is a general coding standard, not an absolute rule for every situation. If a rule cannot reasonably be followed because of Roblox limitations, legacy code, compatibility, performance, or correctness, it may be adapted for that specific case.

Any exception should still keep the code as clear, predictable, maintainable, and safe as possible.
---

> ## Lily Studio standard
>
> **Write code that another Lily developer can understand quickly, trust immediately, and maintain safely.**
````
