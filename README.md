Graph Library for OpenComputers
A lightweight Lua graphics and charting library designed for OpenComputers. It provides three main features:

A canvas for drawing shapes and text.

A chart system for plotting functions, data series, and bar charts.

A rolling data buffer for live-updating graphs and dashboards.

The library uses Unicode Braille characters to render graphics at a higher resolution than ordinary terminal characters. It is designed to work with Lua 5.1, 5.2, and 5.3, including the Lua environments commonly used by OpenComputers.

# Installation
Save the library as: ```graph_lib.lua```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

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

```x```	Left screen coordinate in character cells	1
```y```	Top screen coordinate in character cells	1
```width```	Canvas width in character cells	Remaining screen width
```height```	Canvas height in character cells	Remaining screen height
```background```	Default background color	0x000000
```foreground```	Default drawing color	0xFFFFFF
```useBuffer```	Enables or disables GPU buffering	Automatic










<img width="1123" height="768" alt="Image" src="https://github.com/user-attachments/assets/e716535d-63ca-4ce8-8f5e-9ee2cbdb862b" />
