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










<img width="1123" height="768" alt="Image" src="https://github.com/user-attachments/assets/e716535d-63ca-4ce8-8f5e-9ee2cbdb862b" />
