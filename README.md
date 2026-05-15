# Drawing Library

A Roblox drawing library that renders 2D shapes directly on screen using `ScreenGui` elements parented inside a hidden UI container, built on top of real `GuiObject` instances.

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
  - [Rounder](#rounder)
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

**Example 1 — Check if the screen center is inside a circle:**

```luau
local Circle = Drawing.new("Circle")
Circle.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Circle.Radius   = 80
Circle.Color    = Color3.fromRGB(0, 200, 255)
Circle.Filled   = true
Circle.Visible  = true

local IsInside = Drawing.IsInShape(Circle, UDim2.new(0.5, 0, 0.5, 0))
print(IsInside) -- true, the center of the screen is inside the circle
```

**Example 2 — Check if a specific pixel is inside a square:**

```luau
local Square = Drawing.new("Square")
Square.Position = Vector2.new(400, 300)
Square.Size     = Vector2.new(200, 120)
Square.Color    = Color3.fromRGB(255, 180, 0)
Square.Filled   = true
Square.Visible  = true

local IsInside = Drawing.IsInShape(Square, Vector2.new(450, 320))
print(IsInside) -- true, the point is within the square bounds
```

**Example 3 — Check if a point is inside a triangle:**

```luau
local Triangle = Drawing.new("Triangle")
Triangle.Position = Vector2.new(150, 500)
Triangle.PointA   = Vector2.new(0, -70)
Triangle.PointB   = Vector2.new(60, 40)
Triangle.PointC   = Vector2.new(-60, 40)
Triangle.Color    = Color3.fromRGB(100, 255, 140)
Triangle.Filled   = true
Triangle.Visible  = true

local IsInside = Drawing.IsInShape(Triangle, Vector2.new(150, 480))
print(IsInside) -- true, the point is inside the triangle
```

**Example 4 — Check using a UDim2 scale position against a polygon:**

```luau
local Hexagon = Drawing.new("Hexagon")
Hexagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Hexagon.Radius   = 100
Hexagon.Color    = Color3.fromRGB(180, 80, 255)
Hexagon.Filled   = true
Hexagon.Visible  = true

-- Test a point slightly off center
local IsInside = Drawing.IsInShape(Hexagon, UDim2.new(0.5, 50, 0.5, 30))
print(IsInside) -- true or false depending on if that offset lands inside the hexagon
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

Sub-objects are special nested tables attached to drawing objects that control extra visual behaviour like borders, gradients, corner rounding, and auto-spinning. **You cannot assign a sub-object directly** — you must set its properties one by one.

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

### Rounder

Controls the corner rounding of the shape via a `UICorner` instance. Available on `Line`, `Square`, and `Circle`. You do not set `Rounder` directly — instead you set its `CornerRadius` property, which is a `UDim`.

```luau
-- Slightly round the corners of a square
Shape.Rounder.CornerRadius = UDim.new(0, 8)

-- Fully round — turns a square into a pill/circle shape
Shape.Rounder.CornerRadius = UDim.new(1, 0)
```

> **Note:** `Circle` already sets `CornerRadius` to `UDim.new(1, 0)` by default to achieve its circular shape. Changing it will distort the circle. For `Line` and `Square`, the default is `UDim.new(0, 0)` (sharp corners).

**Example — rounded square:**

```luau
local Square = Drawing.new("Square")
Square.Position             = UDim2.new(0.5, 0, 0.5, 0)
Square.Size                 = Vector2.new(200, 80)
Square.Color                = Color3.fromRGB(80, 200, 255)
Square.Filled               = true
Square.Visible              = true
Square.Rounder.CornerRadius = UDim.new(0, 16) -- 16px rounded corners
```

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

| Property     | Type           | Default             | Description                    |
|--------------|----------------|---------------------|--------------------------------|
| From         | Vector2\|UDim2 | `Vector2.zero`      | Start point of the line        |
| To           | Vector2\|UDim2 | `Vector2.zero`      | End point of the line          |
| Thickness    | number         | `1`                 | Width of the line in pixels    |
| Visible      | boolean        | `true`              | Inherited from base            |
| Transparency | number         | `1`                 | Inherited from base            |
| Color        | Color3         | `Color3.new(1,1,1)` | Inherited from base            |
| ZIndex       | number         | `0`                 | Inherited from base            |
| AnchorPoint  | Vector2        | `Vector2.one / 2`   | Inherited from base            |

> **Note:** `From` and `To` accept `Vector2` or `UDim2`. When set with a `UDim2`, the value is immediately converted and stored as a `Vector2`. Reading `From` or `To` back will always return a `Vector2`.

**Sub-objects:** `Stroke`, `Gradient`, `Rounder`

#### Methods

```luau
Line:Trace(From: Vector2 | Vector3 | BasePart | Model, To: Vector2 | Vector3 | BasePart | Model)
```

Automatically updates `From` and `To` by converting 3D world targets to screen positions on each call. If either target goes off-screen, the line is hidden automatically and shown again once both targets are visible.

#### Example 1 — Centered diagonal line with a gradient:

```luau
local Line = Drawing.new("Line")
Line.From      = UDim2.new(0.5, -150, 0.5, -80) -- offset left and up from center
Line.To        = UDim2.new(0.5,  150, 0.5,  80) -- offset right and down from center
Line.Thickness = 4
Line.Color     = Color3.fromRGB(255, 80, 180)
Line.Visible   = true

Line.Gradient.Enabled = true
Line.Gradient.Color   = ColorSequence.new(Color3.fromRGB(255, 80, 180), Color3.fromRGB(80, 180, 255))
```

#### Example 2 — Thin white line in the top-left corner with a stroke:

```luau
local Line = Drawing.new("Line")
Line.From      = Vector2.new(50, 80)
Line.To        = Vector2.new(300, 200)
Line.Thickness = 2
Line.Color     = Color3.fromRGB(255, 255, 255)
Line.Visible   = true

Line.Stroke.Enabled   = true
Line.Stroke.Color     = Color3.fromRGB(0, 0, 0)
Line.Stroke.Thickness = 1
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

#### Example 1 — Centered square with a gradient, stroke, and auto-rotation:

```luau
local Square = Drawing.new("Square")
Square.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Square.Size     = Vector2.new(160, 160)
Square.Color    = Color3.fromRGB(255, 120, 40)
Square.Filled   = true
Square.Visible  = true

Square.Gradient.Enabled  = true
Square.Gradient.Color    = ColorSequence.new(Color3.fromRGB(255, 60, 60), Color3.fromRGB(255, 220, 0))
Square.Gradient.Rotation = 45

Square.Stroke.Enabled   = true
Square.Stroke.Color     = Color3.fromRGB(255, 255, 255)
Square.Stroke.Thickness = 3

Square.AutoRotation.Enabled = true
Square.AutoRotation.Amount  = 60
Square.AutoRotation.Speed   = 1
```

#### Example 2 — Unfilled square (border only) with rounded corners in the top-right:

```luau
local Square = Drawing.new("Square")
Square.Position             = Vector2.new(820, 80)
Square.Size                 = Vector2.new(120, 120)
Square.Color                = Color3.fromRGB(80, 200, 255)
Square.Filled               = false
Square.Visible              = true
Square.Rounder.CornerRadius = UDim.new(0, 12)

Square.Stroke.Enabled   = true
Square.Stroke.Color     = Color3.fromRGB(80, 200, 255)
Square.Stroke.Thickness = 3
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

#### Example 1 — Centered circle with a spinning gradient and stroke:

```luau
local Circle = Drawing.new("Circle")
Circle.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Circle.Radius   = 90
Circle.Color    = Color3.fromRGB(40, 120, 255)
Circle.Filled   = true
Circle.Visible  = true

Circle.Gradient.Enabled  = true
Circle.Gradient.Color    = ColorSequence.new(Color3.fromRGB(0, 200, 255), Color3.fromRGB(180, 0, 255))
Circle.Gradient.Rotation = 0

Circle.Gradient.AutoRotation.Enabled = true
Circle.Gradient.AutoRotation.Amount  = 120
Circle.Gradient.AutoRotation.Speed   = 1

Circle.Stroke.Enabled   = true
Circle.Stroke.Color     = Color3.fromRGB(255, 255, 255)
Circle.Stroke.Thickness = 3
```

#### Example 2 — Unfilled circle used as a ring near the bottom-left:

```luau
local Circle = Drawing.new("Circle")
Circle.Position = Vector2.new(120, 500)
Circle.Radius   = 55
Circle.Color    = Color3.fromRGB(255, 80, 80)
Circle.Filled   = false
Circle.Visible  = true

Circle.Stroke.Enabled   = true
Circle.Stroke.Color     = Color3.fromRGB(255, 80, 80)
Circle.Stroke.Thickness = 4
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

#### Example 1 — Centered triangle with a gradient and spinning auto-rotation:

```luau
local Triangle = Drawing.new("Triangle")
Triangle.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Triangle.PointA   = Vector2.new(0, -90)
Triangle.PointB   = Vector2.new(78, 45)
Triangle.PointC   = Vector2.new(-78, 45)
Triangle.Color    = Color3.fromRGB(100, 220, 100)
Triangle.Filled   = true
Triangle.Visible  = true

Triangle.Gradient.Enabled  = true
Triangle.Gradient.Color    = ColorSequence.new(Color3.fromRGB(0, 255, 120), Color3.fromRGB(0, 80, 255))
Triangle.Gradient.Rotation = 90

Triangle.AutoRotation.Enabled = true
Triangle.AutoRotation.Amount  = 45
Triangle.AutoRotation.Speed   = 1
```

#### Example 2 — Small triangle in the bottom-right with an outline:

```luau
local Triangle = Drawing.new("Triangle")
Triangle.Position = Vector2.new(800, 500)
Triangle.PointA   = Vector2.new(0, -50)
Triangle.PointB   = Vector2.new(44, 25)
Triangle.PointC   = Vector2.new(-44, 25)
Triangle.Color    = Color3.fromRGB(255, 160, 0)
Triangle.Filled   = true
Triangle.Visible  = true

Triangle.Outlines.Visible   = true
Triangle.Outlines.Color     = Color3.fromRGB(255, 255, 255)
Triangle.Outlines.Thickness = 2
```

---

### Quad

#### Example 1 — Centered quad with a gradient and auto-rotation:

```luau
local Quad = Drawing.new("Quad")
Quad.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Quad.PointA   = Vector2.new(-90, -60)
Quad.PointB   = Vector2.new(90, -60)
Quad.PointC   = Vector2.new(90, 60)
Quad.PointD   = Vector2.new(-90, 60)
Quad.Color    = Color3.fromRGB(255, 160, 0)
Quad.Filled   = true
Quad.Visible  = true

Quad.Gradient.Enabled  = true
Quad.Gradient.Color    = ColorSequence.new(Color3.fromRGB(255, 100, 0), Color3.fromRGB(255, 220, 50))
Quad.Gradient.Rotation = 0

Quad.AutoRotation.Enabled = true
Quad.AutoRotation.Amount  = 30
Quad.AutoRotation.Speed   = 1
```

#### Example 2 — Skewed quad (non-rectangular) near the top-left with an outline:

```luau
local Quad = Drawing.new("Quad")
Quad.Position = Vector2.new(200, 150)
Quad.PointA   = Vector2.new(-70, -50)
Quad.PointB   = Vector2.new(100, -30)
Quad.PointC   = Vector2.new(70, 60)
Quad.PointD   = Vector2.new(-50, 40)
Quad.Color    = Color3.fromRGB(80, 180, 255)
Quad.Filled   = true
Quad.Visible  = true

Quad.Outlines.Visible   = true
Quad.Outlines.Color     = Color3.fromRGB(255, 255, 255)
Quad.Outlines.Thickness = 2
```

---

### Pentagon

#### Example 1 — Centered pentagon with a gradient and spinning auto-rotation:

```luau
local Pentagon = Drawing.new("Pentagon")
Pentagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Pentagon.Radius   = 90
Pentagon.Color    = Color3.fromRGB(180, 100, 255)
Pentagon.Filled   = true
Pentagon.Visible  = true

Pentagon.Gradient.Enabled  = true
Pentagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(180, 60, 255), Color3.fromRGB(255, 80, 160))
Pentagon.Gradient.Rotation = 45

Pentagon.AutoRotation.Enabled = true
Pentagon.AutoRotation.Amount  = 40
Pentagon.AutoRotation.Speed   = 1
```

#### Example 2 — Pentagon near the right side with visible outlines:

```luau
local Pentagon = Drawing.new("Pentagon")
Pentagon.Position = Vector2.new(750, 300)
Pentagon.Radius   = 70
Pentagon.Color    = Color3.fromRGB(255, 220, 60)
Pentagon.Filled   = true
Pentagon.Visible  = true

Pentagon.Outlines.Visible   = true
Pentagon.Outlines.Color     = Color3.fromRGB(200, 120, 0)
Pentagon.Outlines.Thickness = 3
```

---

### Hexagon

#### Example 1 — Centered hexagon with a gradient, stroke on outlines, and auto-rotation:

```luau
local Hexagon = Drawing.new("Hexagon")
Hexagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Hexagon.Radius   = 90
Hexagon.Color    = Color3.fromRGB(0, 200, 180)
Hexagon.Filled   = true
Hexagon.Visible  = true

Hexagon.Gradient.Enabled  = true
Hexagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(0, 220, 180), Color3.fromRGB(0, 100, 255))
Hexagon.Gradient.Rotation = 60

Hexagon.Outlines.Visible            = true
Hexagon.Outlines.Color              = Color3.fromRGB(255, 255, 255)
Hexagon.Outlines.Thickness          = 2
Hexagon.Outlines.Stroke.Enabled     = true
Hexagon.Outlines.Stroke.Color       = Color3.fromRGB(0, 0, 0)
Hexagon.Outlines.Stroke.Thickness   = 1

Hexagon.AutoRotation.Enabled = true
Hexagon.AutoRotation.Amount  = 30
Hexagon.AutoRotation.Speed   = 1
```

#### Example 2 — Small hexagon in the top-center, unfilled with colored outlines:

```luau
local Hexagon = Drawing.new("Hexagon")
Hexagon.Position = Vector2.new(500, 100)
Hexagon.Radius   = 50
Hexagon.Color    = Color3.fromRGB(0, 200, 180)
Hexagon.Filled   = false
Hexagon.Visible  = true

Hexagon.Outlines.Visible   = true
Hexagon.Outlines.Color     = Color3.fromRGB(0, 200, 180)
Hexagon.Outlines.Thickness = 3
```

---

### Heptagon

#### Example 1 — Centered heptagon with a gradient and auto-rotation:

```luau
local Heptagon = Drawing.new("Heptagon")
Heptagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Heptagon.Radius   = 90
Heptagon.Color    = Color3.fromRGB(255, 80, 160)
Heptagon.Filled   = true
Heptagon.Visible  = true

Heptagon.Gradient.Enabled  = true
Heptagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(255, 0, 120), Color3.fromRGB(255, 180, 0))
Heptagon.Gradient.Rotation = 0

Heptagon.AutoRotation.Enabled = true
Heptagon.AutoRotation.Amount  = 35
Heptagon.AutoRotation.Speed   = 1
```

#### Example 2 — Heptagon in the bottom-center with a thick outline:

```luau
local Heptagon = Drawing.new("Heptagon")
Heptagon.Position = Vector2.new(500, 520)
Heptagon.Radius   = 65
Heptagon.Color    = Color3.fromRGB(255, 100, 180)
Heptagon.Filled   = true
Heptagon.Visible  = true

Heptagon.Outlines.Visible   = true
Heptagon.Outlines.Color     = Color3.fromRGB(255, 255, 255)
Heptagon.Outlines.Thickness = 3
```

---

### Octagon

#### Example 1 — Centered octagon with a spinning gradient:

```luau
local Octagon = Drawing.new("Octagon")
Octagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Octagon.Radius   = 90
Octagon.Color    = Color3.fromRGB(80, 160, 255)
Octagon.Filled   = true
Octagon.Visible  = true

Octagon.Gradient.Enabled  = true
Octagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(0, 120, 255), Color3.fromRGB(80, 255, 200))
Octagon.Gradient.Rotation = 0

Octagon.Gradient.AutoRotation.Enabled = true
Octagon.Gradient.AutoRotation.Amount  = 90
Octagon.Gradient.AutoRotation.Speed   = 1
```

#### Example 2 — Small octagon on the left side with a gradient outline:

```luau
local Octagon = Drawing.new("Octagon")
Octagon.Position = Vector2.new(120, 300)
Octagon.Radius   = 55
Octagon.Color    = Color3.fromRGB(60, 140, 255)
Octagon.Filled   = true
Octagon.Visible  = true

Octagon.Outlines.Visible   = true
Octagon.Outlines.Color     = Color3.fromRGB(255, 255, 255)
Octagon.Outlines.Thickness = 2

Octagon.Outlines.Gradient.Enabled = true
Octagon.Outlines.Gradient.Color   = ColorSequence.new(Color3.fromRGB(0, 200, 255), Color3.fromRGB(0, 80, 200))
```

---

### Nonagon

#### Example 1 — Centered nonagon with a yellow gradient and auto-rotation:

```luau
local Nonagon = Drawing.new("Nonagon")
Nonagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Nonagon.Radius   = 90
Nonagon.Color    = Color3.fromRGB(255, 200, 50)
Nonagon.Filled   = true
Nonagon.Visible  = true

Nonagon.Gradient.Enabled  = true
Nonagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(255, 220, 0), Color3.fromRGB(255, 100, 0))
Nonagon.Gradient.Rotation = 90

Nonagon.AutoRotation.Enabled = true
Nonagon.AutoRotation.Amount  = 25
Nonagon.AutoRotation.Speed   = 1
```

#### Example 2 — Nonagon in the top-right corner with outlines only (unfilled):

```luau
local Nonagon = Drawing.new("Nonagon")
Nonagon.Position = Vector2.new(780, 120)
Nonagon.Radius   = 60
Nonagon.Color    = Color3.fromRGB(255, 200, 50)
Nonagon.Filled   = false
Nonagon.Visible  = true

Nonagon.Outlines.Visible   = true
Nonagon.Outlines.Color     = Color3.fromRGB(255, 200, 50)
Nonagon.Outlines.Thickness = 3
```

---

### Decagon

#### Example 1 — Centered decagon with a green gradient and spinning auto-rotation:

```luau
local Decagon = Drawing.new("Decagon")
Decagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Decagon.Radius   = 90
Decagon.Color    = Color3.fromRGB(50, 255, 120)
Decagon.Filled   = true
Decagon.Visible  = true

Decagon.Gradient.Enabled  = true
Decagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(0, 255, 100), Color3.fromRGB(0, 180, 80))
Decagon.Gradient.Rotation = 0

Decagon.AutoRotation.Enabled = true
Decagon.AutoRotation.Amount  = 20
Decagon.AutoRotation.Speed   = 1
```

#### Example 2 — Decagon near the center-left with thick white outlines:

```luau
local Decagon = Drawing.new("Decagon")
Decagon.Position = Vector2.new(250, 300)
Decagon.Radius   = 70
Decagon.Color    = Color3.fromRGB(30, 200, 90)
Decagon.Filled   = true
Decagon.Visible  = true

Decagon.Outlines.Visible   = true
Decagon.Outlines.Color     = Color3.fromRGB(255, 255, 255)
Decagon.Outlines.Thickness = 3
```

---

### Hendecagon

#### Example 1 — Centered hendecagon with an orange gradient:

```luau
local Hendecagon = Drawing.new("Hendecagon")
Hendecagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Hendecagon.Radius   = 90
Hendecagon.Color    = Color3.fromRGB(255, 100, 50)
Hendecagon.Filled   = true
Hendecagon.Visible  = true

Hendecagon.Gradient.Enabled  = true
Hendecagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(255, 80, 0), Color3.fromRGB(255, 200, 50))
Hendecagon.Gradient.Rotation = 45

Hendecagon.AutoRotation.Enabled = true
Hendecagon.AutoRotation.Amount  = 18
Hendecagon.AutoRotation.Speed   = 1
```

#### Example 2 — Hendecagon in the bottom-left with a gradient outline:

```luau
local Hendecagon = Drawing.new("Hendecagon")
Hendecagon.Position = Vector2.new(160, 480)
Hendecagon.Radius   = 65
Hendecagon.Color    = Color3.fromRGB(220, 80, 30)
Hendecagon.Filled   = true
Hendecagon.Visible  = true

Hendecagon.Outlines.Visible              = true
Hendecagon.Outlines.Thickness            = 2
Hendecagon.Outlines.Gradient.Enabled     = true
Hendecagon.Outlines.Gradient.Color       = ColorSequence.new(Color3.fromRGB(255, 180, 0), Color3.fromRGB(255, 60, 0))
Hendecagon.Outlines.Gradient.Rotation    = 90
```

---

### Dodecagon

#### Example 1 — Centered dodecagon with a teal gradient and auto-rotation:

```luau
local Dodecagon = Drawing.new("Dodecagon")
Dodecagon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
Dodecagon.Radius   = 90
Dodecagon.Color    = Color3.fromRGB(150, 255, 200)
Dodecagon.Filled   = true
Dodecagon.Visible  = true

Dodecagon.Gradient.Enabled  = true
Dodecagon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(100, 255, 200), Color3.fromRGB(0, 180, 140))
Dodecagon.Gradient.Rotation = 0

Dodecagon.AutoRotation.Enabled = true
Dodecagon.AutoRotation.Amount  = 15
Dodecagon.AutoRotation.Speed   = 1
```

#### Example 2 — Dodecagon near the right side with colored outlines:

```luau
local Dodecagon = Drawing.new("Dodecagon")
Dodecagon.Position = Vector2.new(750, 350)
Dodecagon.Radius   = 65
Dodecagon.Color    = Color3.fromRGB(100, 220, 180)
Dodecagon.Filled   = true
Dodecagon.Visible  = true

Dodecagon.Outlines.Visible   = true
Dodecagon.Outlines.Color     = Color3.fromRGB(0, 255, 180)
Dodecagon.Outlines.Thickness = 3
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

#### Example 1 — Centered 6-sided NGon with Radius, gradient, and auto-rotation:

```luau
local NGon = Drawing.new("NGon")
NGon.Points   = 6
NGon.Position = UDim2.new(0.5, 0, 0.5, 0) -- center of screen
NGon.Radius   = 90
NGon.Color    = Color3.fromRGB(200, 150, 255)
NGon.Filled   = true
NGon.Visible  = true

NGon.Gradient.Enabled  = true
NGon.Gradient.Color    = ColorSequence.new(Color3.fromRGB(180, 80, 255), Color3.fromRGB(80, 180, 255))
NGon.Gradient.Rotation = 45

NGon.AutoRotation.Enabled = true
NGon.AutoRotation.Amount  = 50
NGon.AutoRotation.Speed   = 1
```

#### Example 2 — 8-sided NGon with fully custom manual points near the bottom-right:

```luau
-- Arrow-like custom shape with 8 manual points
local NGon = Drawing.new("NGon")
NGon.Points   = 8
NGon.Position = Vector2.new(700, 450)

-- Arrow pointing right, built from 8 vertices
NGon.Point1 = Vector2.new(-80,  -20)
NGon.Point2 = Vector2.new(  0,  -20)
NGon.Point3 = Vector2.new(  0,  -50)
NGon.Point4 = Vector2.new( 80,    0)
NGon.Point5 = Vector2.new(  0,   50)
NGon.Point6 = Vector2.new(  0,   20)
NGon.Point7 = Vector2.new(-80,   20)
NGon.Point8 = Vector2.new(-80,  -20)

NGon.Color   = Color3.fromRGB(255, 220, 60)
NGon.Filled  = true
NGon.Visible = true

NGon.Outlines.Visible   = true
NGon.Outlines.Color     = Color3.fromRGB(200, 150, 0)
NGon.Outlines.Thickness = 2
```

#### Example 3 — 3-sided NGon (triangle) with manual star-like spike points:

```luau
local NGon = Drawing.new("NGon")
NGon.Points   = 3
NGon.Position = Vector2.new(300, 200)
NGon.Point1   = Vector2.new(0, -100)
NGon.Point2   = Vector2.new(86, 50)
NGon.Point3   = Vector2.new(-86, 50)
NGon.Color    = Color3.fromRGB(255, 80, 80)
NGon.Filled   = true
NGon.Visible  = true
```

---

## Target Types

Properties like `Position`, `From`, `To`, and `Size`, as well as functions like `:Trace()` and `Drawing.IsInShape()`, all accept a flexible target type that is automatically resolved to a `Vector2` screen position.

| Type       | Behaviour                                                                                         |
|------------|---------------------------------------------------------------------------------------------------|
| `Vector2`  | Used directly as a screen position                                                                |
| `UDim2`    | Converted to `Vector2` using the current viewport resolution at the time of assignment            |
| `Vector3`  | Projected via `Camera:WorldToViewportPoint` — returns `nil` if off-screen                        |
| `BasePart` | The closest visible surface point of the part is projected to screen                             |
| `Model`    | The closest descendant `BasePart` to screen center is used; final position comes from model pivot |

> **Important:** When a `UDim2` is assigned to any property, it is immediately converted to a `Vector2` and that is what gets stored. Reading the property back after assigning a `UDim2` will always return a `Vector2`.
