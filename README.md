# Drawing Library

A Roblox drawing library that renders 2D shapes directly on screen using `ScreenGui` elements parented inside a hidden UI container. Unlike the default Roblox `Drawing` API which is exploit-only, this library works in any context — normal scripts, local scripts, or executors — by using real `GuiObject` instances under the hood.

The library supports a wide range of shapes: lines, squares, circles, and every regular polygon from a triangle all the way up to a dodecagon, plus a fully flexible N-sided polygon (`NGon`) that lets you define as many vertices as you want. Every shape shares a common set of base properties and supports sub-objects like `Stroke`, `Gradient`, and `AutoRotation` for more advanced visual effects.

---

## Table of Contents

- [Getting Started](#getting-started)
- [Drawing.new](#drawingnew)
- [Drawing.IsInShape](#drawingsisinshape)
- [Shared Properties](#shared-properties)
- [Sub-Objects](#sub-objects)
  - [Stroke](#stroke)
  - [Gradient](#gradient)
  - [AutoRotation](#autorotation)
  - [Outlines (Polygons only)](#outlines-polygons-only)
- [UDim2 Auto-Conversion](#udim2-auto-conversion)
- [Shapes](#shapes)
  - [Line](#line)
  - [Square](#square)
  - [Circle](#circle)
  - [Triangle](#triangle)
  - [Quad](#quad)
  - [Pentagon](#pentagon)
  - [Hexagon](#hexagon)
  - [Heptagon](#heptagon)
  - [Octagon](#octagon)
  - [Nonagon](#nonagon)
  - [Decagon](#decagon)
  - [Hendecagon](#hendecagon)
  - [Dodecagon](#dodecagon)
  - [NGon](#ngon)
- [Target Types](#target-types)

---

## Getting Started

Load the library and create your first shape:

```luau
local Drawing = loadstring(game:HttpGet("https://raw.githubusercontent.com/TheRealXORA/Drawing/refs/heads/main/Drawing.luau", true))()

local Circle = Drawing.new("Circle")
Circle.Radius   = 50
Circle.Position = Vector2.new(200, 200)
Circle.Color    = Color3.fromRGB(255, 80, 80)
Circle.Filled   = true
Circle.Visible  = true
```

When you are done with a shape, always clean it up to avoid memory leaks:

```luau
Circle:Destroy()
-- or
Circle:Remove()
```

---

## Drawing.new

```luau
Drawing.new(Shape: string) -> DrawingObject
```

Creates and returns a new drawing object for the given shape name. The name is **case-insensitive** and all non-word characters are stripped before lookup, so `"NGon"`, `"ngon"`, and `"n-gon"` all resolve to the same shape.

**Valid shape names:**

| Name         | Shape                    |
|--------------|--------------------------|
| `line`       | Straight line            |
| `square`     | Rectangle                |
| `circle`     | Circle                   |
| `triangle`   | 3-sided polygon          |
| `quad`       | 4-sided polygon          |
| `pentagon`   | 5-sided polygon          |
| `hexagon`    | 6-sided polygon          |
| `heptagon`   | 7-sided polygon          |
| `octagon`    | 8-sided polygon          |
| `nonagon`    | 9-sided polygon          |
| `decagon`    | 10-sided polygon         |
| `hendecagon` | 11-sided polygon         |
| `dodecagon`  | 12-sided polygon         |
| `ngon`       | N-sided polygon (custom) |

```luau
local Square = Drawing.new("Square")
local Line   = Drawing.new("line")   -- case-insensitive
local NGon   = Drawing.new("NGon")
```

---

## Drawing.IsInShape

```luau
Drawing.IsInShape(Shape: DrawingObject, Target: Vector2 | UDim2 | Vector3 | BasePart | Model, OnScreenCheck: boolean?) -> boolean
```

Returns `true` if the given `Target` position lies within the bounds of `Shape`. Works for all shape types including polygons.

| Parameter     | Type                                         | Description                                            |
|---------------|----------------------------------------------|--------------------------------------------------------|
| Shape         | DrawingObject                                | A valid drawing object returned by `Drawing.new`       |
| Target        | Vector2 / UDim2 / Vector3 / BasePart / Model | The position to test against the shape                 |
| OnScreenCheck | boolean?                                     | When `false`, skips the on-screen check for 3D targets |

```luau
local IsInside = Drawing.IsInShape(Circle, Vector2.new(200, 200))
-- Returns true if the point is inside the circle
```

---

## Shared Properties

Every drawing object inherits these base properties regardless of shape type.

| Property     | Type    | Default             | Description                                          |
|--------------|---------|---------------------|------------------------------------------------------|
| Visible      | boolean | `true`              | Whether the shape is rendered on screen              |
| Transparency | number  | `1`                 | Opacity from `0` (invisible) to `1` (fully opaque)   |
| Color        | Color3  | `Color3.new(1,1,1)` | Fill color of the shape                              |
| ZIndex       | number  | `0`                 | Render order — higher values draw on top             |
| AnchorPoint  | Vector2 | `Vector2.one / 2`   | Pivot point for positioning (0–1 range on each axis) |

### Shared Methods

| Method       | Description                                                              |
|--------------|--------------------------------------------------------------------------|
| `:Remove()`  | Destroys the underlying GuiObject instance and clears the drawing object |
| `:Destroy()` | Alias for `:Remove()`                                                    |

---

## Sub-Objects

Sub-objects are special nested tables attached to drawing objects that control extra visual behaviour like borders, gradients, and auto-spinning. **You cannot assign a sub-object directly** — you must set its properties one by one.

```luau
-- ✅ Correct
Shape.Stroke.Color   = Color3.fromRGB(255, 0, 0)
Shape.Stroke.Enabled = true

-- ❌ Wrong — will throw an error
Shape.Stroke = { Color = Color3.fromRGB(255, 0, 0) }
```

---

### Stroke

Controls the border drawn around the shape. Available on `Line`, `Square`, `Circle`, and all polygon `Outlines`.

```luau
Shape.Stroke.Enabled      = true
Shape.Stroke.Color        = Color3.fromRGB(0, 0, 0)
Shape.Stroke.Thickness    = 2
Shape.Stroke.LineJoinMode = Enum.LineJoinMode.Round
```

| Property     | Type              | Default                   | Description                                       |
|--------------|-------------------|---------------------------|---------------------------------------------------|
| Enabled      | boolean           | `true`                    | Whether the stroke is drawn                       |
| Color        | Color3            | `Color3.new(1,1,1)`       | Color of the stroke                               |
| Thickness    | number            | `1`                       | Width of the stroke in pixels                     |
| LineJoinMode | Enum.LineJoinMode | `Enum.LineJoinMode.Miter` | Corner join style for the stroke                  |
| Gradient     | Gradient          | —                         | Gradient applied to the stroke (read-only handle) |

---

### Gradient

Controls a color or transparency gradient applied across the shape. Available on `Line`, `Square`, `Circle`, polygon fill, polygon `Outlines`, and `Stroke`.

```luau
Shape.Gradient.Enabled      = true
Shape.Gradient.Rotation     = 45
Shape.Gradient.Color        = ColorSequence.new(Color3.fromRGB(255, 0, 0), Color3.fromRGB(0, 0, 255))
Shape.Gradient.Transparency = NumberSequence.new(0)
Shape.Gradient.Offset       = Vector2.new(0, 0)
```

| Property     | Type           | Default                 | Description                                              |
|--------------|----------------|-------------------------|----------------------------------------------------------|
| Enabled      | boolean        | `false`                 | Whether the gradient is applied                          |
| Color        | ColorSequence  | White → Black → White   | Color gradient mapped across the shape                   |
| Rotation     | number         | `0`                     | Rotation of the gradient direction in degrees            |
| Transparency | NumberSequence | `NumberSequence.new(0)` | Transparency gradient mapped across the shape            |
| Offset       | Vector2        | `Vector2.new(0, 0)`     | Offset of the gradient origin point                      |
| AutoRotation | AutoRotation   | —                       | Auto-rotation sub-object for the gradient (read-only handle) |

---

### AutoRotation

Automatically rotates a shape or gradient every frame using `RunService.Heartbeat`. Available on `Square`, `Circle`, all polygon shapes, and `Gradient`.

The rotation loops between `Start` and `End` degrees, incrementing by `Amount * Speed` degrees per second.

```luau
Shape.AutoRotation.Enabled = true
Shape.AutoRotation.Amount  = 90
Shape.AutoRotation.Speed   = 1
Shape.AutoRotation.Start   = 0
Shape.AutoRotation.End     = 360
```

| Property | Type    | Default | Description                                                          |
|----------|---------|---------|----------------------------------------------------------------------|
| Enabled  | boolean | `false` | Whether auto-rotation is active                                      |
| Amount   | number  | `90`    | Base degrees rotated per second                                      |
| Speed    | number  | `1`     | Multiplier applied on top of `Amount`                                |
| Start    | number  | `0`     | Lower bound of the rotation range in degrees                         |
| End      | number  | `360`   | Upper bound of the rotation range; rotation wraps back at this value |

> **Note:** `Line` does not support `AutoRotation` — its angle is always derived from the `From` and `To` positions.

---

### Outlines (Polygons only)

Available on all polygon shapes (`Triangle`, `Quad`, … `NGon`). Controls the border lines drawn along each edge of the polygon. Cannot be assigned directly — set individual properties.

```luau
Shape.Outlines.Visible      = true
Shape.Outlines.Color        = Color3.fromRGB(0, 0, 0)
Shape.Outlines.Thickness    = 2
Shape.Outlines.Transparency = 1
Shape.Outlines.ZIndex       = 1
```

| Property     | Type     | Default             | Description                            |
|--------------|----------|---------------------|----------------------------------------|
| Visible      | boolean  | `true`              | Whether the outline edges are rendered |
| Transparency | number   | `1`                 | Opacity of the outlines                |
| Color        | Color3   | `Color3.new(1,1,1)` | Color of the outline edges             |
| ZIndex       | number   | `0`                 | Render order for the outlines          |
| Thickness    | number   | `1`                 | Width of each outline edge in pixels   |
| Gradient     | Gradient | —                   | Gradient applied to the outline edges  |
| Stroke       | Stroke   | —                   | Stroke applied to the outline edges    |

---

## UDim2 Auto-Conversion

The properties `Position`, `Size`, `From`, and `To` all accept either a `Vector2` **or** a `UDim2`. When you assign a `UDim2`, the library automatically converts it to a `Vector2` using the current viewport resolution before storing it.

This means if you read the property back after setting it with a `UDim2`, **you will get a `Vector2`**, not the original `UDim2` you passed in. This is expected behaviour.

```luau
local Square = Drawing.new("Square")

-- You can set with UDim2 — scale and offset are both supported
Square.Position = UDim2.new(0.5, 0, 0.5, 0)  -- center of screen
Square.Size     = UDim2.fromOffset(200, 100)

-- Reading it back will return a Vector2, not a UDim2
print(Square.Position) -- Vector2 (e.g. 960, 540 on a 1920x1080 screen)
```

> **Note:** The conversion happens at the moment of assignment using the viewport size at that time. If the screen is resized afterwards, the stored `Vector2` will not update automatically — re-assign the `UDim2` to recalculate.

---

## Shapes

---

### Line

Draws a straight line between two screen positions.

| Property     | Type            | Default             | Description                    |
|--------------|-----------------|---------------------|--------------------------------|
| From         | Vector2\|UDim2  | `Vector2.zero`      | Start point of the line        |
| To           | Vector2\|UDim2  | `Vector2.zero`      | End point of the line          |
| Thickness    | number          | `1`                 | Width of the line in pixels    |
| Visible      | boolean         | `true`              | Inherited from base            |
| Transparency | number          | `1`                 | Inherited from base            |
| Color        | Color3          | `Color3.new(1,1,1)` | Inherited from base            |
| ZIndex       | number          | `0`                 | Inherited from base            |
| AnchorPoint  | Vector2         | `Vector2.one / 2`   | Inherited from base            |

> **Note:** `From` and `To` accept `Vector2` or `UDim2`. When set with a `UDim2`, the value is immediately converted and stored as a `Vector2`. Reading `From` or `To` back will always return a `Vector2`.

**Sub-objects:** `Stroke`, `Gradient`, `Rounder`

#### Methods

```luau
Line:Trace(From: Vector2 | Vector3 | BasePart | Model, To: Vector2 | Vector3 | BasePart | Model)
```

Automatically updates `From` and `To` by converting 3D world targets to screen positions on each call. If either target goes off-screen, the line is hidden automatically and shown again once both targets are visible.

#### Example

```luau
local Line = Drawing.new("Line")
Line.From      = Vector2.new(100, 100)
Line.To        = Vector2.new(400, 300)
Line.Thickness = 2
Line.Color     = Color3.fromRGB(255, 255, 0)
Line.Visible   = true
```

---

### Square

Draws a rectangle that can be filled or just outlined.

| Property     | Type           | Default             | Description                                 |
|--------------|----------------|---------------------|---------------------------------------------|
| Position     | Vector2\|UDim2 | `Vector2.zero`      | Position of the square on screen            |
| Size         | Vector2\|UDim2 | `Vector2.zero`      | Width and height of the square in pixels    |
| Filled       | boolean        | `true`              | If `false`, only the stroke border is drawn |
| Rotation     | number         | `0`                 | Rotation of the square in degrees           |
| Visible      | boolean        | `true`              | Inherited from base                         |
| Transparency | number         | `1`                 | Inherited from base                         |
| Color        | Color3         | `Color3.new(1,1,1)` | Inherited from base                         |
| ZIndex       | number         | `0`                 | Inherited from base                         |
| AnchorPoint  | Vector2        | `Vector2.one / 2`   | Inherited from base                         |

> **Note:** `Position` and `Size` accept `Vector2` or `UDim2`. When set with a `UDim2`, the value is immediately converted and stored as a `Vector2`. Reading them back will always return a `Vector2`.

**Sub-objects:** `Stroke`, `Gradient`, `Rounder`, `AutoRotation`

> **Note:** When `Filled` is `false`, `Stroke.Enabled` is forced to `true` automatically so the border is always visible.

#### Example

```luau
local Square = Drawing.new("Square")
Square.Position = Vector2.new(200, 200)
Square.Size     = Vector2.new(150, 100)
Square.Color    = Color3.fromRGB(50, 120, 255)
Square.Filled   = true
Square.Visible  = true
```

---

### Circle

Draws a circle defined by a center position and a radius.

| Property     | Type           | Default             | Description                                              |
|--------------|----------------|---------------------|----------------------------------------------------------|
| Position     | Vector2\|UDim2 | `Vector2.zero`      | Center of the circle on screen                           |
| Radius       | number         | `0`                 | Radius of the circle in pixels                           |
| Filled       | boolean        | `true`              | If `false`, only the stroke border is drawn              |
| Rotation     | number         | `0`                 | Rotation in degrees (mainly affects gradient direction)  |
| Visible      | boolean        | `true`              | Inherited from base                                      |
| Transparency | number         | `1`                 | Inherited from base                                      |
| Color        | Color3         | `Color3.new(1,1,1)` | Inherited from base                                      |
| ZIndex       | number         | `0`                 | Inherited from base                                      |
| AnchorPoint  | Vector2        | `Vector2.one / 2`   | Inherited from base                                      |

> **Note:** `Position` accepts `Vector2` or `UDim2`. When set with a `UDim2`, the value is immediately converted and stored as a `Vector2`. Reading it back will always return a `Vector2`.

**Sub-objects:** `Stroke`, `Gradient`, `Rounder`, `AutoRotation`

> **Note:** `Stroke.LineJoinMode` is pre-set to `Enum.LineJoinMode.Round` to ensure smooth circle borders. When `Filled` is `false`, `Stroke.Enabled` is forced to `true` automatically.

#### Example

```luau
local Circle = Drawing.new("Circle")
Circle.Position = Vector2.new(300, 300)
Circle.Radius   = 60
Circle.Color    = Color3.fromRGB(255, 80, 80)
Circle.Filled   = true
Circle.Visible  = true
```

---

### Polygon Shapes

The following shapes all share the same property set and behave identically. Each shape defines its vertices using named point properties (`PointA`, `PointB`, etc.) relative to `Position`.

| Shape       | Sides | Point Properties             |
|-------------|-------|------------------------------|
| Triangle    | 3     | `PointA`, `PointB`, `PointC` |
| Quad        | 4     | `PointA` … `PointD`          |
| Pentagon    | 5     | `PointA` … `PointE`          |
| Hexagon     | 6     | `PointA` … `PointF`          |
| Heptagon    | 7     | `PointA` … `PointG`          |
| Octagon     | 8     | `PointA` … `PointH`          |
| Nonagon     | 9     | `PointA` … `PointI`          |
| Decagon     | 10    | `PointA` … `PointJ`          |
| Hendecagon  | 11    | `PointA` … `PointK`          |
| Dodecagon   | 12    | `PointA` … `PointL`          |

#### Shared Polygon Properties

| Property     | Type           | Default             | Description                                                                         |
|--------------|----------------|---------------------|-------------------------------------------------------------------------------------|
| Position     | Vector2\|UDim2 | `Vector2.zero`      | Origin offset added to all vertex points                                            |
| Filled       | boolean        | `true`              | Whether the interior of the polygon is filled                                       |
| Radius       | number         | `0`                 | When `> 0`, vertices are auto-computed evenly around a circle; `PointX` is ignored |
| Rotation     | number         | `0`                 | Rotation in degrees applied around `Position`                                       |
| PointX       | Vector2        | `Vector2.zero`      | Local offset for each named vertex, relative to `Position`                          |
| Visible      | boolean        | `true`              | Inherited from base                                                                 |
| Transparency | number         | `1`                 | Inherited from base                                                                 |
| Color        | Color3         | `Color3.new(1,1,1)` | Inherited from base                                                                 |
| ZIndex       | number         | `0`                 | Inherited from base                                                                 |
| AnchorPoint  | Vector2        | `Vector2.one / 2`   | Inherited from base                                                                 |

> **Note:** `Position` accepts `Vector2` or `UDim2`. When set with a `UDim2`, the value is immediately converted and stored as a `Vector2`. Reading it back will always return a `Vector2`.

**Sub-objects:** `Gradient`, `Outlines`, `AutoRotation`

> **Tip:** Setting `Radius > 0` is the easiest way to draw a regular polygon. All vertices are spaced evenly around `Position` at the given radius, so you don't need to set any `PointX` values manually.

---

### Triangle

#### Example

```luau
local Triangle = Drawing.new("Triangle")
Triangle.Position = Vector2.new(300, 300)
Triangle.PointA   = Vector2.new(0, -80)
Triangle.PointB   = Vector2.new(70, 50)
Triangle.PointC   = Vector2.new(-70, 50)
Triangle.Color    = Color3.fromRGB(100, 220, 100)
Triangle.Filled   = true
Triangle.Visible  = true
```

---

### Quad

#### Example

```luau
local Quad = Drawing.new("Quad")
Quad.Position = Vector2.new(300, 300)
Quad.PointA   = Vector2.new(-80, -60)
Quad.PointB   = Vector2.new(80, -60)
Quad.PointC   = Vector2.new(80, 60)
Quad.PointD   = Vector2.new(-80, 60)
Quad.Color    = Color3.fromRGB(255, 160, 0)
Quad.Filled   = true
Quad.Visible  = true
```

---

### Pentagon

#### Example

```luau
local Pentagon = Drawing.new("Pentagon")
Pentagon.Position = Vector2.new(300, 300)
Pentagon.Radius   = 80
Pentagon.Color    = Color3.fromRGB(180, 100, 255)
Pentagon.Filled   = true
Pentagon.Visible  = true
```

---

### Hexagon

#### Example

```luau
local Hexagon = Drawing.new("Hexagon")
Hexagon.Position = Vector2.new(300, 300)
Hexagon.Radius   = 80
Hexagon.Color    = Color3.fromRGB(0, 200, 180)
Hexagon.Filled   = true
Hexagon.Visible  = true
```

---

### Heptagon

#### Example

```luau
local Heptagon = Drawing.new("Heptagon")
Heptagon.Position = Vector2.new(300, 300)
Heptagon.Radius   = 80
Heptagon.Color    = Color3.fromRGB(255, 80, 160)
Heptagon.Filled   = true
Heptagon.Visible  = true
```

---

### Octagon

#### Example

```luau
local Octagon = Drawing.new("Octagon")
Octagon.Position = Vector2.new(300, 300)
Octagon.Radius   = 80
Octagon.Color    = Color3.fromRGB(80, 160, 255)
Octagon.Filled   = true
Octagon.Visible  = true
```

---

### Nonagon

#### Example

```luau
local Nonagon = Drawing.new("Nonagon")
Nonagon.Position = Vector2.new(300, 300)
Nonagon.Radius   = 80
Nonagon.Color    = Color3.fromRGB(255, 200, 50)
Nonagon.Filled   = true
Nonagon.Visible  = true
```

---

### Decagon

#### Example

```luau
local Decagon = Drawing.new("Decagon")
Decagon.Position = Vector2.new(300, 300)
Decagon.Radius   = 80
Decagon.Color    = Color3.fromRGB(50, 255, 120)
Decagon.Filled   = true
Decagon.Visible  = true
```

---

### Hendecagon

#### Example

```luau
local Hendecagon = Drawing.new("Hendecagon")
Hendecagon.Position = Vector2.new(300, 300)
Hendecagon.Radius   = 80
Hendecagon.Color    = Color3.fromRGB(255, 100, 50)
Hendecagon.Filled   = true
Hendecagon.Visible  = true
```

---

### Dodecagon

#### Example

```luau
local Dodecagon = Drawing.new("Dodecagon")
Dodecagon.Position = Vector2.new(300, 300)
Dodecagon.Radius   = 80
Dodecagon.Color    = Color3.fromRGB(150, 255, 200)
Dodecagon.Filled   = true
Dodecagon.Visible  = true
```

---

### NGon

An arbitrary polygon with a configurable number of sides. Shares all the same properties as the fixed polygon shapes, with two additional properties specific to it.

| Property | Type    | Default        | Description                                                                           |
|----------|---------|----------------|---------------------------------------------------------------------------------------|
| Points   | number  | `3`            | Number of vertices (whole integer ≥ 3). Increasing this creates new `PointN` entries |
| PointN   | Vector2 | `Vector2.zero` | Local offset for vertex N, where N is `1` through `Points`                            |

All shared polygon properties and sub-objects apply.

> **Note:** Increasing `Points` automatically creates the new `PointN` entries initialized to `Vector2.zero`. Decreasing `Points` does not remove old entries — they are simply ignored during rendering.

#### Example

```luau
-- Using Radius for an evenly-spaced 5-sided shape
local NGon = Drawing.new("NGon")
NGon.Points   = 5
NGon.Position = Vector2.new(300, 300)
NGon.Radius   = 80
NGon.Color    = Color3.fromRGB(200, 150, 255)
NGon.Filled   = true
NGon.Visible  = true

-- Using manual points for a custom 5-sided shape
local NGon2 = Drawing.new("NGon")
NGon2.Points  = 5
NGon2.Point1  = Vector2.new(0, -80)
NGon2.Point2  = Vector2.new(76, -25)
NGon2.Point3  = Vector2.new(47, 65)
NGon2.Point4  = Vector2.new(-47, 65)
NGon2.Point5  = Vector2.new(-76, -25)
NGon2.Color   = Color3.fromRGB(200, 150, 255)
NGon2.Filled  = true
NGon2.Visible = true
```

---

## Target Types

Properties like `Position`, `From`, `To`, and `Size`, as well as functions like `:Trace()` and `Drawing.IsInShape()`, all accept a flexible target type that is automatically resolved to a `Vector2` screen position.

| Type       | Behaviour                                                                                          |
|------------|----------------------------------------------------------------------------------------------------|
| `Vector2`  | Used directly as a screen position                                                                 |
| `UDim2`    | Converted to `Vector2` using the current viewport resolution at the time of assignment             |
| `Vector3`  | Projected via `Camera:WorldToViewportPoint` — returns `nil` if off-screen                         |
| `BasePart` | The closest visible surface point of the part is projected to screen                              |
| `Model`    | The closest descendant `BasePart` to screen center is used; final position comes from model pivot  |

> **Important:** When a `UDim2` is assigned to any property, it is immediately converted to a `Vector2` and that is what gets stored. Reading the property back after assigning a `UDim2` will always return a `Vector2`.
