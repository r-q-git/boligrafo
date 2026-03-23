## Boligrafo – Freehand SVG Drawing Engine

Boligrafo is a powerful **freehand drawing application** built with Angular that enables smooth, pressure-sensitive strokes rendered as SVG paths. It supports multiple tools, real-time stroke preview, SVG import/export, and seamless integration with external applications via iframe messaging.

---

## 🚀 Features

### 🖊️ Drawing Engine

- Smooth freehand drawing powered by `perfect-freehand`
- Pressure-sensitive strokes
- Real-time preview rendering
- SVG-based vector output

### 🎨 Tools

- **Pen 1** – stylized smooth pen
- **Pen 2** – alternate stroke feel
- **Highlighter** – semi-transparent strokes
- **Eraser** – proximity-based stroke removal

### ⚙️ Customization

- Stroke size, thinning, smoothing, streamline
- Start/end taper controls
- Easing functions (custom curve behavior)
- Fill color & stroke outline

### 📂 File Support

- Import SVG → converts into editable strokes
- Export SVG → download drawings
- Drag & drop support

### 🔄 History

- Undo / Redo support

### 🔗 Integration

- Send drawing as Base64 SVG to parent app
- Receive SVG from parent via iframe messaging

---

## 🏗️ Tech Stack

- **Framework:** Angular
- **UI:** PrimeNG
- **Drawing Engine:** perfect-freehand
- **Rendering:** SVG
- **Language:** TypeScript

---

## 📁 Project Structure

```
src/app
├── drawing/           # Core drawing logic + UI
├── models/            # Type definitions
├── services/          # Iframe communication
├── utils/             # Stroke, SVG, easing utilities
├── app.component      # Root component
└── app.module.ts      # Module configuration
```

---

## 🧠 Core Concepts

### 1. Stroke System

Each stroke is stored as:

```ts
interface DrawingStroke {
  points: number[][];
  path: string;
  color: string;
  opacity: number;
  outlineColor: string;
  outlineWidth: number;
}
```

- `points` → raw pointer data
- `path` → generated SVG path
- Used for rendering + erasing logic

---

### 2. Drawing Flow

```
Pointer Down → Start stroke
Pointer Move → Collect points → Generate stroke → Preview
Pointer Up   → Finalize stroke → Store in state
```

---

### 3. Stroke Generation

Pipeline:

```
Raw Points → perfect-freehand → Stroke Points → SVG Path
```

Handled via:

- `getStroke()` → generates smooth stroke
- `getSvgPathFromStroke()` → converts to SVG path

---

### 4. Tool System

Each tool defines:

```ts
{
  (size, thinning, smoothing, streamline, easing, start, end, outline);
}
```

Generated using:

- `getDefaultToolPen1`
- `getDefaultToolPen2`
- `getDefaultToolHighlighter`

---

### 5. Eraser Logic

- Checks distance between cursor and stroke points
- Removes stroke if within threshold

```ts
Math.hypot(p[0] - x, p[1] - y) < eraser.size;
```

---

### 6. SVG Import/Export

#### Export:

- Serialize SVG
- Convert to Base64
- Download or send to parent

#### Import:

- Parse `<path>` elements
- Extract `d` attribute
- Reconstruct points (approximation)

---

### 7. Iframe Communication

#### Send to Parent:

```ts
window.parent.postMessage(...)
```

Payload:

```ts
{
  type: ADD_OBJECT,
  payload: {
    dataString: base64SVG,
    type: 'stickerbox',
    metaData
  }
}
```

#### Receive from Parent:

- Listens to `window.message`
- Imports Base64 SVG

---

## 🖥️ UI Overview

### Top Toolbar

- Tool selection
- Undo / Redo
- Upload / Download
- Clear canvas
- Send to project

### Side Panel

- Tool customization
- Sliders + easing selection
- Color pickers

### Canvas

- Fullscreen SVG
- Pointer-based drawing

---

## ⚡ Getting Started

### Install Dependencies

```bash
npm install
```

### Run Development Server

```bash
ng serve
```

### Build

```bash
ng build
```

---

## 📌 Future Improvements

- Layer system
- Selection & transform tools
- Lasso tool
- Stroke editing (control points)
- Pressure simulation for mouse
- Performance optimization for large drawings

---

## 🧩 Key Utilities

| Utility                | Purpose                 |
| ---------------------- | ----------------------- |
| `getStroke`            | Stroke generation       |
| `getSvgPathFromStroke` | SVG path conversion     |
| `parseSVGFile`         | Import SVG              |
| `saveSVG`              | Export SVG              |
| `captureSvgAsBase64`   | Convert to Base64       |
| `EASINGS`              | Stroke animation curves |

---

## 📜 License

MIT License

---

# 🏗️ Architecture Diagram

Here’s a clean conceptual architecture of your system:

```
                        ┌──────────────────────────┐
                        │     Parent Application   │
                        │ (Iframe Host / Editor)   │
                        └──────────┬───────────────┘
                                   │ postMessage
                                   ▼
                    ┌──────────────────────────────┐
                    │   QlIframeMessageService     │
                    └──────────┬───────────────────┘
                               │
                               ▼
                    ┌──────────────────────────────┐
                    │      DrawingComponent        │
                    │  (Core Controller Layer)     │
                    └──────────┬───────────────────┘
                               │
        ┌──────────────────────┼────────────────────────┐
        ▼                      ▼                        ▼
┌───────────────┐   ┌───────────────────┐   ┌────────────────────┐
│ Pointer Input │   │ Stroke Engine     │   │ Tool Configuration │
│ (Mouse/Touch) │   │ (perfect-freehand)│   │ (Pen, Highlighter) │
└──────┬────────┘   └────────┬──────────┘   └────────┬───────────┘
       │                     │                       │
       ▼                     ▼                       ▼
   Raw Points        Stroke Points             Tool Settings
       │                     │                       │
       └──────────────┬──────┴──────────────┬────────┘
                      ▼                     ▼
             ┌────────────────────────────────────┐
             │        SVG Path Generator          │
             │  getSvgPathFromStroke()            │
             └────────────────────────────────────┘
                               │
                               ▼
                    ┌──────────────────────────┐
                    │       SVG Renderer       │
                    │   (<svg> + <path>)       │
                    └──────────┬───────────────┘
                               │
       ┌───────────────────────┼─────────────────────────┐
       ▼                       ▼                         ▼
┌──────────────┐     ┌──────────────────┐      ┌────────────────────┐
│ Undo/Redo    │     │ Import/Export    │      │ Base64 Conversion  │
│ State Stack  │     │ parse/save SVG   │      │ captureSvgAsBase64 │
└──────────────┘     └──────────────────┘      └────────────────────┘
```

---

## 🧠 High-Level Layers

1. **UI Layer**
   - Toolbar + settings panel
   - SVG canvas

2. **Interaction Layer**
   - Pointer events
   - Tool switching

3. **Processing Layer**
   - Stroke generation
   - Path conversion
   - Eraser logic

4. **Data Layer**
   - Stroke state
   - Undo/redo stacks

5. **Integration Layer**
   - Iframe messaging
   - Base64 export/import
