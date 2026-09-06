# Graph Library for OpenComputers : a Program by NaomiK , FluoroPolymers™

A lightweight Lua graphics and charting library designed for OpenComputers. It provides three main features:
- A canvas for drawing shapes and text.
- A chart system for plotting functions, data series, and bar charts.
- A rolling data buffer for live-updating graphs and dashboards.

The library uses Unicode Braille characters to render graphics at a higher resolution than ordinary terminal characters. It is designed to work with Lua 5.1, 5.2, and 5.3, including the Lua environments commonly used by OpenComputers.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Preview :

<img width="1123" height="768" alt="Image" src="https://github.com/user-attachments/assets/e716535d-63ca-4ce8-8f5e-9ee2cbdb862b" />

# Installation :
Save the library as: ```graph_lib.lua```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Then load it from another OpenComputers program:
```local Graph = require("graph_lib")```
You must also obtain a GPU proxy. A typical OpenComputers setup looks like this:
```local component = require("component") local gpu = component.gpu```
If you are using multiple screens, bind the GPU to the desired screen before creating a canvas:
```gpu.bind(screenAddress)```
The GPU passed to the library must already be bound to a screen.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Tutorial :
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

# 12. Automatically Scaling a Chart
You can calculate a range from arrays of data:
```chart:autoscale(xs, ys)```
Example:
```
local xs = {1, 2, 3, 4, 5}
local ys = {10, 14, 12, 19, 25}
chart:autoscale(xs, ys)
```
The function finds the minimum and maximum values and adds a small margin around them.
The default margin is 5 percent:
```chart:autoscale(xs, ys, 0.10)```
The third argument sets the margin to 10 percent.
If all x-values or all y-values are identical, the library expands that range automatically so the data remains visible.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 13. Plotting a Mathematical Function
Use:
```
chart:plotFunction(
    function,
    xmin,
    xmax,
    samples,
    color,
    options
)
```
Example:
```
chart:setRange(-math.pi, math.pi, -1.2, 1.2)

chart:plotFunction(
    math.sin,
    -math.pi,
    math.pi,
    300,
    0x00FF00
)
This plots 
𝑦
=
sin
⁡
(
𝑥
)
y=sin(x).
```
The function is sampled repeatedly between xmin and xmax. The resulting points are connected by line segments.
# Options :
You can display sample points:
```
chart:plotFunction(
    function(x)
        return x * x
    end,
    -2,
    2,
    200,
    0xFF0000,
    {
        points = true,
        pointRadius = 0.01
    }
)
```
Available options include:
- Option	Description
- points	Draws markers in addition to connecting lines
- pointsOnly	Draws markers without connecting lines
- pointRadius	Sets the normalized radius of each marker
- The function is called with pcall, so errors do not crash the plotting operation. Invalid values and NaN values break the line, which is useful for discontinuous functions.
For example:
```
chart:plotFunction(
    function(x)
        return 1 / x
    end,
    -5,
    5,
    300,
    0xFFFF00
)
```
The graph will not connect across the discontinuity at  $x = 0$

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 14. Plotting a Data Series
Use:
```chart:plotSeries(xs, ys, color, options)```
Example:
```
local xs = {0, 1, 2, 3, 4}
local ys = {2, 5, 3, 8, 6}

chart:setRange(0, 4, 0, 10)
chart:plotSeries(xs, ys, 0x00FFFF)
```
The values in ys are plotted against the corresponding values in xs.
If xs is nil, the array index is used as the x-coordinate:
```
local values = {4, 8, 6, 10, 12}

chart:setRange(1, 5, 0, 15)
chart:plotSeries(nil, values, 0x00FF00)
```
This is equivalent to plotting:
```(1, 4), (2, 8), (3, 6), (4, 10), (5, 12)```
You can add point markers:
```
chart:plotSeries(
    xs,
    ys,
    0xFF0000,
    {
        points = true,
        pointRadius = 0.008
    }
)
```
To display only markers:
```
chart:plotSeries(
    xs,
    ys,
    0xFF0000,
    {
        pointsOnly = true
    }
)
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 15. Plotting Bar Charts
```chart:plotBars(values, color, options)```
Example:
```
local values = {10, 25, 15, 30, 20}
chart:setRange(0, 5, 0, 35)
chart:plotBars(values, 0x3366FF)
```
The bars are evenly spaced across the plot area.
- The library determines:
- The width of each bar.
- The vertical position of zero.
- The top and bottom of each bar.
Whether a bar extends above or below zero.
Negative values are supported:
```
local values = {-10, 15, -5, 20, 8}
chart:setRange(0, 5, -15, 25)
chart:plotBars(values, 0xFF8800)
```
Bars use whole-cell background fills rather than individual Braille dots. This makes them more visually solid and generally more efficient to render.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 16. Drawing Axes and Labels
```chart:drawAxes()```
- This draws:
- The horizontal axis.
- The vertical axis.
- Y-axis labels.
- X-axis labels.
- The chart title, if one was configured.
Example:
```
chart:drawAxes({
    color = 0x888888,
    yTicks = 5,
    xTicks = 4,
    titleColor = 0xFFFFFF
})
```
Axis options :
- Option	Description	Default
- color	Axis line color	0x888888
- yTicks	Number of intervals on the y-axis	5
- xTicks	Number of intervals on the x-axis	3
- titleColor	Title color	0xFFFFFF
The axes are positioned at zero when zero lies within the current data range. Otherwise, they are placed at the relevant plot boundary.
For example, if the y-range is:
```-10 to 10```
the x-axis appears in the middle of the graph. If the y-range is:
```10 to 100```
zero is outside the range, so the x-axis is drawn at the lower edge.
After drawing all chart content, render it:
```chart:render()```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 17. RollingSeries
RollingSeries is a fixed-size circular buffer intended for live graphs.
Create one with:
```local history = Graph.newRollingSeries(60)```
This buffer stores at most 60 values. Adding values :
```
history:push(1250)
history:push(1275)
history:push(1300)
```
When the buffer is full, adding a new value overwrites the oldest one. Retrieving values :
```local values = history:values()```
The returned table is ordered from oldest to newest, making it directly compatible with plotSeries():
```chart:plotSeries(nil, history:values(), 0x00FF00)```
This makes it suitable for:
- Energy history.
- Power output.
- Temperature monitoring.
- Machine throughput.
- Computer performance graphs.
- Live OpenComputers dashboards.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 18. Live Graph Example
The following example creates a live-updating graph:
```
local component = require("component")
local event = require("event")
local math = math
local Graph = require("graph_lib")

local gpu = component.gpu

local chart = Graph.newChart(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25,
    background = 0x000000,
    foreground = 0xFFFFFF,
    title = "Live Power Output"
})

local history = Graph.newRollingSeries(60)

while true do
    local value = 50 + 30 * math.sin(os.clock()) + math.random() * 5
    history:push(value)

    chart:clear(0x000000)
    chart:setRange(1, 60, 0, 100)
    chart:plotSeries(nil, history:values(), 0x00FF00, {
        points = true,
        pointRadius = 0.006
    })
    chart:drawAxes({
        color = 0x555555,
        yTicks = 5,
        xTicks = 3
    })
    chart:render()

    local _, _, reason = event.pull(1, "interrupted")
    if reason == "interrupted" then
        break
    end
end

chart.canvas:destroy()
```
The important detail is the use of: ```event.pull(1, "interrupted")```
instead of ```os.sleep().```
This allows the program to wait while continuing to respond to interruption events and other OpenComputers event handling.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 19. Combining Multiple Graphs
You can create several canvases or charts on the same screen.
```
local topChart = Graph.newChart(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 12,
    title = "Power"
})
local bottomChart = Graph.newChart(gpu, {
    x = 1,
    y = 14,
    width = 80,
    height = 12,
    title = "Temperature"
})
```
Each chart has its own region and can be rendered independently.
You can also use a regular canvas for dashboard labels:
```
local dashboard = Graph.newCanvas(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25
})

dashboard:text(2, 2, "System Status", 0xFFFFFF)
dashboard:text(2, 3, "Online", 0x00FF00)
dashboard:render()
```
When several objects overlap, the object rendered last will generally overwrite the same screen cells.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 20. Rendering Priority
When multiple drawing layers occupy the same cell, the library uses this priority:
- Text
- Whole-cell block fill.
- Braille dots.
- Empty background.
In other words, text is rendered over a block, and a block is rendered over Braille dots.
This is important when combining charts with labels. A label may cover part of a graph if both use the same cells.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 21. Color Format
Colors are specified as hexadecimal RGB values: ```0xRRGGBB```
Examples:
```
0xFF0000 -- red
0x00FF00 -- green
0x0000FF -- blue
0xFFFFFF -- white
0x000000 -- black
0xFFFF00 -- yellow
0x00FFFF -- cyan
0xFF00FF -- magenta
```
The library normalizes color values into the range from ```0x000000``` to ```0xFFFFFF```
It intentionally avoids Lua 5.3-only bitwise operators such as:
- &
- |
- ~
- <<
- >>
It also avoids integer floor division: ```//```
This keeps the source compatible with older Lua runtimes used by OpenComputers.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 22. Important Implementation Details
# Sparse storage
The canvas stores only cells that contain content or have been modified.
It maintains separate layers for:
```
cellDots
cellColor
cellBlock
cellText
touched
```
This avoids allocating a complete large screen-sized structure when only a small part of the canvas is being used.
# Braille rendering
Each Braille character represents eight possible dots. The dot layout is:
```
(0,0) (1,0)   weights 0x01, 0x08
(0,1) (1,1)   weights 0x02, 0x10
(0,2) (1,2)   weights 0x04, 0x20
(0,3) (1,3)   weights 0x40, 0x80
```
The final Unicode code point is:```0x2800 + sum of active dot weights```
The library stores each dot as a Boolean instead of using bitwise operators. This is deliberate because some OpenComputers Lua environments use Lua 5.2, where Lua 5.3 bitwise syntax is unavailable.
# Dirty-cell rendering
When a drawing operation modifies a cell, that cell is added to the touched set.
During rendering, only touched cells are considered. This significantly reduces GPU activity when updating small portions of a display.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 23. Complete Basic Example
```
local component = require("component")
local Graph = require("graph_lib")

local gpu = component.gpu

local canvas = Graph.newCanvas(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25,
    background = 0x101010,
    foreground = 0xFFFFFF
})

canvas:clear(0x101010)

canvas:text(2, 1, "Graph Library Demo", 0xFFFFFF)

canvas:line(0.05, 0.15, 0.95, 0.15, 0xFF0000)

canvas:rect(
    0.10, 0.25,
    0.30, 0.30,
    0x00FF00,
    false
)

canvas:rect(
    0.55, 0.25,
    0.30, 0.30,
    0x0000FF,
    true
)

canvas:circle(
    0.50, 0.75,
    0.15,
    0xFFFF00,
    false
)

canvas:render()
```
This example displays:
- A title.
- A red line.
- A green rectangle outline.
- A filled blue rectangle.
- A yellow circle.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 24. Complete Chart Example
```
local component = require("component")
local Graph = require("graph_lib")
local gpu = component.gpu
local chart = Graph.newChart(gpu, {
    x = 1,
    y = 1,
    width = 80,
    height = 25,
    background = 0x000000,
    foreground = 0xFFFFFF,
    title = "Sine Function"
})

chart:clear(0x000000)

chart:setRange(
    -math.pi,
    math.pi,
    -1.2,
    1.2
)

chart:plotFunction(
    math.sin,
    -math.pi,
    math.pi,
    300,
    0x00FF00,
    {
        points = false
    }
)

chart:drawAxes({
    color = 0x666666,
    yTicks = 4,
    xTicks = 4
})

chart:render()
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 25. API Reference
# Main constructors
```
Graph.newCanvas(gpu, opts)
Graph.newChart(gpu, opts)
Graph.newRollingSeries(maxPoints)
```
# Canvas methods
```
canvas:clear(background)
canvas:setDot(x, y, color)
canvas:clearDot(x, y)
canvas:lineRaw(x0, y0, x1, y1, color)
canvas:rectRaw(x, y, width, height, color, filled)
canvas:circleRaw(cx, cy, radius, color, filled)
canvas:line(x0, y0, x1, y1, color)
canvas:rect(x, y, width, height, color, filled)
canvas:circle(cx, cy, radius, color, filled)
canvas:fillCellsRaw(x, y, width, height, color)
canvas:text(x, y, text, foreground, background)
canvas:render()
canvas:destroy()
```
# Chart methods
```
chart:setRange(xmin, xmax, ymin, ymax)
chart:autoscale(xs, ys, marginPercent)
chart:clear(background)
chart:plotFunction(f, xmin, xmax, samples, color, opts)
chart:plotSeries(xs, ys, color, opts)
chart:plotBars(values, color, opts)
chart:drawAxes(opts)
chart:render()
RollingSeries methods
lua
series:push(value)
series:values()
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# 26. Limitations and Considerations
- The canvas dimensions are measured in terminal character cells, not pixels.
- Braille rendering requires a screen/font that supports Unicode Braille characters.
- Text coordinates are character-based, while drawing coordinates are normalized or raw dot-based.
- A text string can occupy multiple cells, so long text may overwrite neighboring content.
- plotBars() uses whole-cell fills, so its resolution is lower than a Braille-based graph.
- Charts should be cleared and redrawn before each live update unless you intentionally want to preserve old content.
- The chart’s y-axis label positioning assumes the default chart layout and may require adjustment for unusual margins.
- GPU buffering depends on hardware and OpenComputers version support.
- The graph library performs no automatic legend management or multiple-series styling; those must be implemented by the caller.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Summary
- This library provides a compact graphics layer for OpenComputers:
- Use Graph.newCanvas() for general drawings, diagrams, UI elements, and shapes.
- Use normalized coordinates when you want layouts to scale between screen sizes.
- Use raw coordinates when you need precise Braille-dot control.
- Use Graph.newChart() for mathematical functions and data visualization.
- Use Graph.newRollingSeries() for live-updating graphs and dashboards.
- Call render() after drawing.
 -Call destroy() when a canvas is no longer needed.
