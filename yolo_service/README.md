# eVTOL YOLO Service

## Setup & Run
```bash
cd yolo_service
pip install -r requirements.txt
python yolo_service.py
```
Service runs on **http://localhost:5050**

## What it does
- Receives frames (base64 PNG) at 10fps from the frontend
- Saves raw frames → `captured_frames/` (default) or Azure Blob Storage (optional)
- Detects buildings/walls/vehicles/structures using OpenCV colour-segmentation (no YOLO/COCO — avoids false positives on 3-D renders)
- Computes world-space coordinates using quaternion-corrected camera→world transform
- Saves annotated frames (YOLO-style bboxes) → `captured_frames/annotated/` (default) or Azure Blob Storage (optional)
- Returns JSON with obstacle list + annotated image to frontend

## Store frames in Azure Blob Storage
1) Install deps:
```bash
cd yolo_service
pip install -r requirements.txt
```

2) Set env vars (PowerShell example):
```powershell
$env:FRAME_STORE="azure"
$env:AZURE_CONTAINER="imageframes"
$env:AZURE_PREFIX="frames"
$env:AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=...;AccountName=evtolframes;AccountKey=...;EndpointSuffix=core.windows.net"
python yolo_service.py
```

Notes:
- Your Azure resource group (`arc-assdl-rg`) is not needed in code; only the storage account credentials are.
- Frames are uploaded as blobs under `frames/raw/` and `frames/annotated/` in the `imageframes` container.

## Folder structure after running
```
yolo_service/
├── yolo_service.py
├── requirements.txt
└── captured_frames/
    ├── 20250429_120000_frame_000001.png   ← raw frames
    └── annotated/
        └── 20250429_120000_frame_000001_annotated.png
```

## API
| Method | Path           | Description                    |
|--------|----------------|--------------------------------|
| GET    | /health        | Service status                 |
| POST   | /detect        | Send frame, get obstacles      |
| POST   | /reset_smooth  | Clear EMA state (call on reset)|
| GET    | /frames        | List saved frames              |
| POST   | /clear         | Delete all saved frames        |
