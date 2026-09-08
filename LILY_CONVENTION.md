# Lily Coding Convention

> A simple coding standard for the **Lily** Roblox/Luau codebase.

This guide explains how Lily code should be written, organized, typed, reviewed, and maintained.

The main goal is simple:

> **Lily code should be easy to read, easy to understand, easy to change, and hard to misuse.**

Lily code should not try to look clever.  
It should look clear.

---

## Table of Contents

- [1. Main Goals](#1-main-goals)
- [2. Preserve Existing Behavior](#2-preserve-existing-behavior)
- [3. Naming](#3-naming)
- [4. Guard Clauses](#4-guard-clauses)
- [5. No Conditional Nesting](#5-no-conditional-nesting)
- [6. Avoid `else` and `elseif`](#6-avoid-else-and-elseif)
- [7. Positive Conditions](#7-positive-conditions)
- [8. Loops](#8-loops)
- [9. Functions](#9-functions)
- [10. Function Arguments](#10-function-arguments)
- [11. State](#11-state)
- [12. Cleanup](#12-cleanup)
- [13. File Organization](#13-file-organization)
- [14. Lily Separators](#14-lily-separators)
- [15. Formatting](#15-formatting)
- [16. Type Checking](#16-type-checking)
- [17. Services](#17-services)
- [18. Comments](#18-comments)
- [19. Error Handling](#19-error-handling)
- [20. Performance](#20-performance)
- [21. Luau Local/Register Limit](#21-luau-localregister-limit)
- [22. Avoid Boolean Expression Control Flow](#22-avoid-boolean-expression-control-flow)
- [23. Composition Over Inheritance](#23-composition-over-inheritance)
- [24. Public State](#24-public-state)
- [25. Generic Solutions](#25-generic-solutions)
- [26. Lily Anti-Patterns](#26-lily-anti-patterns)
- [27. Example Lily Function](#27-example-lily-function)
- [28. Example Lily Module](#28-example-lily-module)
- [29. Review Checklist](#29-review-checklist)
- [30. Main Lily Principle](#30-main-lily-principle)

---

# 1. Main Goals

Lily code should be:

- **clear**
- **compact**
- **type-safe**
- **predictable**
- **easy to debug**
- **easy to clean up**
- **easy to extend**
- **performant**
- **consistent**
- **boring in a good way**

A developer should be able to open a Lily file and quickly understand:

1. what the file does
2. what state it owns
3. what functions are public
4. what data is expected
5. what can fail
6. what gets cleaned up
7. where important logic lives

### Quick rule

> If code saves a few lines but becomes harder to read, Lily should usually choose the clearer version.

---

# 2. Preserve Existing Behavior

When cleaning or refactoring Lily code, keep the existing behavior the same unless a behavior change was explicitly requested.

This is one of the most important Lily rules.

## Do not silently change

- RemoteEvent names
- RemoteFunction names
- payload fields
- default values
- effect timing
- state behavior
- cleanup timing
- input behavior
- networking behavior
- module return values
- public method names
- existing APIs

## Example

If the current remote sends:

```lua
remote:FireServer({
	value = value,
	enabled = enabled,
})
```

a cleanup should **not** silently change it to:

```lua
remote:FireServer({
	amount = value,
	active = enabled,
})
```

Even if the new names seem better, that is a behavior/API change.

### Why

Large systems often have many files depending on the same API.  
A small silent change can break something far away from the edited file.

### Quick rule

> **Refactoring should improve the code without changing what the code does.**

---

# 3. Naming

Good names reduce the need for comments.

A name should tell the reader what something is or what it does.

---

## 3.1 Use `camelCase`

Lily uses `camelCase` for normal variables, functions, fields, and module methods.

### Good

```lua
local profileToken
local selectedItems
local colorBrightness
local poolController

local function updateColor()
end

function module.setActiveProfile()
end
```

### Avoid

```lua
local ProfileToken
local selected_items
local COLORBRIGHTNESS
```

### Quick rule

> Normal Lily names use `camelCase`.

---

## 3.2 Use Full Words

Do not shorten names unless the short name is already a very common technical term.

### Good

```lua
local player
local connection
local controller
local profileToken
local selectedItems
```

### Avoid

```lua
local plr
local conn
local ctrl
local prof
local sel
```

### Why

Short names save very little typing but make larger files harder to scan.

### Quick rule

> Prefer a longer obvious name over a short unclear name.

---

## 3.3 Boolean Names Should Read Like Questions

Boolean names should sound like something that can be answered with **yes** or **no**.

### Good

```lua
local isRunning
local isFirstRun
local hasAccess
local shouldUpdate
local wasCalled
```

### Avoid

```lua
local running
local first
local access
local flag
```

### Why

This makes conditions easier to read:

```lua
if hasAccess then
```

reads naturally.

---

## 3.4 Function Names Should Describe Actions

Functions should usually start with a verb.

### Good

```lua
updatePool()
createObject()
destroyContext()
sendInput()
resolveProfile()
releasePressedKeys()
```

### Avoid

```lua
pool()
objectThing()
contextData()
inputStuff()
```

---

## 3.5 Event Handlers Should Explain When They Run

Use names that describe the event.

### Good

```lua
onInputBegan()
onInputEnded()
onClick()
onRemoteEvent()
onAttributeChanged()
```

### Avoid

```lua
input()
click()
remote()
changed()
```

### Quick rule

> A function name should make the call site easy to understand without opening the function.

---

# 4. Guard Clauses

Guard clauses are a core Lily convention.

A guard clause exits a function early when a required condition is not met.

## Good

```lua
local function updatePool(context)
	if not context then return end
	if context.destroyed then return end
	if not context.pool then return end

	context.pool.Enabled = true
end
```

## Avoid

```lua
local function updatePool(context)
	if context then
		if not context.destroyed then
			if context.pool then
				context.pool.Enabled = true
			end
		end
	end
end
```

## Why

Guard clauses:

- remove nesting
- make failure conditions obvious
- keep the main logic near the left side of the file
- make functions easier to scan
- make type narrowing easier

---

## 4.1 Do Not Add Useless Guards

Guard clauses are good only when they protect a real condition.

### Avoid

```lua
local function sendInput(context, keyCode, pressed)
	if not context then return end
	if not context.remote then return end
	if not context.remote.Parent then return end

	context.remote:FireServer({
		keyCode = keyCode,
		pressed = pressed,
	})
end
```

If the caller already guarantees a valid context and remote, keep the function simple:

### Better

```lua
local function sendInput(context, keyCode, pressed)
	context.remote:FireServer({
		keyCode = keyCode,
		pressed = pressed,
	})
end
```

### Quick rule

> **Use necessary guards. Remove redundant guards.**

---

# 5. No Conditional Nesting

Lily code should avoid conditional nesting entirely.

This is a hard style preference.

## Avoid

```lua
if context then
	if pool then
		updatePool(pool)
	end
end
```

## Prefer

```lua
if not context then return end
if not pool then return end

updatePool(pool)
```

---

## 5.1 Inside Loops

Use `continue` instead of nesting.

### Avoid

```lua
for _, item in pool do
	if item.Parent then
		if not item:GetAttribute("Disabled") then
			updateItem(item)
		end
	end
end
```

### Prefer

```lua
for _, item in pool do
	if not item.Parent then continue end
	if item:GetAttribute("Disabled") then continue end

	updateItem(item)
end
```

---

## 5.2 If a Function Still Needs Nesting

Extract a small helper.

### Example

Instead of:

```lua
if context then
	if context.pool then
		if context.pool.Enabled then
			updatePool(context.pool)
		end
	end
end
```

write:

```lua
local function canUpdatePool(context)
	if not context then return false end
	if not context.pool then return false end

	return context.pool.Enabled
end

--————————————————————————————————————————————————————————————————————--

if not canUpdatePool(context) then return end

updatePool(context.pool)
```

### Quick rule

> Use **guards**, `continue`, `break`, lookup tables, or small helpers instead of nested conditionals.

---

# 6. Avoid `else` and `elseif`

Lily prefers flat control flow.

---

## 6.1 Avoid `else`

### Avoid

```lua
if enabled then
	startPool()
else
	stopPool()
end
```

### Prefer

```lua
if enabled then
	startPool()
	return
end

stopPool()
```

---

## 6.2 Avoid `elseif`

### Avoid

```lua
if mode == "All" then
	selectAll()
elseif mode == "Even" then
	selectEven()
elseif mode == "Odd" then
	selectOdd()
end
```

### Prefer a lookup when it fits

```lua
local handlers = {
	All = selectAll,
	Even = selectEven,
	Odd = selectOdd,
}

local handler = handlers[mode]
if not handler then return end

handler()
```

### Why

Large `if / elseif / else` chains become hard to scan and harder to extend.

### Quick rule

> Lily control flow should move downward in a straight line.

---

# 7. Positive Conditions

Prefer positive names and positive conditions.

## Good

```lua
if isRunning then
	updatePool()
end
```

## Avoid

```lua
if not isNotRunning then
	updatePool()
end
```

## Good

```lua
local hasValue = value ~= nil
```

## Avoid

```lua
local isMissingValue = value == nil
```

Use the positive form unless the negative state is the real concept.

### Why

Positive conditions are usually easier to understand quickly.

---

# 8. Event-Driven Code and Loops

Lily should be **event-driven** whenever possible.

Do not constantly check whether something changed.

Instead:

> **When the thing changes, the code should run because of that change.**

This means Lily should prefer Roblox signals, events, and callbacks over polling.

---

## 8.1 No Polling Loops

Do not write loops that repeatedly check state.

### Avoid

```lua
while true do
	if pool.Enabled then
		updatePool()
	end

	task.wait()
end
```

### Avoid

```lua
while task.wait(.1) do
	if object.Value ~= previousValue then
		previousValue = object.Value
		updateValue(object.Value)
	end
end
```

### Prefer

```lua
object:GetPropertyChangedSignal("Value"):Connect(function()
	updateValue(object.Value)
end)
```

---

## 8.2 React When the Value Changes

If Roblox already provides a signal for the change, use it.

### Property changes

```lua
object:GetPropertyChangedSignal("Enabled"):Connect(function()
	updatePool(object.Enabled)
end)
```

### Attributes

```lua
object:GetAttributeChangedSignal("active"):Connect(function()
	updateState(object:GetAttribute("active"))
end)
```

### Value objects

```lua
valueObject.Changed:Connect(function(value)
	updateValue(value)
end)
```

### Remote events

```lua
remote.OnClientEvent:Connect(function(arguments)
	updateState(arguments)
end)
```

### Input

```lua
userInputService.InputBegan:Connect(onInputBegan)
userInputService.InputEnded:Connect(onInputEnded)
```

### Quick rule

> **Do not watch for changes in a loop. Subscribe to the change.**

---

## 8.3 Do Not Use Heartbeat as a General Change Detector

Do not use `Heartbeat`, `RenderStepped`, or `Stepped` just to check whether normal state changed.

### Avoid

```lua
runService.Heartbeat:Connect(function()
	if context.value ~= lastValue then
		lastValue = context.value
		updateValue(context.value)
	end
end)
```

### Prefer

Call `updateValue()` at the point where the value changes, or connect to the signal that reports the change.

```lua
local function setValue(context, value)
	if context.value == value then return end

	context.value = value
	updateValue(context)
end
```

---

## 8.4 Frame Loops Are Only for Real Frame-Based Work

Some Lily systems genuinely need per-frame updates.

Examples include:

- effect animation
- movement interpolation
- synchronized clocks
- real-time visual animation
- frame-based engine calculations

Those are valid uses of:

```lua
runService.Heartbeat
```

or:

```lua
runService.RenderStepped
```

Do not use a frame connection for ordinary state watching.

### Quick rule

> **Frame updates are for animation and real-time simulation, not for checking whether normal data changed.**

---

## 8.5 Finite Collection Iteration Is Still Allowed

This rule does **not** mean Lily can never process a table.

A one-time iteration is fine when Lily actually needs to process several items.

### Good

```lua
for _, item in items do
	disconnectItem(item)
end
```

### Good

```lua
for key, value in data do
	copy[key] = value
end
```

What Lily avoids is a loop whose job is to repeatedly wait and check for changes.

---

## 8.6 Do Not Use `pairs()` or `ipairs()`

Lily uses generalized Luau iteration.

### Good

```lua
for key, value in data do
end
```

```lua
for index, item in items do
end
```

### Avoid

```lua
for key, value in pairs(data) do
end
```

```lua
for index, item in ipairs(items) do
end
```

### Quick rule

> **Never use `pairs()` or `ipairs()` in Lily code.**

---

## 8.7 Prefer the Change Source

The best place to react is as close as possible to the place where the state changes.

### Good

```lua
local function setEnabled(context, enabled)
	if context.enabled == enabled then return end

	context.enabled = enabled
	updateViews(context)
end
```

This is better than storing the value and having another system constantly inspect it.

---

## 8.8 Avoid Duplicate Change Sources

Do not update the same state from several unrelated polling systems.

Prefer one clear owner.

### Better flow

```text
input changes
    ↓
state setter runs
    ↓
state updates
    ↓
views / remotes / effects react
```

This makes the data flow easy to understand.

---

## 8.9 Event-Driven Cleanup

Event-driven code still needs cleanup.

Every connection should belong to something that can disconnect it.

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

Do not create signals without a clear cleanup path.

---

## 8.10 Event-Driven Review Rule

Before using a loop or frame connection, ask:

1. Is there already an event for this?
2. Is there a property-changed signal?
3. Is there an attribute-changed signal?
4. Can the code run directly where the value changes?
5. Does this truly need to run every frame?

If the answer to one of the first four is yes, do not poll.

### Main rule

> **Lily reacts to changes. Lily does not repeatedly ask whether something changed.**

---

# 9. Functions

A function should have one clear job.

## Good

```lua
local function sendColor()
end

local function syncColorFaders()
end

local function updateColorViews()
end
```

## Avoid

One function that:

- validates data
- creates UI
- changes state
- sends remotes
- starts effects
- destroys objects
- handles cleanup

all at once.

---

## 9.1 Extract Helpers Only When They Help

A helper should reduce at least one of these:

- repetition
- nesting
- complexity
- unclear responsibilities

### Bad helper

```lua
local function enableObject(object)
	object.Enabled = true
end
```

if it is used once and adds no meaning.

### Better helper

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

This is useful because the cleanup behavior is repeated and meaningful.

---

## 9.2 Keep Helpers Single-Purpose

A helper should be easy to describe in one sentence.

If the description needs several unrelated verbs, the function is probably doing too much.

### Quick rule

> One function = one responsibility.

---

# 10. Function Arguments

Function calls should be obvious when read.

---

## 10.1 Avoid Boolean Arguments That Hide Meaning

### Avoid

```lua
updatePool(pool, true, false)
```

The reader cannot easily tell what `true` and `false` mean.

### Prefer

```lua
updatePool(pool, {
	replicate = true,
	force = false,
})
```

---

## 10.2 Avoid Positional `nil`

### Avoid

```lua
createPool(name, nil, true)
```

### Prefer

```lua
createPool(name, {
	enabled = true,
})
```

---

## 10.3 Do Not Use Options Tables Everywhere

Use them only when they improve the call site.

Simple functions can stay simple:

```lua
setBrightness(value)
```

### Quick rule

> The function call should explain itself.

---

# 11. State

State should have a clear owner.

---

## 11.1 Avoid Global State

Prefer state owned by the module, feature, profile, or object.

### Good

```lua
local contexts = {}
```

```lua
contexts[profileToken] = {
	connections = {},
	selectedItems = {},
	destroyed = false,
}
```

---

## 11.2 Separate Independent Instances

If several profiles, pools, controllers, or effects can exist at the same time, they should not accidentally share state.

### Bad idea

```lua
local currentValue = 0
```

when multiple independent objects need their own value.

### Better

```lua
contexts[profileToken].value = 0
```

---

## 11.3 Keep Related State Together

Color state should stay near color behavior.

Effect state should stay near effect behavior.

Pool state should stay near pool behavior.

### Why

Scattered state makes bugs difficult to trace.

### Quick rule

> The code that owns a value should also control how that value changes.

---

# 12. Cleanup

Everything Lily creates should have a cleanup path.

This includes:

- `RBXScriptConnection`
- UI instances
- temporary tables
- runtime contexts
- cached objects
- tasks associated with an object
- profile state
- effect state
- references that could block garbage collection

---

## 12.1 Connections

### Good

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

---

## 12.2 Contexts

When a context is destroyed:

```lua
contexts[profileToken] = nil
```

Do not keep stale entries.

---

## 12.3 UI

Temporary UI should be destroyed when it is no longer valid.

---

## 12.4 Avoid Stacking

Do not accidentally create:

- duplicate connections
- duplicate tasks
- duplicate runtimes
- duplicate UI
- duplicate listeners

every time setup runs.

### Quick rule

> If Lily creates something, Lily should know how to remove it.

---

# 13. File Organization

Lily files should use a predictable top-level order.

## Recommended Order

| Order | Section |
|---:|---|
| 1 | File description |
| 2 | Services |
| 3 | Module |
| 4 | Constants / configuration |
| 5 | State |
| 6 | Dependencies |
| 7 | Types |
| 8 | Private helpers |
| 9 | Feature functions |
| 10 | Public module methods |
| 11 | `return module` |

---

## Example

```lua
--!strict

-- handles pool input

local replicatedStorage = game:GetService("ReplicatedStorage")
local userInputService = game:GetService("UserInputService")

--————————————————————————————————————————————————————————————————————--

local module = {}

--————————————————————————————————————————————————————————————————————--

local inputProfile = "Foo"

--————————————————————————————————————————————————————————————————————--

local contexts = {}

--————————————————————————————————————————————————————————————————————--

local source = replicatedStorage:WaitForChild("source")

--————————————————————————————————————————————————————————————————————--

type Context = {
	remote: RemoteEvent,
}

--————————————————————————————————————————————————————————————————————--

local function getContext(profileToken: string): Context?
	return contexts[profileToken]
end

--————————————————————————————————————————————————————————————————————--

function module.setup(profileToken: string): boolean
	return true
end

--————————————————————————————————————————————————————————————————————--

return module
```

### Why

A predictable structure reduces time spent searching through files.

---

# 14. Lily Separators

Use the standard Lily separator:

```lua
--————————————————————————————————————————————————————————————————————--
```

Use it between:

- top-level functions
- major file sections
- public module methods

## Example

```lua
local function foo()
end

--————————————————————————————————————————————————————————————————————--

local function bar()
end

--————————————————————————————————————————————————————————————————————--
```

### Quick rule

> Every top-level function should be visually separated.

---

# 15. Formatting

Lily formatting should be compact but not cramped.

---

## 15.1 Keep Related Lines Together

### Good

```lua
local function setState(context, value)
	context.value = value
	updateViews(context)
end
```

### Avoid

```lua
local function setState(context, value)

	context.value = value


	updateViews(context)

end
```

---

## 15.2 Keep Simple Expressions on One Line

### Good

```lua
local x = math.clamp((mouseX - position.X) / size.X, 0, 1)
local y = math.clamp((mouseY - position.Y) / size.Y, 0, 1)
```

### Avoid

```lua
local x = math.clamp(
	(mouseX - position.X) / size.X,
	0,
	1
)
```

when the one-line version is still easy to read.

---

## 15.3 Break Long Data Tables Cleanly

### Good

```lua
local button = interface.TextButton(parent, {
	Name = name,
	BackgroundColor3 = Color3.fromRGB(0, 0, 0),
	Text = title,
	TextSize = 33,
})
```

### Quick rule

> Compact expressions; readable structures.

---

# 16. Type Checking

Type checking is part of the Lily standard.

Lily should use Luau types to catch mistakes before runtime and improve autocomplete.

---

## 16.1 Use `--!strict`

Production Lily modules should normally start with:

```lua
--!strict
```

### Why

Strict mode can catch:

- wrong property names
- invalid argument types
- missing fields
- incorrect return values
- accidental `nil`
- incorrect module usage
- impossible assumptions

before the code runs.

---

## 16.2 Type Function Parameters

### Good

```lua
local function getContext(profileToken: string): Context?
	return contexts[profileToken]
end
```

### Avoid when the type is already known

```lua
local function getContext(profileToken)
	return contexts[profileToken]
end
```

---

## 16.3 Type Return Values

### Good

```lua
function module.setup(profileToken: string): boolean
	return true
end
```

```lua
local function updatePool(pool: Pool): ()
end
```

### Why

Return types make public APIs easier to understand and harder to accidentally change.

---

## 16.4 Define Reusable Types

### Good

```lua
export type Pool = {
	name: string,
	enabled: boolean,
	items: { Instance },
}
```

```lua
type Context = {
	profileToken: string,
	remote: RemoteEvent,
	connections: { RBXScriptConnection },
}
```

### Avoid

Repeating a large anonymous table type in many functions.

---

## 16.5 Export Public Types

Use `export type` when another module needs the type.

```lua
export type Personality = {
	name: string,
	title: string,
	category: string,
}
```

Keep internal implementation types private when outside code does not need them.

---

## 16.6 Type Collections

### Good

```lua
local contexts: { [string]: Context } = {}
local connections: { RBXScriptConnection } = {}
local selectedItems: { number } = {}
```

### Why

Typed collections prevent accidental insertion of the wrong value.

---

## 16.7 Use Optional Types Intentionally

If a value can really be missing:

```lua
local activeProfileToken: string?
```

```lua
local function getContext(profileToken: string): Context?
end
```

Do not make everything optional just to avoid type errors.

---

## 16.8 Narrow Types With Guards

Guard clauses work well with Luau type narrowing.

### Good

```lua
local function useRemote(remote: Instance?)
	if not remote then return end
	if not remote:IsA("RemoteEvent") then return end

	remote:FireServer()
end
```

After the guards, Luau understands that `remote` is a `RemoteEvent`.

---

## 16.9 Avoid `any`

Do not use `any` just to silence the type checker.

### Avoid

```lua
local value: any = arguments.value
```

### Prefer

```lua
local value: unknown = arguments.value
if type(value) ~= "number" then return end
```

---

## 16.10 Avoid Unsafe Casts

### Avoid

```lua
local remote = object :: RemoteEvent
```

when the object has not been verified.

### Prefer

```lua
if not object:IsA("RemoteEvent") then return end

local remote = object
```

Use casts only when the architecture guarantees the type and Luau cannot infer it.

---

## 16.11 Runtime Validation Still Matters

Types cannot protect data coming from outside the trusted module at runtime.

Still validate:

- RemoteEvent arguments
- user input
- JSON data
- attributes
- dynamically found Instances
- unknown module output

### Example

```lua
local function onRemote(arguments: unknown)
	if type(arguments) ~= "table" then return end

	local value = arguments.value
	if type(value) ~= "number" then return end

	updateValue(value)
end
```

### Quick rule

> Static types protect the codebase. Runtime checks protect external data.

---

## 16.12 Type Checking Must Not Change Behavior

Do not change runtime behavior just to make a type error disappear.

Fix:

- the type
- the function signature
- the ownership model
- the validation

before changing behavior.

---

# 17. Services

Keep Roblox services in alphabetical order when practical.

## Good

```lua
local collectionService = game:GetService("CollectionService")
local httpService = game:GetService("HttpService")
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local serverScriptService = game:GetService("ServerScriptService")
```

Only request services the file actually uses.

### Quick rule

> Alphabetical services make files easier to compare and scan.

---

# 18. Comments

Comments should explain **why**, not repeat **what**.

## Avoid

```lua
-- set enabled to true
enabled = true
```

The code already says that.

## Better

```lua
-- Release old held inputs before switching profiles so they cannot stay active.
releasePressedKeys()
```

### Good comment topics

- why unusual behavior is required
- why a workaround exists
- why an optimization exists
- why something must happen in a specific order
- why a seemingly unnecessary guard is actually required

### Quick rule

> Comment the reason, not the obvious action.

---

# 19. Error Handling

Use errors for real programming problems, not normal control flow.

---

## 19.1 Recoverable Failures

Return a status.

```lua
function module.setup(profileToken: string): boolean
	if profileToken == "" then return false end

	return true
end
```

---

## 19.2 Programmer Errors

Use `assert` when an invariant must be true.

```lua
assert(type(definitions) == "table", "Definitions must return a table.")
```

---

## 19.3 Warnings

Use `warn()` when the system can continue but something is wrong or missing.

Do not spam warnings in normal expected flows.

### Quick rule

> Expected failure = return a status.  
> Broken assumption = assert/error.  
> Recoverable problem worth reporting = warn.

---

# 20. Performance

Lily is a real-time Roblox stage-lighting system.

Small costs can become large when repeated:

- every frame
- for many effects
- across hundreds of fixtures
- across several profiles
- across many UI controls

---

## 20.1 Prefer Cached References

Avoid repeatedly searching the hierarchy in hot code.

### Avoid

```lua
local object = workspace.Stage.Pool.Item
```

inside a per-frame loop when the reference can be cached once.

---

## 20.2 Avoid Per-Frame Allocations

Be careful creating new:

- tables
- closures
- arrays
- temporary objects

inside hot loops.

---

## 20.3 Avoid Duplicate Connections

Setup functions should not keep stacking new event connections.

Use clear setup/destroy lifecycles.

---

## 20.4 Avoid Repeated `WaitForChild` in Runtime Loops

Resolve required objects during setup when possible.

---

## 20.5 Optimize the Hot Path First

The most important areas include:

- `Heartbeat`
- `RenderStepped`
- scheduler updates
- effect update loops
- per-fixture patch functions
- high-frequency remote handling

### Quick rule

> Optimize repeated work before worrying about one-time setup work.

---

## 20.6 Do Not Sacrifice Clarity for Tiny Gains

Performance matters, but unreadable code creates maintenance bugs.

Prefer the clearest fast solution.

---

# 21. Luau Local/Register Limit

Large Luau modules can hit the local/register limit.

A common error looks like:

```text
Out of local registers
exceeded limit 200
```

This can happen when a very large file contains too many top-level locals and local functions.

---

## 21.1 Avoid Hundreds of Top-Level Local Helpers

Instead of:

```lua
local function foo()
end

local function bar()
end

local function baz()
end
```

repeated hundreds of times, cold internal helpers can be grouped:

```lua
local internals = {}

function internals.foo()
end

function internals.bar()
end
```

---

## 21.2 Keep Hot Helpers Fast

Do not move every helper into a table automatically.

Performance-sensitive helpers may still be better as locals.

### Quick rule

> Stay safely below Luau's local limit without creating unnecessary architecture.

---

# 22. Avoid Boolean Expression Control Flow

Lily does not use chained `and/or` expressions as control flow.

## Avoid

```lua
local value = condition and firstValue or secondValue
```

Especially avoid long chains:

```lua
local result = firstCondition and firstValue
	or secondCondition and secondValue
	or thirdCondition and thirdValue
	or nil
```

---

## Prefer Explicit Logic

```lua
local result

if firstCondition then
	result = firstValue
	return result
end

if secondCondition then
	result = secondValue
	return result
end

return thirdValue
```

### Why

The explicit version is easier to debug and safer when values like `false` or `nil` are valid.

### Quick rule

> Do not trade readability for expression tricks.

---

# 23. Composition Over Inheritance

Prefer combining focused systems instead of building deep inheritance trees.

## Generic Example

```text
Pool
+ PoolController
+ FooEngine
+ FooController
```

Each part has a clear responsibility.

### Why

Composition is usually easier to:

- replace
- test
- understand
- reuse
- clean up

### Quick rule

> Prefer small parts working together over deep class hierarchies.

---

# 24. Public State

Public state should be controlled when direct mutation could break rules.

## Good

```lua
function controller:getValue(): number
	return self.value
end

function controller:setValue(value: number)
	self.value = math.max(value, 0)
end
```

This makes the allowed state change clear.

Do not create getters and setters for every private variable automatically.

### Quick rule

> Protect state only when the state has rules that need protection.

---

# 25. Generic Solutions

Prefer a reusable solution when several features perform the same real operation.

## Good

```lua
local function disconnectConnections(owner)
	for _, connection in owner.connections do
		connection:Disconnect()
	end

	table.clear(owner.connections)
end
```

This can work for several feature owners.

---

## Do Not Over-Generalize

Do not force unrelated systems into one abstraction just because some code looks similar.

Two systems can stay separate if their meaning is different.

### Quick rule

> Generalize repeated behavior, not unrelated concepts.

---

# 26. Lily Anti-Patterns

These patterns should usually be removed during review.

---

## 26.1 Conditional Nesting

### Avoid

```lua
if context then
	if pool then
		updatePool(pool)
	end
end
```

### Prefer

```lua
if not context then return end
if not pool then return end

updatePool(pool)
```

---

## 26.2 Chained Boolean Control Flow

### Avoid

```lua
local value = condition and foo or bar
```

---

## 26.3 Redundant Guards

### Avoid

```lua
if not context then return end
if not context.remote then return end
```

when the caller already guarantees both.

---

## 26.4 Meaningless Abbreviations

### Avoid

```lua
local ctx
local obj
local mgr
```

Prefer names that explain the value.

---

## 26.5 Hidden Boolean Arguments

### Avoid

```lua
updatePool(pool, true, false, true)
```

---

## 26.6 Excessive Helpers

### Avoid

```lua
local function setTrue(data)
	data.value = true
end
```

if the helper adds no meaning.

---

## 26.7 Giant Functions

Avoid one function controlling an entire feature lifecycle.

---

## 26.8 Polling for Changes

### Avoid

```lua
while true do
	if value ~= previousValue then
		previousValue = value
		updateValue(value)
	end

	task.wait()
end
```

### Prefer

```lua
valueObject.Changed:Connect(function(value)
	updateValue(value)
end)
```

The system should react to the change instead of repeatedly checking for it.

---

## 26.9 `pairs()` and `ipairs()`

### Avoid

```lua
for key, value in pairs(data) do
end
```

```lua
for index, item in ipairs(items) do
end
```

### Prefer

```lua
for key, value in data do
end
```

```lua
for index, item in items do
end
```

---

## 26.10 Stale State

Do not leave old:

- connections
- contexts
- objects
- UI
- tasks
- runtime tables

alive after destruction.

---

## 26.11 Unnecessary `any`

### Avoid

```lua
local data: any
```

just to stop type errors.

---

## 26.12 Unsafe Casts

### Avoid

```lua
local remote = object :: RemoteEvent
```

without a real guarantee.

---

# 27. Example Lily Function

```lua
--!strict

local function updatePoolState(context: Context, poolName: string, enabled: boolean)
	context.activePools[poolName] = nil
	if enabled then context.activePools[poolName] = true end

	for _, view in context.views do
		local buttonData = view.buttons[poolName]
		if not buttonData then continue end
		if buttonData.enabled == enabled then continue end

		buttonData.enabled = enabled
		moveBar(buttonData, enabled)
	end
end
```

## Why this matches Lily style

- descriptive names
- typed parameters
- no conditional nesting
- no `else`
- no `elseif`
- flat loop body
- uses `continue`
- one clear responsibility
- compact formatting

---

# 28. Example Lily Module

```lua
--!strict

-- handles pool input

local replicatedStorage = game:GetService("ReplicatedStorage")
local userInputService = game:GetService("UserInputService")

--————————————————————————————————————————————————————————————————————--

local module = {}

--————————————————————————————————————————————————————————————————————--

type Context = {
	profileToken: string,
	remote: RemoteEvent,
}

--————————————————————————————————————————————————————————————————————--

local contexts: { [string]: Context } = {}
local connections: { RBXScriptConnection } = {}

--————————————————————————————————————————————————————————————————————--

local function connect(signal: RBXScriptSignal, callback: (...any) -> ()): RBXScriptConnection
	local connection = signal:Connect(callback)
	connections[#connections + 1] = connection

	return connection
end

--————————————————————————————————————————————————————————————————————--

local function getContext(profileToken: string): Context?
	local context = contexts[profileToken]
	if not context then return end
	if not context.remote.Parent then return end

	return context
end

--————————————————————————————————————————————————————————————————————--

local function disconnectInput()
	for _, connection in connections do
		connection:Disconnect()
	end

	table.clear(connections)
end

--————————————————————————————————————————————————————————————————————--

function module.setup(profileToken: string): boolean
	if profileToken == "" then return false end
	if getContext(profileToken) then return true end

	return true
end

--————————————————————————————————————————————————————————————————————--

function module.destroy(profileToken: string)
	if not contexts[profileToken] then return end

	contexts[profileToken] = nil
	if next(contexts) then return end

	disconnectInput()
end

--————————————————————————————————————————————————————————————————————--

return module
```

---

# 29. Review Checklist

Use this before calling Lily code finished.

---

## Naming

- [ ] Uses `camelCase`
- [ ] Names are descriptive
- [ ] Avoids unnecessary abbreviations
- [ ] Boolean names read like yes/no questions
- [ ] Functions use action names
- [ ] Event handlers use `on...` names

---

## Control Flow

- [ ] Uses event-driven logic for normal state changes
- [ ] Does not use polling loops
- [ ] Does not use `Heartbeat`, `RenderStepped`, or `Stepped` as normal change detectors
- [ ] Frame updates are only used for real frame-based work
- [ ] Uses necessary guard clauses
- [ ] Removes redundant guard clauses
- [ ] Has no conditional nesting
- [ ] Avoids `else`
- [ ] Avoids `elseif`
- [ ] Does not use `pairs()` or `ipairs()`
- [ ] Uses generalized Luau iteration
- [ ] Uses `continue` where useful
- [ ] Uses `break` when searching is complete
- [ ] Avoids chained `and/or` control flow
- [ ] Keeps conditions easy to understand

---

## Functions

- [ ] Each function has one responsibility
- [ ] Helpers provide real value
- [ ] Function calls are understandable
- [ ] Avoids unclear positional booleans
- [ ] Avoids unnecessary positional `nil`
- [ ] Options tables are used only when helpful

---

## Types

- [ ] Uses `--!strict` when practical
- [ ] Public parameters are typed
- [ ] Public returns are typed
- [ ] Important private helpers are typed
- [ ] Repeated structures use named types
- [ ] Shared types use `export type`
- [ ] Collections are typed
- [ ] Optional values use `?` intentionally
- [ ] Avoids unnecessary `any`
- [ ] Avoids unsafe casts
- [ ] Runtime input is still validated
- [ ] Type fixes do not change runtime behavior

---

## State

- [ ] State has a clear owner
- [ ] Independent instances do not accidentally share state
- [ ] Related state stays near related behavior
- [ ] Old state is removed

---

## Cleanup

- [ ] Connections can be disconnected
- [ ] Contexts can be destroyed
- [ ] Temporary UI can be destroyed
- [ ] Runtime objects can be cleaned up
- [ ] Setup does not stack duplicate listeners
- [ ] Stale references are removed

---

## Performance

- [ ] Hot paths avoid unnecessary allocations
- [ ] Repeated hierarchy searches are avoided
- [ ] Required references are cached when useful
- [ ] Duplicate connections are prevented
- [ ] Per-frame work is kept small
- [ ] Large modules stay below Luau's local/register limit

---

## Formatting

- [ ] Services are organized consistently
- [ ] Top-level sections are easy to find
- [ ] Lily separators are used
- [ ] Simple expressions stay compact
- [ ] Large tables stay readable
- [ ] Blank lines are not excessive

---

## Behavior

- [ ] Existing behavior is preserved
- [ ] Remote names are unchanged unless intentionally changed
- [ ] Payload formats are unchanged unless intentionally changed
- [ ] Default values remain the same
- [ ] Behavior changes are explicit and intentional

---

# 30. Main Lily Principle

> ## Clear first. Compact second. Clever never.

A good Lily file should feel predictable.

A developer should not need to guess:

- what a variable means
- what a boolean argument means
- what state a function changes
- whether cleanup exists
- whether a value can be `nil`
- where a feature lives
- why a condition is nested

The code should explain itself through:

- good names
- flat control flow
- guard clauses
- strong types
- simple functions
- clear state ownership
- consistent formatting

That is the Lily standard.
