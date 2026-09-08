# Lily Studio Coding Convention

> **Official coding standard for Lily Studio Roblox and Luau development.**

The Lily Studio coding convention exists so every Lily project follows the same structure, naming style, control-flow rules, type-safety expectations, performance habits, and architectural patterns. The goal is not to make every file look identical, but to make every file feel familiar enough that another Lily Studio developer can open it, understand it quickly, and continue working without having to learn a different style each time.

Lily code should always favor **clarity, consistency, predictable behavior, simple control flow, strong typing, clean ownership, and easy cleanup**. Code should be written for the next developer who needs to read it, debug it, extend it, or safely replace part of it later.

> ### Core principle
>
> **Clear first. Compact second. Clever never.**

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
  - [17. Guard Clauses](#17-guard-clauses)
  - [18. No Conditional Nesting](#18-no-conditional-nesting)
  - [19. Avoid `else` and `elseif`](#19-avoid-else-and-elseif)
  - [20. Positive Conditions](#20-positive-conditions)
  - [21. Avoid Boolean Expression Control Flow](#21-avoid-boolean-expression-control-flow)
  - [22. Iteration](#22-iteration)
- **Types, Documentation, and Failures**
  - [23. Type Checking](#23-type-checking)
  - [24. Comments and Documentation](#24-comments-and-documentation)
  - [25. Error Handling](#25-error-handling)
- **File Layout and Source Organization**
  - [26. File Organization](#26-file-organization)
  - [27. Alphabetical Top-Level Order](#27-alphabetical-top-level-order)
  - [28. Lily Separators](#28-lily-separators)
  - [29. Formatting](#29-formatting)
- **User Interface**
  - [30. Script-Created UI Only](#30-script-created-ui-only)
- **Performance and Scale**
  - [31. Performance](#31-performance)
  - [32. Luau Local and Register Limits](#32-luau-local-and-register-limits)
- **Reference Examples**
  - [33. Common Lily Anti-Patterns](#33-common-lily-anti-patterns)
  - [34. Example Lily Function](#34-example-lily-function)
  - [35. Example Lily Module](#35-example-lily-module)
  - [36. Final Standard](#36-final-standard)

---

## Foundation

These sections define the basic expectations that apply to every Lily Studio file. They establish what Lily code is trying to achieve before architecture or implementation details are considered.

### 1. Purpose

This document defines the normal coding style for Lily Studio projects. It should be used when creating new systems, reviewing pull requests, refactoring old code, building shared packages, writing UI, creating runtime systems, and maintaining project infrastructure.

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

Lily code should be easy to follow from top to bottom, with each section having a clear purpose and each function doing one clear job. A developer should not need to mentally untangle deeply nested conditions, hidden state changes, or clever expression tricks just to understand what a function does.

Lily code should normally be:

- **clear**, because the meaning should be obvious without extra explanation
- **compact**, because unnecessary lines and repeated logic make files harder to maintain
- **type-safe**, because mistakes should be caught before runtime whenever possible
- **event-driven**, because code should react when something changes instead of repeatedly checking for change
- **modular**, because features should be separated into focused reusable modules
- **predictable**, because similar systems should use similar patterns
- **cleanable**, because every connection, object, task, and runtime state should have a clear lifecycle
- **performant**, because repeated small costs can become large in a large Roblox project
- **consistent**, because the same idea should be written the same way throughout Lily Studio

Lily code should not try to impress the reader with unusual patterns. The best Lily code should feel simple, direct, and easy to trust.

---

### 3. Preserve Existing Behavior

When existing Lily code is cleaned, reorganized, optimized, or typed, its behavior should remain the same unless the task explicitly includes a behavior change. Refactoring should improve how the code is written without silently changing what the code does.

Do not silently change:

- remote names
- payload fields
- public method names
- return values
- default values
- timing behavior
- state behavior
- cleanup behavior
- input behavior
- networking behavior
- module APIs
- object ownership
- initialization order

##### Example

If existing code sends:

```lua
remote:FireServer({
	value = value,
	enabled = enabled,
})
```

a style cleanup should not silently change it to:

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

Lily systems should behave in a controlled and predictable way. A developer should be able to understand where a value comes from, what can change it, what happens after it changes, and how the system returns to a clean state.

The code should not depend on accidental timing, hidden state, unexplained fallbacks, or behavior that only works because several unrelated parts happen to run in a certain order.

> **Main rule:** Lily should know what will happen, why it will happen, and which part of the code is responsible for making it happen.

#### Explicit state changes

State should change through clear functions or owners instead of being modified from unrelated places.

##### Preferred

```lua
local function setFooValue(fooContext: FooContext, value: number)
	if fooContext.value == value then return end

	fooContext.value = value
	updateFoo(fooContext)
end
```

The function shows exactly when the value changes and what happens afterward.

##### Avoid

```lua
fooContext.value = value
```

when many unrelated files can change the same value without going through a shared owner.

#### Explicit defaults

Defaults should be defined clearly instead of depending on missing data or accidental engine behavior.

##### Preferred

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

#### Avoid hidden side effects

A function should not silently change unrelated state.

The function name, API, and structure should make important side effects easy to discover.

If `setFooValue()` also destroys objects, sends network data, starts tasks, and changes unrelated configuration, the function is doing too much and should be separated.

#### Avoid accidental timing dependencies

Do not design normal behavior around assumptions such as:

- another script will probably run first
- a value will probably exist by the next frame
- a connection will probably be ready in time
- two independent callbacks will probably execute in the expected order

If an order or dependency matters, Lily should control it directly through initialization, modules, signals, or explicit state.

#### Random behavior must still be controlled

Randomness is allowed only when the feature intentionally requires it. The system should still control when randomness is used, what range is allowed, and how the result affects state.

Do not use randomness as a replacement for missing logic or uncertain behavior.

#### No unknown ownership

Every important value should have a clear owner.

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

Once the basic expectations are clear, the next concern is structure. Lily systems should be made from focused modules with clear dependencies, controlled ownership, and reusable code that remains inside the Lily ecosystem.

### 5. ModuleScript-First Architecture

Lily Studio uses **ModuleScripts for most code** and keeps normal `Script` and `LocalScript` files to a minimum. Most features should live in modules because modules are easier to reuse, test, organize, type, and compose.

> **Main rule:** Lily systems live in modules. Scripts are small entry points.

---

#### 5.1 Most Logic Belongs in ModuleScripts

Feature logic, controllers, state owners, utilities, runtime systems, UI builders, data handling, and reusable behavior should normally be placed in ModuleScripts.

##### Preferred structure

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

A normal `Script` or `LocalScript` should usually perform a small amount of startup work and then hand control to modules.

##### Preferred

```lua
local foo = require(source:WaitForChild("foo"))

foo.start()
```

The entry script should not contain hundreds of lines of business logic when that logic can live in modules.

---

#### 5.3 Use Few Entry Scripts

Lily projects should avoid scattering many independent scripts throughout the hierarchy because each additional script creates another startup path, another lifecycle to understand, and another place where hidden behavior can begin.

A smaller number of intentional entry scripts makes execution order and ownership easier to understand.

Good uses for scripts include:

- server bootstrap
- client bootstrap
- environment-specific startup
- top-level initialization that cannot be represented as a reusable module

Everything below that level should normally be delegated to ModuleScripts.

---

#### 5.4 Modules Should Have Clear APIs

A module should expose only the functions or values that outside code actually needs.

##### Preferred

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

### 6. Lily Packages Only

Lily Studio code should use **Lily-owned packages and modules** instead of packages maintained by outside organizations. Shared behavior that Lily depends on should live under Lily's own package structure so the codebase remains controlled, consistent, reviewable, and maintainable by Lily Studio.

> **Main rule:** Lily code uses Lily packages. Do not add outside organization packages as project dependencies.

---

#### 6.1 Prefer Lily-Owned Dependencies

When reusable functionality is needed, first use an existing Lily package or create a Lily-owned package for that responsibility.

##### Preferred

```lua
local foo = require(lilyPackages:WaitForChild("foo"))
```

The package should be maintained as part of the Lily codebase and follow the same coding convention described in this document.

---

#### 6.2 Do Not Add Third-Party Organization Packages

Do not add packages owned by unrelated organizations only because they already solve a small problem. External packages introduce another API style, another update schedule, another ownership boundary, and another source of changes that Lily Studio does not control.

Lily should not depend on outside package organizations for normal project architecture.

---

#### 6.3 Roblox APIs Are Still Normal Dependencies

This rule does not apply to Roblox's built-in APIs, services, types, enums, or engine functionality. Roblox itself is the platform and does not count as an outside organization package under this convention.

---

#### 6.4 Build Shared Lily Packages When Reuse Is Real

If the same behavior is needed across several Lily systems, move the behavior into a focused Lily package instead of copying it between projects or importing an outside package.

A Lily package should:

- have one clear responsibility
- expose a small clear API
- use `--!strict` when practical
- follow Lily naming and formatting
- have a clear lifecycle if it creates runtime state
- avoid hidden global state
- remain easy to replace or update

> **Rule:** Reusable shared code should become a Lily package, not an uncontrolled outside dependency.

---

### 7. Composition Over Inheritance

Lily prefers small focused systems working together instead of deep inheritance trees.

##### Generic example

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

Repeated behavior should be generalized when several systems truly perform the same operation.

##### Good example

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

---

## State, Change, and Lifecycle

After the architecture is defined, state should have one owner, changes should happen through clear paths, and everything created by the system should have a matching cleanup path.

### 9. State Ownership

A predictable system begins with clear ownership. State should have one clear owner, and the code that owns that state should normally control the approved ways that state changes.

##### Preferred

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
- another clear owner

---

#### 9.2 Keep Related State Together

Values that describe the same feature should normally be stored together instead of being spread across unrelated top-level tables.

> **Rule:** The code that owns a value should also control the normal ways that value changes.

---

### 10. Public State

Public state should be controlled when direct mutation could break an invariant or bypass required behavior.

##### Example

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

Lily prefers **Instance Attributes** for simple values that belong directly to an Instance.

##### Preferred

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

Attributes are the normal choice for simple values such as:

- booleans
- numbers
- strings
- identifiers
- modes
- small configuration values
- lightweight metadata
- simple state that belongs directly to an Instance

##### Example

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

> **Rule:** Use Attributes for simple Instance metadata and Luau state for complex runtime data.

---

### 12. Event-Driven Code

Lily should react when state changes instead of continuously checking whether state changed. Roblox already provides signals for many common changes, and Lily should use those signals instead of polling whenever possible.

> **Main rule:** Lily reacts to changes. Lily does not repeatedly ask whether something changed.

---

#### 12.1 Do Not Poll for Normal State Changes

##### Avoid

```lua
while true do
	if object.Enabled ~= previousValue then
		previousValue = object.Enabled
		updateFoo(object.Enabled)
	end

	task.wait()
end
```

##### Preferred

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

#### 12.3 Do Not Use Frame Events as General Change Detectors

`Heartbeat`, `RenderStepped`, and `Stepped` should not be used only to check whether ordinary data changed.

##### Avoid

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

#### 12.4 Frame Updates Are Still Valid for Real Frame-Based Work

A frame event is correct when the work genuinely must happen continuously, such as animation, simulation, interpolation, real-time calculation, or other frame-dependent behavior.

The important distinction is simple:

> **Frame loops are for continuous runtime work, not for watching ordinary state.**

---

### 13. Cleanup and Lifecycle

Everything Lily creates should have a clear cleanup path. If a system creates a connection, runtime object, UI object, task, cache entry, or context, it should also know how that object is removed.

This includes:

- `RBXScriptConnection`
- created Instances
- UI objects
- runtime contexts
- cached tables
- temporary objects
- scheduled tasks
- references that could prevent garbage collection

##### Preferred

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

---

## Functions and Control Flow

With ownership established, the implementation should remain easy to follow at the function level. Lily favors descriptive names, one clear responsibility, flat control flow, and direct iteration.

### 14. Naming

Good names make Lily code easier to understand without requiring extra comments. A developer should usually be able to understand the purpose of a value or function by reading its name at the call site.

---

#### 14.1 Use `camelCase`

Normal Lily variables, functions, fields, and module methods use `camelCase`.

##### Preferred

```lua
local fooValue
local activeFoo
local selectedBars
local fooController

local function updateFoo()
end

function module.setFooValue()
end
```

##### Avoid

```lua
local FooValue
local selected_bars
local FOO_VALUE
```

---

#### 14.2 Use Full and Descriptive Words

Names should be long enough to clearly explain their meaning, and unnecessary abbreviations should be avoided because they make larger files harder to scan.

##### Preferred

```lua
local connection
local controller
local selectedBars
local currentValue
```

##### Avoid

```lua
local conn
local ctrl
local sel
local val
```

A short name is acceptable only when it is already a widely understood technical term and the meaning is obvious in context.

---

#### 14.3 Boolean Names Should Read Like Questions

Boolean values should sound like something that can naturally be answered with yes or no.

##### Preferred

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
```

---

#### 14.4 Function Names Should Describe Actions

Functions should normally begin with a verb so the call site clearly describes what is happening.

##### Preferred

```lua
updateFoo()
createBar()
destroyFoo()
sendBaz()
resolveValue()
releaseFoo()
```

##### Avoid

```lua
foo()
barThing()
valueData()
```

---

#### 14.5 Event Handlers Should Describe When They Run

Event callbacks should use names that explain what event causes them to execute.

##### Preferred

```lua
onInputBegan()
onInputEnded()
onClick()
onRemoteEvent()
onAttributeChanged()
```

This style makes event wiring easier to read and keeps handler names consistent across Lily projects.

---

### 15. Functions

Once names are clear, functions should stay focused enough that their purpose can be understood without tracing several unrelated operations. Each Lily function should have one clear responsibility and should be easy to describe in one sentence.

##### Preferred separation

```lua
local function createFoo()
end

local function updateFoo()
end

local function destroyFoo()
end
```

A single function should not simultaneously handle validation, UI creation, networking, state mutation, cleanup, and unrelated runtime behavior unless those actions genuinely form one small operation.

---

#### 15.1 Create Helpers Only When They Add Meaning

A helper should reduce repeated logic, remove nesting, isolate a responsibility, or make the main flow easier to read.

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

This helper represents a real lifecycle operation and can be reused safely.

---

### 16. Function Arguments

Function calls should explain themselves without requiring the reader to open the function definition.

---

#### 16.1 Avoid Hidden Boolean Arguments

##### Avoid

```lua
updateFoo(foo, true, false)
```

The reader cannot easily tell what each boolean means.

##### Preferred

```lua
updateFoo(foo, {
	force = false,
	replicate = true,
})
```

Use an options table when several optional behaviors need names.

---

#### 16.2 Avoid Positional `nil`

##### Avoid

```lua
createFoo(name, nil, true)
```

##### Preferred

```lua
createFoo(name, {
	enabled = true,
})
```

Options tables should still be used only when they make the call clearer. Simple functions should remain simple.

---

### 17. Guard Clauses

Guard clauses are a normal part of Lily code because they keep functions flat and make invalid states easy to see. A guard clause should exit early when the function cannot safely continue.

##### Preferred

```lua
local function updateFoo(fooContext)
	if not fooContext then return end
	if fooContext.destroyed then return end
	if not fooContext.foo then return end

	fooContext.foo.Enabled = true
end
```

##### Avoid

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

#### 17.1 Do Not Add Redundant Guards

Guard clauses should protect real conditions, not repeat guarantees that were already established by the surrounding architecture.

##### Avoid

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

> **Rule:** Lily uses necessary guard clauses, but removes checks that are already guaranteed elsewhere.

---

### 18. No Conditional Nesting

Lily avoids conditional nesting because nested conditions make control flow harder to follow and create the familiar rightward arrow shape that becomes difficult to maintain.

##### Avoid

```lua
if fooContext then
	if foo then
		updateFoo(foo)
	end
end
```

##### Preferred

```lua
if not fooContext then return end
if not foo then return end

updateFoo(foo)
```

The same rule applies inside loops, callbacks, event handlers, and public module methods.

---

#### 18.1 Use Helpers When Flat Code Becomes Too Large

If a function cannot stay flat without becoming difficult to read, move a meaningful decision into a small helper rather than adding nested branches.

##### Example

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

The helper should still have one clear responsibility and should not exist only to hide complexity.

> **Rule:** Lily uses guard clauses, `continue`, `break`, lookup tables, and small helpers instead of nested conditionals.

---

### 19. Avoid `else` and `elseif`

Lily prefers control flow that moves downward in a straight line. `else` and `elseif` are avoided because they often make functions harder to scan and usually indicate that a guard clause, early return, or lookup table would be clearer.

---

#### 19.1 Avoid `else`

##### Avoid

```lua
if enabled then
	startFoo()
else
	stopFoo()
end
```

##### Preferred

```lua
if enabled then
	startFoo()
	return
end

stopFoo()
```

---

#### 19.2 Avoid `elseif`

##### Avoid

```lua
if mode == "Foo" then
	runFoo()
elseif mode == "Bar" then
	runBar()
elseif mode == "Baz" then
	runBaz()
end
```

##### Preferred

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

### 20. Positive Conditions

Conditions and boolean names should normally use positive wording because positive logic is easier to read quickly.

##### Preferred

```lua
if isRunning then
	updateFoo()
end
```

##### Avoid

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

### 21. Avoid Boolean Expression Control Flow

Lily does not use chained `and/or` expressions as a replacement for normal control flow.

##### Avoid

```lua
local value = condition and foo or bar
```

Especially avoid long chains that choose between several values:

```lua
local result = firstCondition and foo
	or secondCondition and bar
	or baz
```

##### Preferred

```lua
if firstCondition then return foo end
if secondCondition then return bar end

return baz
```

The explicit version is easier to debug and behaves correctly when `false` or `nil` are valid values.

---

### 22. Iteration

Lily allows finite collection iteration when several values actually need to be processed, but does not use polling loops to wait for change.

---

#### 22.1 Use Generalized Luau Iteration

Lily does not use `pairs()` or `ipairs()`.

##### Preferred

```lua
for key, value in data do
end
```

```lua
for index, bar in bars do
end
```

##### Avoid

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

#### 22.2 Use `continue` to Keep Loop Bodies Flat

##### Preferred

```lua
for _, bar in bars do
	if not bar.Parent then continue end
	if bar:GetAttribute("disabled") then continue end

	updateBar(bar)
end
```

---

#### 22.3 Use `break` When the Work Is Complete

##### Preferred

```lua
for _, bar in bars do
	if bar.Name ~= fooName then continue end

	foundBar = bar
	break
end
```

A loop should not continue doing work after the required result has already been found.

---

---

## Types, Documentation, and Failures

Types and documentation should make an API easier to understand before it is used, while failure behavior should remain explicit and consistent with the rest of the code.

### 23. Type Checking

Clear ownership and APIs become easier to maintain when the types are equally clear. Type checking is part of the Lily Studio standard because it improves autocomplete, documents expectations, and catches mistakes before runtime.

---

#### 23.1 Use `--!strict`

Production Lily modules should normally begin with:

```lua
--!strict
```

Strict mode helps catch invalid property access, incorrect arguments, missing fields, accidental `nil`, incorrect return values, and incorrect module usage before the code reaches runtime.

---

#### 23.2 Type Function Parameters and Returns

##### Preferred

```lua
local function getFoo(fooKey: string): FooContext?
	return fooContexts[fooKey]
end
```

```lua
function module.start(fooKey: string): boolean
	return true
end
```

Important public APIs should not rely on the reader guessing what type is expected.

---

#### 23.3 Use Named Types for Repeated Structures

##### Preferred

```lua
export type FooData = {
	enabled: boolean,
	name: string,
	value: number,
}
```

```lua
type FooContext = {
	connections: { RBXScriptConnection },
	destroyed: boolean,
	value: number,
}
```

Named types improve autocomplete and make large functions easier to read.

---

#### 23.4 Export Types Only When Other Modules Need Them

Use `export type` for types that are part of a module's public API. Internal implementation types should remain local.

---

#### 23.5 Type Collections

##### Preferred

```lua
local bars: { string } = {}
local connections: { RBXScriptConnection } = {}
local fooContexts: { [string]: FooContext } = {}
```

Typed collections prevent accidental insertion of incompatible values.

---

#### 23.6 Use Optional Types Intentionally

```lua
local activeFooKey: string?
```

```lua
local function getFoo(fooKey: string): FooContext?
end
```

Do not make every value optional only to make the type checker stop reporting errors.

---

#### 23.7 Narrow Types With Guard Clauses

##### Preferred

```lua
local function useFoo(object: Instance?)
	if not object then return end
	if not object:IsA("RemoteEvent") then return end

	object:FireServer()
end
```

Guard clauses work naturally with Luau type narrowing and also match Lily's flat control-flow style.

---

#### 23.8 Avoid `any`

Do not use `any` only to hide a type error.

##### Avoid

```lua
local value: any = data.value
```

Prefer a known type, or validate an unknown value before use.

---

#### 23.9 Avoid Unsafe Casts

##### Avoid

```lua
local foo = object :: RemoteEvent
```

when the type has not been established.

##### Preferred

```lua
if not object:IsA("RemoteEvent") then return end

local foo = object
```

Casts should be used only when the architecture genuinely guarantees the type and Luau cannot infer it.

---

#### 23.10 Runtime Validation Still Matters

Static typing cannot guarantee the shape of data that arrives from runtime boundaries such as RemoteEvents, attributes, user input, JSON, or dynamically discovered Instances.

Lily should still validate external data before trusting it.

> **Rule:** Static types protect the codebase. Runtime validation protects runtime boundaries.

---

### 24. Comments and Documentation

Types explain the shape of an API, while comments should explain the parts that types and names cannot communicate on their own. Comments in Lily code must have a real reason to exist and should help another developer understand behavior, intent, limits, or important side effects.

Comments should explain **why**, important behavior, unusual decisions, expectations, side effects, or API usage. They should not repeat simple code in plain English.

> **Main rule:** A Lily comment should make the code easier to understand, not add noise around code that was already clear.

---

#### 24.1 Comments Must Be Proper and Useful

A comment should be complete enough to be understood by another developer who did not write the original code.

##### Avoid

```lua
-- update foo
updateFoo()
```

The function name already explains that the function updates `foo`, so the comment adds no useful information.

##### Better

```lua
-- Update the cached value before notifying listeners so every listener reads the new state.
updateFoo()
```

The second comment explains an important reason for the order of operations.

---

#### 24.2 Explain Why, Not What

Comments should usually explain why the code is written a certain way.

##### Avoid

```lua
-- Set foo to true.
foo = true
```

##### Better

```lua
-- Keep this enabled until cleanup so callbacks cannot recreate the object during destruction.
foo = true
```

The code already shows **what** happens. The comment explains **why** it happens.

---

#### 24.3 Comments Should Explain Non-Obvious Behavior

A comment is useful when the behavior would otherwise be surprising.

Good reasons for a comment include:

- a specific execution order is required
- a workaround exists because of a Roblox limitation
- a performance decision is not obvious
- cleanup must happen before another operation
- a value intentionally does not update immediately
- an API has an unusual requirement
- a function has an important side effect
- a temporary compatibility rule exists
- a section depends on behavior outside the current file

##### Example

```lua
-- Disconnect first so the callback cannot run again while the owner is being destroyed.
disconnectConnections(fooContext)
```

---

#### 24.4 Comments Must Stay Accurate

Incorrect comments are worse than missing comments because they give developers false information.

When behavior changes, any comment describing that behavior must be updated at the same time.

Do not leave comments that describe:

- old parameter names
- removed behavior
- old state ownership
- outdated workarounds
- old APIs
- behavior that no longer exists

> **Rule:** If the code changes, the related documentation must change with it.

---

#### 24.5 Use Documentation Comments for Important Functions

Important public functions, reusable helpers, shared package APIs, and functions with non-obvious parameters should use clear documentation comments when documentation improves the API.

Luau documentation comments can use `---` and tags such as `@param`, `@return`, and `@error` when those tags make the function easier to understand.

##### Example

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

#### 24.6 `@param` Should Add Meaning

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

#### 24.7 Useful Documentation Tags

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

#### 24.8 Public APIs Need Better Documentation Than Private Details

Shared Lily modules and Lily packages are used by other developers, so their public APIs should be especially clear.

A public function should make it easy to understand:

- what the function does
- what each important parameter means
- what it returns
- what state it changes
- whether it creates anything
- whether cleanup is required
- whether it can fail
- whether it has important side effects

Private helpers usually need less documentation when their name, types, and implementation already make the behavior obvious.

---

#### 24.9 Comments Are Not a Replacement for Clear Code

Do not keep confusing code and then try to explain the confusion with a large comment.

##### Avoid

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
- simpler state ownership
- fewer responsibilities
- flat control flow
- focused helpers

Comments should support good code, not compensate for bad code.

---

#### 24.10 Keep Comments Professional

Comments are part of the Lily Studio codebase and should be written professionally.

Avoid:

- jokes that make behavior unclear
- temporary frustration comments
- insults
- vague notes such as `fix later`
- unexplained abbreviations
- comments written only for the original author
- comments that do not use normal grammar when a full explanation is needed

##### Avoid

```lua
-- idk why this breaks lol
```

##### Better

```lua
-- Roblox may return nil while the object is being reparented, so defer the lookup until the next task cycle.
```

---

#### 24.11 Use `TODO` Only When the Work Is Real and Specific

A `TODO` should explain exactly what remains to be done and why it is not being completed in the current change.

##### Avoid

```lua
-- TODO fix this
```

##### Better

```lua
-- TODO: Replace this compatibility path after the old data format is no longer supported.
```

Do not use `TODO` as permanent documentation for known broken behavior.

---

# 24.12 Code Must Be Easy to Explain

A Lily developer should be able to explain a function in simple words without needing a long technical speech.

If a function cannot be explained clearly, the function is probably doing too much, hiding too much state, or using control flow that is too complicated.

> **Main rule:** If you cannot easily explain what a function does and how it works, the function should be improved.

A good explanation should normally sound simple:

> "This function gets the current value, returns early when nothing changed, stores the new value, and notifies the listeners."

That explanation is short because the function has a clear responsibility.

A warning sign sounds more like:

> "This function sometimes changes the value, but it also creates objects, checks another system, sends data, updates several tables, and then does different cleanup depending on how it was called."

That function should probably be split into smaller operations.

---

#### 24.13 Functions Should Be Explainable From Their Structure

A clear function should make its behavior visible through:

- a descriptive name
- clear parameter names
- strong types
- necessary guard clauses
- no conditional nesting
- one main responsibility
- obvious state changes
- clear return behavior
- small focused helpers when needed

##### Preferred

```lua
local function setFooValue(fooContext: FooContext, value: number)
	if fooContext.value == value then return end

	fooContext.value = value
	updateFooView(fooContext)
	notifyFoo(fooContext)
end
```

This function is easy to explain:

> It ignores duplicate values, stores the new value, updates the view, and notifies listeners.

---

#### 24.14 Difficulty Explaining Code Is a Design Warning

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

The goal is not to make every function tiny. The goal is to make every function understandable.

---

#### 24.15 Documentation Should Match the Level of Complexity

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

This keeps comments useful without covering every line with unnecessary text.

---

### 25. Error Handling

Lily should use different failure behavior depending on whether the problem is expected, recoverable, or represents a broken programming assumption.

##### Recoverable failure

```lua
function module.start(fooKey: string): boolean
	if fooKey == "" then return false end

	return true
end
```

##### Broken invariant

```lua
assert(type(data) == "table", "Foo data must return a table.")
```

Use `warn()` when the system can continue but the problem is important enough to report.

Errors should not be used as normal control flow.

---

---

## File Layout and Source Organization

The source file itself should also be predictable. Related declarations should stay together, top-level groups should follow the same order, and formatting should make large files easy to scan.

### 26. File Organization

Lily files should follow a predictable top-level structure so developers can quickly find the section they need.

#### Recommended order

| Order | Section |
| ---: | --- |
| 1 | `--!strict` |
| 2 | File description |
| 3 | Roblox services |
| 4 | Module table |
| 5 | Constants and configuration |
| 6 | State |
| 7 | Dependencies |
| 8 | Types |
| 9 | Private helpers |
| 10 | Feature functions |
| 11 | Public module methods |
| 12 | `return module` |

##### Example

```lua
--!strict

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

type FooContext = {
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

### 27. Alphabetical Top-Level Order

Anything grouped near the top of a Lily file should be alphabetized **inside its own logical section**. This makes files easier to compare and reduces random ordering differences between developers.

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

#### 27.1 Services

##### Preferred

```lua
local collectionService = game:GetService("CollectionService")
local httpService = game:GetService("HttpService")
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local tweenService = game:GetService("TweenService")
local userInputService = game:GetService("UserInputService")
```

---

#### 27.2 Required Modules

##### Preferred

```lua
local bar = require(source:WaitForChild("bar"))
local baz = require(source:WaitForChild("baz"))
local foo = require(source:WaitForChild("foo"))
```

---

#### 27.3 References

##### Preferred

```lua
local barFolder = source:WaitForChild("bar")
local bazFolder = source:WaitForChild("baz")
local fooFolder = source:WaitForChild("foo")
```

---

#### 27.4 Dependency Order Can Override Alphabetical Order

##### Preferred when values depend on each other

```lua
local maximumFoo = 10
local minimumFoo = 1
local fooRange = maximumFoo - minimumFoo
```

`fooRange` depends on the two values above it, so dependency order is more important than forcing every name into alphabetical order.

---

### 28. Lily Separators

Use the standard Lily separator between major file sections and after top-level functions.

```lua
--————————————————————————————————————————————————————————————————————--
```

##### Example

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

### 29. Formatting

Lily formatting should be compact enough to avoid unnecessary vertical space, while still leaving enough structure for the code to be easy to scan.

---

#### 29.1 Keep Related Lines Together

##### Preferred

```lua
local function setFoo(fooContext, value)
	fooContext.value = value
	updateFoo(fooContext)
end
```

##### Avoid

```lua
local function setFoo(fooContext, value)

	fooContext.value = value


	updateFoo(fooContext)

end
```

---

#### 29.2 Keep Simple Expressions on One Line

##### Preferred

```lua
local x = math.clamp((foo - bar) / baz, 0, 1)
```

Avoid breaking a simple expression across several lines when the one-line version remains easy to understand.

---

#### 29.3 Keep Large Tables Easy to Scan

##### Preferred

```lua
local data = {
	enabled = true,
	name = "Foo",
	value = 1,
}
```

Lily code should be compact, but not compressed to the point where the structure becomes hard to see.

---

---

## User Interface

Lily interfaces follow the same ownership rules as the rest of the codebase: they are created by code, configured in source, updated from runtime state, and removed by the system that owns them.

### 30. Script-Created UI Only

Lily Studio UI is created through code. Lily should not depend on a manually assembled UI hierarchy in Roblox Studio for interfaces that belong to Lily systems.

> **Main rule:** Lily UI is created, configured, connected, updated, and cleaned up by scripts and modules.

---

#### 30.1 Do Not Depend on Manually Built Lily UI

Avoid requiring a prebuilt hierarchy such as:

```text
StarterGui
└── FooGui
    ├── Bar
    ├── Baz
    └── Foo
```

when the Lily runtime expects those UI objects to already exist.

---

#### 30.2 Create UI From Code

##### Preferred

```lua
local fooFrame = interface.Frame(parent, {
	Name = "FooFrame",
	Position = UDim2.fromOffset(10, 10),
	Size = UDim2.fromOffset(300, 200),
})
```

The code that creates the interface should also own the important configuration and lifecycle of that interface.

---

#### 30.3 UI Properties Belong in Code

Important properties should be visible in source control, including:

- position
- size
- anchor point
- colors
- transparency
- text
- fonts
- image IDs
- layouts
- scrolling behavior
- `ZIndex`
- input behavior

This makes Lily UI reproducible and easier to review through GitHub.

---

#### 30.4 UI Should Reflect Runtime State

The interface should display state, not become the main storage location for important state.

A normal flow should look like:

```text
state changes
    ↓
update function runs
    ↓
UI reflects the new state
```

---

#### 30.5 Script-Created UI Must Be Cleanable

If Lily creates a UI object, the owning system must also have a clear way to destroy it or replace it without creating duplicates.

---

---

## Performance and Scale

Performance comes after correctness and clarity. Lily optimizes repeated work, avoids unnecessary runtime cost, and respects Luau limits without making ordinary code harder to understand.

### 31. Performance

After behavior and ownership are correct, Lily should consider the cost of repeating that behavior at scale. A small cost that is harmless during setup can become expensive when it runs every frame or across many active objects and systems.

Performance work should focus first on repeated work rather than one-time setup work.

---

#### 31.1 Cache Repeated References

Avoid repeatedly searching the hierarchy in hot code when a stable reference can be resolved once and reused.

---

#### 31.2 Avoid Unnecessary Per-Frame Allocation

Be careful about creating new tables, closures, arrays, temporary objects, or other garbage inside code that runs every frame.

---

#### 31.3 Prevent Duplicate Connections

Setup should not repeatedly connect the same events without cleaning the previous connections first.

---

#### 31.4 Avoid Repeated Runtime `WaitForChild`

Required references should normally be resolved during setup rather than repeatedly searched inside runtime update paths.

---

#### 31.5 Optimize Real Hot Paths

The most important areas include:

- frame updates
- scheduler work
- repeated object updates
- high-frequency callbacks
- frequently called networking paths
- large collection processing

Lily should still favor readable solutions and should not make ordinary code difficult to understand for a meaningless micro-optimization.

> **Rule:** Optimize repeated work first, while keeping the code simple enough to maintain.

---

### 32. Luau Local and Register Limits

Large Luau modules can hit the local/register limit when too many top-level locals and local functions are declared in one chunk.

A common error looks similar to:

```text
Out of local registers
exceeded limit 200
```

For large modules, cold private helpers can be grouped under an internal table when doing so keeps the file below Luau's limit.

##### Example

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

---

## Reference Examples

The final sections bring the convention together with common anti-patterns and complete examples that show how the rules work when combined.

### 33. Common Lily Anti-Patterns

The following patterns should normally be removed during review because they make Lily code harder to understand, harder to maintain, or easier to break.

---

#### Conditional nesting

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

Do not place an entire feature inside one `Script` or `LocalScript` when the behavior belongs in focused ModuleScripts.

---

#### Outside organization packages

Do not add third-party package organizations as normal Lily dependencies. Shared code should be Lily-owned.

---

#### Manually built Lily UI

Do not depend on a hidden Studio UI hierarchy when Lily can create the interface from code.

---

#### ValueObjects used only as metadata

Use Attributes for simple Instance-owned data.

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

### 34. Example Lily Function

```lua
--!strict

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

This follows the Lily convention because the function has one responsibility, uses descriptive names, keeps control flow flat, avoids `else` and `elseif`, uses generalized iteration, uses `continue`, and exposes clear types.

---

### 35. Example Lily Module

```lua
--!strict

-- handles foo runtime

local replicatedStorage = game:GetService("ReplicatedStorage")
local userInputService = game:GetService("UserInputService")

--————————————————————————————————————————————————————————————————————--

local module = {}

--————————————————————————————————————————————————————————————————————--

type FooContext = {
	fooKey: string,
	remote: RemoteEvent,
}

--————————————————————————————————————————————————————————————————————--

local connections: { RBXScriptConnection } = {}
local fooContexts: { [string]: FooContext } = {}

--————————————————————————————————————————————————————————————————————--

local function connect(signal: RBXScriptSignal, callback: (...any) -> ()): RBXScriptConnection
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

The important part is not the exact names in the example. The important part is the structure: strict typing, alphabetical top sections, flat control flow, focused helpers, clear state ownership, generalized iteration, predictable lifecycle, and a small public module API.

---

### 36. Final Standard

Lily Studio code should feel consistent no matter which developer wrote it. A file should be easy to scan, the important behavior should be easy to find, and the lifecycle of the system should be clear without requiring the reader to trace hidden state through several unrelated places.

A strong Lily implementation should normally have:

- clear names
- simple and longer enough names to explain meaning
- flat control flow
- guard clauses without redundant checks
- no conditional nesting
- minimal use of `Script` and `LocalScript`
- most behavior in ModuleScripts
- event-driven state changes
- generalized Luau iteration
- no `pairs()` or `ipairs()`
- strong Luau typing
- Attributes for simple Instance metadata
- script-created UI
- Lily-owned packages only
- clean lifecycle ownership
- predictable top-level organization
- alphabetical declarations inside logical sections
- no uncontrolled third-party package dependencies
- no hidden behavior changes during refactors

> ## Lily Studio standard
>
> **Write code that another Lily developer can understand quickly, trust immediately, and maintain safely.**
