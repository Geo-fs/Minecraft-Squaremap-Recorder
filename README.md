# 🗺️ Squaremap Structure Tools

Two companion tools for Minecraft explorers who want **full control over structure locations** without needing server admin privileges.  

1. **Squaremap Multi-Structure Markers** – a Tampermonkey userscript that adds custom markers directly onto your server’s Squaremap/LiveAtlas map.  
2. **Structure Plotter** – a standalone offline HTML app for plotting and labeling structures on a clean 2D grid map.

---

## 🚀 Squaremap Multi-Structure Markers (Userscript)

### Features
- Works directly on your server’s **Squaremap/LiveAtlas** map.
- Reads coordinates from the URL hash:  
#world;map;X,Y,Z;zoom
- Add markers with hotkey **M** (from the map center).
- Choose from **12 preset structure types** (monument, village, outpost, bastion, etc.) or define a custom type.
- Optional **labels** (e.g., `“west of spawn”`).
- Export/Import marker lists as JSON.
- Copy coordinates in multiple formats:
- `X Z type`
- CSV: `x,z,type,label`
- JSON: `[{"x":123,"z":456,"type":"village","label":"spawn"}]`

----

### Installation
1. Install [Tampermonkey](https://www.tampermonkey.net/) in your browser.
2. Create a **new userscript**.
3. Paste the contents of [`squaremap-recorder`](https://github.com/Geo-fs/Minecraft-Squaremap-Recorder/blob/main/Squaremap%20Recorder%20Addon).
4. Save and refresh your Squaremap page.

You’ll see a new **marker panel** in the top-left corner of the map.

### Usage
- Pan/zoom the map until the target structure is centered.
- Pick a structure type and (optionally) enter a label.
- Press **Add from center** or hit **M**.
- Repeat for all structures you want to catalog.
- Use **Export JSON** or **Copy Coords** to save results.

**Example JSON output:**
```json
[
{ "x": 19756, "z": -1645, "type": "monument", "label": "South Ocean" },
{ "x": -928, "z": 240, "type": "outpost", "label": "Near swamp" }
]
```
🖥️ Structure Plotter (Offline HTML App)
Features
Single-file HTML app – no server required.

Accepts multiple input formats:

X Z

type: X Z

[type] X,Y,Z

JSON: [{"x":123,"z":456,"type":"outpost","label":"swamp"}]

Auto-detects common structure names from labels.

Assigns colors and shapes per structure type (customizable).

---

### Map options:

Grid lines (100 blocks)

Chunk grid (16)

Region grid (512)

World axes (0/0)

Labels (coords + type, or custom labels)

Snap markers to chunk centers

Export as PNG or save JSON.

Smooth pan/zoom controls:

Drag to pan

Scroll to zoom

Double-click to fit view

---

### Hotkeys:

F → fit all markers

0 → reset zoom

L → toggle labels

### Installation:

Download [`UI Bonus Map`](https://github.com/Geo-fs/Minecraft-Squaremap-Recorder/blob/main/UI%20Bonus%20Map).

Open it in your browser (works completely offline).

---

### Usage
Paste coordinates into the input box. Example:
```json
monument: 19756 -1645
[village] 832, 70, -304
outpost: -928 240
{"x": -1200, "z": 512, "type": "stronghold", "label": "eye trail"}
```

# Click Parse & Plot.

Adjust zoom, toggle grids/labels, and customize type colors/shapes.

## Export the finished map as PNG or JSON.
---
### 🔄 Workflow
Use the Userscript while browsing your server’s Squaremap.

Export the marker list as JSON.

Load that JSON into the HTML Plotter.

Get a clean, styled map with gridlines, labels, and export options.

---

### ⚙️ Requirements
## A modern browser (tested in Chrome, Firefox, Edge).

## Tampermonkey extension (for the userscript).

*No server-side access required – both tools run client-side.*
