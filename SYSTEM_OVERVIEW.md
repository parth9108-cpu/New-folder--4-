# Final System Overview: 2D Blueprint to 3D (Web / VR / AR)

## End-to-End Pipeline

```text
User uploads blueprint
        ↓
Image preprocessing
        ↓
AI floorplan parsing
        ↓
Layout vectorization
        ↓
Room & object detection
        ↓
3D mesh generation
        ↓
Object placement
        ↓
3D visualization (Web / VR / AR)
```

---

## Module 1 — Blueprint Upload System

**Goal:** Accept a 2D blueprint image or PDF.

### Frontend
- React / Next.js upload page (`Upload 2D Blueprint`)
- Supported formats:
  - PNG
  - JPG
  - PDF

### Example request
```js
const formData = new FormData();
formData.append("file", blueprint);

fetch("/api/upload", {
  method: "POST",
  body: formData,
});
```

---

## Module 2 — Image Preprocessing

Blueprints may include scan noise, labels, and artifacts.

### Tools
- OpenCV
- Pillow

### Processing
1. Convert to grayscale
2. Remove noise
3. Detect edges
4. Resize image

### Example
```python
import cv2

img = cv2.imread("blueprint.png")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
edges = cv2.Canny(gray, 50, 150)
```

### Output
- Cleaned blueprint image

---

## Module 3 — AI Floorplan Parsing

**Goal:** Detect architectural elements.

### Classes
- Walls
- Rooms
- Doors
- Windows
- Stairs

### Models
- Primary: CubiCasa5K (pretrained)
- Alternatives: SegFormer, Mask2Former

### Output
Semantic segmentation map:

```json
{
  "walls": "mask",
  "doors": "mask",
  "windows": "mask",
  "rooms": "mask"
}
```

---

## Module 4 — Vectorization (Layout Graph)

Convert segmentation maps into clean geometry.

### Output objects
- **Nodes:** Wall intersections
- **Edges:** Wall line segments
- **Polygons:** Room boundaries

### Libraries
- Shapely
- NetworkX
- OpenCV

### Graph concept
```text
node1 ---- node2
  |          |
node3 ---- node4
```

This corresponds to the 2D mesh stage.

---

## Module 5 — Room Detection

Determine room structures and semantics.

### Model
- RoomFormer

### Outputs
- Living room polygon
- Bedroom polygon
- Kitchen polygon
- Room connectivity
- Room size
- Room adjacency

---

## Module 6 — Door & Window Detection

Use object detection to localize openings and symbols.

### Model
- YOLOv8

### Detect
- Doors
- Windows
- Stairs
- Furniture symbols

### Example output
- `door at (x, y)`
- `window at (x, y)`

---

## Module 7 — 3D Mesh Generation

Convert 2D geometry into 3D meshes.

### Concept
```text
2D wall line → extrude vertically → 3D wall mesh
```

### Parameters
- Wall height: `3.0m`
- Wall thickness: `0.2m`

### Tools
- Blender Python API
- Trimesh
- PyMesh

---

## Module 8 — Add Objects

Place 3D assets into generated geometry.

### Example models
- `door.obj`
- `window.obj`
- `bed.obj`
- `table.obj`

### Example position
- `door_position = (x, y, z)`

### Asset sources
- Sketchfab
- Poly Haven
- Free3D

---

## Module 9 — Texturing

Apply visual materials for realism.

### Materials
- Wall paint
- Floor texture
- Glass material

### Tools
- Blender
- ThreeJS materials

---

## Module 10 — Web 3D Viewer

Render the output model in-browser.

### Primary option
- ThreeJS

### Load formats
- GLTF
- OBJ

### Interaction
- Rotate
- Zoom
- Walkthrough

### Viewer pipeline
```text
3D model → ThreeJS renderer → interactive scene
```

---

## Module 11 — AR / VR Mode (Optional)

### AR
- WebXR
- ARCore
- ARKit

### VR
- Unity
- Unreal
- Meta Quest

Supports immersive visualization and training scenarios.

---

## Complete Tech Stack

### Frontend
- React
- ThreeJS
- Tailwind

### Backend
- Python
- FastAPI
- PyTorch

### AI
- CubiCasa5K
- YOLOv8
- RoomFormer

### Geometry
- Shapely
- Trimesh
- Blender API

### Visualization
- ThreeJS
- Unity (optional)

---

## Suggested Service Boundary (Implementation-Oriented)

For productionization, separate into services:

1. **Upload Service** (validation, storage, metadata)
2. **Preprocessing Service** (OpenCV/Pillow transforms)
3. **Parsing Service** (segmentation + detection)
4. **Geometry Service** (vectorization, graph, room topology)
5. **Mesh Service** (3D extrusion + asset placement)
6. **Viewer Service** (GLTF delivery + web client)

This keeps model workloads scalable and the UI responsive.


---

## How to Run the Project (Recommended Reference Setup)

Because this document describes a multi-service system, the easiest way to run it is with a **frontend + API + worker** structure.

### 1) Prerequisites

- Node.js 18+
- Python 3.10+
- pip / virtualenv
- (Optional) Docker + Docker Compose
- (Optional for Module 7) Blender installed and available in PATH

### 2) Suggested folder structure

```text
project-root/
  frontend/              # React / Next.js UI
  backend/               # FastAPI app (upload + orchestration)
  workers/               # AI + geometry + mesh jobs
  assets/                # 3D models/textures
  output/                # generated GLTF/OBJ files
```

### 3) Run Frontend (Next.js)

```bash
cd frontend
npm install
npm run dev
```

Default local URL: `http://localhost:3000`

### 4) Run Backend API (FastAPI)

```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -U pip
pip install fastapi uvicorn python-multipart opencv-python pillow shapely networkx trimesh torch ultralytics
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API URL: `http://localhost:8000`

### 5) Run Worker (AI + Mesh pipeline)

If you separate long-running tasks into a worker process:

```bash
cd workers
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install opencv-python pillow torch ultralytics shapely networkx trimesh
python worker.py
```

### 6) Minimal end-to-end local flow

1. Open frontend at `http://localhost:3000`
2. Upload PNG/JPG/PDF blueprint
3. Frontend sends `POST /api/upload` (or proxied backend upload endpoint)
4. Backend preprocesses image and triggers parsing/vectorization/mesh jobs
5. Generated model saved as `output/<project_id>.gltf`
6. Viewer loads GLTF for interaction (rotate/zoom/walkthrough)

### 7) Optional one-command run (Docker Compose)

```yaml
# docker-compose.yml (example)
services:
  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    depends_on: [backend]
  backend:
    build: ./backend
    ports: ["8000:8000"]
    depends_on: [worker]
  worker:
    build: ./workers
```

Start:

```bash
docker compose up --build
```

### 8) Quick smoke checks

```bash
curl http://localhost:8000/health
curl -X POST -F "file=@sample_blueprint.png" http://localhost:8000/upload
```

### 9) Common issues

- **Large model downloads**: first run can be slow when YOLO/segmentation weights are fetched.
- **No GPU**: inference works on CPU but will be significantly slower.
- **Blender not found**: ensure `blender` executable is installed and available in PATH for mesh/texturing stages.
- **CORS/UI upload errors**: configure FastAPI CORS and frontend API base URL.
