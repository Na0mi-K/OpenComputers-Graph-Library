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

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 7. Drawing Text
Text is drawn using character-cell coordinates, not Braille-dot coordinates.
```canvas:text(x, y, text, foreground, background)```
Coordinates are zero-based relative to the canvas.
Example:
```
canvas:text(
    2,
    1,
    "Power Dashboard",
    0xFFFFFF,
    0x202020
)
```
The foreground and background colors are optional:
```canvas:text(2, 2, "Temperature")```
If no foreground color is supplied, the canvas default foreground color is used. If no background color is supplied, the canvas background color is used.
The function can also write a string longer than one character cell:
```canvas:text(5, 5, "Energy: 1250 RF/t", 0x00FF00)```
The library treats this as a text run and sends it to the GPU in one operation where possible.
Text has priority over other canvas layers. If the same cell contains text, the text is rendered instead of a block or Braille character.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 8. Clearing the Canvas
# To clear all drawings:
```canvas:clear()```
You can also change the background color while clearing:
```canvas:clear(0x000000)```
This removes:
- Braille dots.
- Block fills.
- Text.
- The touched-cell tracking information.
After clearing, call:
```canvas:render()```
to update the physical screen.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 9. Rendering
Drawing operations modify the canvas’s internal data structures. They do not immediately draw everything to the screen.
To display the current canvas:
```canvas:render()```
This design allows several objects to be drawn before the GPU is updated.

Partial rendering
The canvas tracks which character cells have changed. During rendering, it updates only those cells instead of redrawing the entire canvas.

This is useful for performance, particularly when:
- Only a small line graph changes.
- A dashboard updates one value.
- A live chart receives a new data point.
- Most of the screen remains static.

Consecutive cells with matching colors are grouped into GPU calls when possible.

# GPU buffering
If supported by the GPU, the library attempts to allocate a VRAM buffer automatically. Buffering helps reduce flickering when rendering large or frequently updated displays.

You can explicitly disable buffering:
```
local canvas = Graph.newCanvas(gpu, {
    width = 80,
    height = 25,
    useBuffer = false
})
```
The buffer is released when the canvas is destroyed:
```canvas:destroy()```
You should call destroy() when the canvas will no longer be used.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 10. Creating a Chart
A chart is a specialized canvas with:
- A data coordinate system.
- Plotting functions.
- Data-series plotting.
- Bar charts.
- Axes and labels.
- Optional titles.
- Create one with:
```
local chart = Graph.newChart(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25,
    title = "Power Output"
})
```
By default, the chart reserves margins for labels:
```
left   = 6
bottom = 2
top    = 1
right  = 1
```
You can customize the margins:
```
local chart = Graph.newChart(gpu, {
    width = 80,
    height = 25,
    margin = {
        left = 8,
        bottom = 3,
        top = 2,
        right = 2
    }
})
```
The actual graph is drawn inside the plot area, while the margins are used for labels and titles.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 11. Setting the Chart Range
A chart needs to know which data values correspond to the visible plot area.
```chart:setRange(xmin, xmax, ymin, ymax)```
Example:
```chart:setRange(0, 100, -50, 50)```
This means:
- The x-axis displays values from 0 to 100.
- The y-axis displays values from -50 to 50.
 The chart automatically converts data coordinates into normalized canvas coordinates.
 The vertical direction is inverted automatically so that larger mathematical y-values appear higher on the screen.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------








<img width="1123" height="768" alt="Image" src="https://github.com/user-attachments/assets/e716535d-63ca-4ce8-8f5e-9ee2cbdb862b" />
