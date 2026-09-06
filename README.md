Graph Library for OpenComputers
A lightweight Lua graphics and charting library designed for OpenComputers. It provides three main features:

A canvas for drawing shapes and text.

A chart system for plotting functions, data series, and bar charts.

A rolling data buffer for live-updating graphs and dashboards.

The library uses Unicode Braille characters to render graphics at a higher resolution than ordinary terminal characters. It is designed to work with Lua 5.1, 5.2, and 5.3, including the Lua environments commonly used by OpenComputers.

# Installation
Save the library as: ```graph_lib.lua```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Then load it from another OpenComputers program:
```local Graph = require("graph_lib")```

You must also obtain a GPU proxy. A typical OpenComputers setup looks like this:
```local component = require("component") local gpu = component.gpu```

If you are using multiple screens, bind the GPU to the desired screen before creating a canvas:
```gpu.bind(screenAddress)```
The GPU passed to the library must already be bound to a screen.

# 1. Creating a Canvas
```local canvas = Graph.newCanvas(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25,
    background = 0x000000,
    foreground = 0xFFFFFF
})
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

```x```	Left screen coordinate in character cells	1

```y```	Top screen coordinate in character cells	1

```width```	Canvas width in character cells	Remaining screen width

```height```	Canvas height in character cells	Remaining screen height

```background```	Default background color	0x000000

```foreground```	Default drawing color	0xFFFFFF

```useBuffer```	Enables or disables GPU buffering	Automatic


# For Example : 

```
local canvas = Graph.newCanvas(gpu, {
    x = 2,
    y = 2,
    width = 60,
    height = 20,
    background = 0x101010,
    foreground = 0xFFFFFF,
    useBuffer = true
})
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The canvas internally represents each terminal character cell as a grid of 2 × 4 Braille dots. Therefore, a canvas of 60 × 20 character cells has an effective drawing resolution of: ```120 × 80 dots```

2. Coordinate Systems
The canvas supports two coordinate systems.

Normalized coordinates
Normalized coordinates range from 0 to 1:

```
(0, 0) = top-left of the canvas
(1, 1) = bottom-right of the canvas
```
This is the recommended coordinate system for general-purpose drawings because it automatically scales to different screen sizes.
ie: ```canvas:line(0, 0, 1, 1, 0xFF0000)```

# Raw dot coordinates
Raw coordinates address the individual Braille dots directly.
For a canvas of width W and height H in character cells:
```
raw width  = W × 2
raw height = H × 4
```
------
```canvas:lineRaw(0, 0, canvas.dotsW - 1, canvas.dotsH - 1, 0x00FF00)```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 3. Drawing Lines
Normalized line

```canvas:line(x0, y0, x1, y1, color)```
Example:
```
canvas:line(
    0.1, 0.1,
    0.9, 0.9,
    0xFF0000
)
```
This draws a red line between two normalized positions.
Raw line :

```canvas:lineRaw(x0, y0, x1, y1, color)```
Example:
```canvas:lineRaw(0, 0, 50, 20, 0x00FF00)```
The implementation uses Bresenham’s line algorithm, which produces efficient integer-coordinate lines without requiring floating-point calculations for every pixel.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 4. Drawing Rectangles
# Normalized rectangle :
```canvas:rect(x, y, width, height, color, filled)```
The position and size are normalized.
```
canvas:rect(
    0.2, 0.2,
    0.6, 0.4,
    0x0000FF,
    false
)
```
This draws a blue rectangle outline.
To draw a filled rectangle:
```
canvas:rect(
    0.2, 0.2,
    0.6, 0.4,
    0x0000FF,
    true
)
```
Raw rectangle
```canvas:rectRaw(x, y, width, height, color, filled)```
Example: ```canvas:rectRaw(10, 10, 40, 20, 0xFFFF00, true)```
The raw version uses exact Braille-dot coordinates.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 5. Drawing Circles
# Normalized circle
```canvas:circle(centerX, centerY, radius, color, filled)```
Example:
```
canvas:circle(
    0.5, 0.5,
    0.25,
    0xFF00FF,
    false
)
```
This draws a magenta circle centered in the canvas.
The radius is relative to the canvas width. A radius of 0.25 means approximately one quarter of the available horizontal drawing range.
For a filled circle:
```
canvas:circle(0.5, 0.5, 0.25, 0xFF00FF, true)
Raw circle
lua
canvas:circleRaw(centerX, centerY, radius, color, filled)
```
Example:
```
canvas:circleRaw(
    canvas.dotsW / 2,
    canvas.dotsH / 2,
    20,
    0x00FFFF,
    false
)
```
The circle implementation uses the midpoint circle algorithm.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 6. Filling Character Cells
Braille dots are useful for detailed lines, but they are not ideal for large solid areas. The library therefore supports whole-cell block fills.

lua
canvas:fillCellsRaw(x, y, width, height, color)
Example:
```canvas:fillCellsRaw(5, 5, 10, 8, 0x3366FF)```
This fills a rectangular area using the background color of each character cell.
This method is especially useful for:
- Bar charts.
- Progress bars.
- Solid panels.
- Colored dashboard areas.
- Large backgrounds.
- Unlike Braille drawing, cell filling changes the background color of each terminal cell.











<img width="1123" height="768" alt="Image" src="https://github.com/user-attachments/assets/e716535d-63ca-4ce8-8f5e-9ee2cbdb862b" />
