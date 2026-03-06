---
name: json-canvas
description: Create and edit JSON Canvas files (.canvas) with nodes, edges, groups, and connections. Use when working with .canvas files, creating visual canvases, mind maps, flowcharts, or when the user mentions Canvas files in Obsidian.
---

# JSON Canvas Skill

## Workflow

Follow these steps when creating or editing a canvas file:

1. **Create file** — use the `.canvas` extension (e.g. `my-canvas.canvas`)
2. **Scaffold structure** — start with the two top-level arrays: `{ "nodes": [], "edges": [] }`
3. **Add nodes** — assign each node a unique 16-char hex `id`, required position/size fields, and type-specific fields
4. **Add edges** — reference valid `fromNode` and `toNode` IDs from the nodes array
5. **Validate JSON** — verify syntax is valid JSON and all cross-references resolve
6. **Open in Obsidian** — confirm the canvas renders as expected

## File Structure

A canvas file contains two top-level arrays:

```json
{
  "nodes": [],
  "edges": []
}
```

- `nodes` (optional): Array of node objects
- `edges` (optional): Array of edge objects connecting nodes

## Nodes

Nodes are objects placed on the canvas. There are four node types:
- `text` - Text content with Markdown
- `file` - Reference to files/attachments
- `link` - External URL
- `group` - Visual container for other nodes

### Z-Index Ordering

Nodes are ordered by z-index in the array:
- First node = bottom layer (displayed below others)
- Last node = top layer (displayed above others)

### Generic Node Attributes

All nodes share these attributes:

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `id` | Yes | string | Unique identifier for the node |
| `type` | Yes | string | Node type: `text`, `file`, `link`, or `group` |
| `x` | Yes | integer | X position in pixels |
| `y` | Yes | integer | Y position in pixels |
| `width` | Yes | integer | Width in pixels |
| `height` | Yes | integer | Height in pixels |
| `color` | No | canvasColor | Node color (see Color section) |

### Text Nodes

Text nodes contain Markdown content.

```json
{
  "id": "6f0ad84f44ce9c17",
  "type": "text",
  "x": 0,
  "y": 0,
  "width": 400,
  "height": 200,
  "text": "# Hello World\n\nThis is **Markdown** content."
}
```

#### Newline Escaping (Common Pitfall)

In JSON, newline characters inside strings **must** be represented as `\n`. Do **not** use the literal sequence `\\n` in a `.canvas` file—Obsidian will render it as the characters `\` and `n` instead of a line break.

```json
{ "type": "text", "text": "Line 1\nLine 2" }
```

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `text` | Yes | string | Plain text with Markdown syntax |

### File Nodes

File nodes reference files or attachments (images, videos, PDFs, notes, etc.).

```json
{
  "id": "a1b2c3d4e5f67890",
  "type": "file",
  "x": 500,
  "y": 0,
  "width": 400,
  "height": 300,
  "file": "Notes/Project Overview.md",
  "subpath": "#Implementation"
}
```

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `file` | Yes | string | Path to file within the system |
| `subpath` | No | string | Link to heading or block (starts with `#`) |

### Link Nodes

Link nodes display external URLs.

```json
{
  "id": "c3d4e5f678901234",
  "type": "link",
  "x": 1000,
  "y": 0,
  "width": 400,
  "height": 200,
  "url": "https://obsidian.md"
}
```

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `url` | Yes | string | External URL |

### Group Nodes

Group nodes are visual containers for organizing other nodes.

```json
{
  "id": "d4e5f6789012345a",
  "type": "group",
  "x": -50,
  "y": -50,
  "width": 1000,
  "height": 600,
  "label": "Project Overview",
  "background": "Attachments/background.png",
  "backgroundStyle": "cover"
}
```

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `label` | No | string | Text label for the group |
| `background` | No | string | Path to background image |
| `backgroundStyle` | No | string | Background rendering style: `cover`, `ratio`, or `repeat` |

#### Background Styles

| Value | Description |
|-------|-------------|
| `cover` | Fills entire width and height of node |
| `ratio` | Maintains aspect ratio of background image |
| `repeat` | Repeats image as pattern in both directions |

## Edges

Edges are lines connecting nodes.

```json
{
  "id": "0123456789abcdef",
  "fromNode": "6f0ad84f44ce9c17",
  "fromSide": "right",
  "fromEnd": "none",
  "toNode": "b2c3d4e5f6789012",
  "toSide": "left",
  "toEnd": "arrow",
  "color": "1",
  "label": "leads to"
}
```

| Attribute | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `id` | Yes | string | - | Unique identifier for the edge |
| `fromNode` | Yes | string | - | Node ID where connection starts |
| `fromSide` | No | string | - | Side where edge starts: `top`, `right`, `bottom`, `left` |
| `fromEnd` | No | string | `none` | Shape at edge start: `none` or `arrow` |
| `toNode` | Yes | string | - | Node ID where connection ends |
| `toSide` | No | string | - | Side where edge ends: `top`, `right`, `bottom`, `left` |
| `toEnd` | No | string | `arrow` | Shape at edge end: `none` or `arrow` |
| `color` | No | canvasColor | - | Line color |
| `label` | No | string | - | Text label for the edge |

## Colors

The `canvasColor` type accepts either a hex color or a preset string:

| Preset | Color |
|--------|-------|
| `"1"` | Red |
| `"2"` | Orange |
| `"3"` | Yellow |
| `"4"` | Green |
| `"5"` | Cyan |
| `"6"` | Purple |

Hex colors use the `#RRGGBB` format, e.g. `"#FF0000"`. Preset color values are intentionally undefined by the spec, allowing applications to use their own brand colors.

## Complete Example

### Simple Mind Map

```json
{
  "nodes": [
    {
      "id": "8a9b0c1d2e3f4a5b",
      "type": "text",
      "x": 0,
      "y": 0,
      "width": 300,
      "height": 150,
      "text": "# Main Idea\n\nThis is the central concept."
    },
    {
      "id": "1a2b3c4d5e6f7a8b",
      "type": "text",
      "x": 400,
      "y": -100,
      "width": 250,
      "height": 100,
      "text": "## Supporting Point A\n\nDetails here."
    },
    {
      "id": "2b3c4d5e6f7a8b9c",
      "type": "text",
      "x": 400,
      "y": 100,
      "width": 250,
      "height": 100,
      "text": "## Supporting Point B\n\nMore details."
    }
  ],
  "edges": [
    {
      "id": "3c4d5e6f7a8b9c0d",
      "fromNode": "8a9b0c1d2e3f4a5b",
      "fromSide": "right",
      "toNode": "1a2b3c4d5e6f7a8b",
      "toSide": "left"
    },
    {
      "id": "4d5e6f7a8b9c0d1e",
      "fromNode": "8a9b0c1d2e3f4a5b",
      "fromSide": "right",
      "toNode": "2b3c4d5e6f7a8b9c",
      "toSide": "left"
    }
  ]
}
```

For additional complete examples (project boards, flowcharts, research canvases), see `EXAMPLES.md`.

## ID Generation

Node and edge IDs must be unique strings. Use 16-character lowercase hexadecimal IDs (64-bit random value):

```
"id": "6f0ad84f44ce9c17"
"id": "1234567890abcdef"
```

## Layout Guidelines

### Positioning

- Coordinates can be negative (canvas extends infinitely)
- `x` increases to the right, `y` increases downward
- Position refers to top-left corner of node

### Recommended Sizes

| Node Type | Suggested Width | Suggested Height |
|-----------|-----------------|------------------|
| Small text | 200-300 | 80-150 |
| Medium text | 300-450 | 150-300 |
| Large text | 400-600 | 300-500 |
| File preview | 300-500 | 200-400 |
| Link preview | 250-400 | 100-200 |
| Group | Varies | Varies |

### Spacing

- Leave 20-50px padding inside groups
- Space nodes 50-100px apart for readability
- Align nodes to grid (multiples of 10 or 20) for cleaner layouts

## Validation Checklist

Run through these checks before saving a canvas file:

- [ ] All `id` values are unique across nodes and edges
- [ ] `fromNode` and `toNode` reference existing node IDs
- [ ] Every node has the required generic attributes (`id`, `type`, `x`, `y`, `width`, `height`)
- [ ] Every edge has `id`, `fromNode`, and `toNode`
- [ ] Type-specific required fields are present (`text` for text nodes, `file` for file nodes, `url` for link nodes)
- [ ] `type` is one of: `text`, `file`, `link`, `group`
- [ ] `backgroundStyle` (if used) is one of: `cover`, `ratio`, `repeat`
- [ ] `fromSide`/`toSide` (if used) are one of: `top`, `right`, `bottom`, `left`
- [ ] `fromEnd`/`toEnd` (if used) are one of: `none`, `arrow`
- [ ] Color presets (if used) are `"1"`–`"6"` or valid hex string
- [ ] Newlines in text strings use `\n`, not `\\n`
- [ ] Overall file is valid JSON

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Edge not visible | `fromNode`/`toNode` ID doesn't match any node | Verify IDs match exactly |
| Node missing | Required field absent | Add missing `id`, `x`, `y`, `width`, `height`, or `type` |
| Text shows `\n` literally | Used `\\n` instead of `\n` | Replace `\\n` with `\n` in text strings |
| File node blank | Wrong file path | Check path is relative to vault root |
| Canvas won't open | Malformed JSON | Validate JSON syntax (missing comma, bracket, etc.) |

## References

- [JSON Canvas Spec 1.0](https://jsoncanvas.org/spec/1.0/)
- [JSON Canvas GitHub](https://github.com/obsidianmd/jsoncanvas)
