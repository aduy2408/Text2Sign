# FBX Animation Viewer

> Web app for Text2Sign. Text → pose → FBX → 3D viewer on web.

---

| Service | Port | Vai trò |
|---|---|---|
| **Frontend** | `3000` | web viewer (HTML/JS, serve bằng http-server) |
| **Backend** (FastAPI) | `8000` | Nhận request, đẩy task vào Redis queue |
| **Redis** | `6379` | Task queue trung gian, lưu trạng thái task |
| **Pseudo API** | `8001` | Mock translation API (trả về video ID(Task is done seperately but not included in this repo)) |
| **Blender Server** | `9999` (internal TCP) | Luôn warm, nhận lệnh retarget qua socket |

---

## Structure

```
fbxviewer/
├── index.html          ← Frontend chính
├── main.js             ← JS logic (Three.js viewer + polling)
├── style.css           ← Giao diện
├── pseudo_api.py       ← Mock translation API (port 8001)
└── fbx_only_backend/
    ├── main.py                 ← FastAPI app entry point (port 8000)
    ├── pseudo_api.py           ← Runner cho pseudo API
    ├── requirements_api.txt    ← Python deps (fastapi, uvicorn, redis, ...)
    ├── blender_server.py       ← Blender TCP socket server (port 9999 internal)
    ├── blender_retarget_auto.py← Script retarget chạy trong Blender
    ├── batch_convert_amass.py  ← Convert SMPL-X → AMASS
    └── api/
        ├── queue.py            ← Redis task queue + worker
        ├── tasks.py            ← Endpoints /tasks/*
        ├── convert.py          ← Endpoint /convert
        ├── translate.py        ← Endpoint /translate
        ├── pipeline.py         ← Pipeline logic (AMASS → Blender → FBX)
        ├── blender_client.py   ← Client kết nối Blender server
        ├── models.py           ← Pydantic request models
        ├── config.py           ← Paths, constants
        └── helpers.py          ← Utilities
```

## Demo
<figure>
  <video src="demo_vids/caulongbien(1).mp4" muted controls="controls" style="max-width: 100%;">
  </video>
  <figcaption align="center"><i>Hoàng hôn ở cầu Long Biên rất đẹp</i></figcaption>
</figure>

<figure>
  <video src="demo_vids/chuacuaban.mp4" muted controls="controls" style="max-width: 100%;">
  </video>
  <figcaption align="center"><i>Chú của bạn đã 40 tuổi</i></figcaption>
</figure>


## Important notes

### Blender
- Blender đã cài và biến môi trường `BLENDER_BIN` được set

```bash
export BLENDER_BIN=/opt/blender-4.2.8/blender  
```

## Instruction


```bash
# Terminal 1 — Redis
sudo systemctl start redis

# Terminal 2 — Backend
conda activate ml2 && cd fbxviewer/fbx_only_backend && uvicorn main:app --host 0.0.0.0 --port 8000

# Terminal 3 — Pseudo API
conda activate ml2 && cd fbxviewer/fbx_only_backend && python pseudo_api.py

# Terminal 4 — Frontend
cd fbxviewer && npx http-server . -p 3000
```

---

## API Endpoints 

| Method | Endpoint | Mô tả |
|---|---|---|
| `GET` | `/` | Health check |
| `GET` | `/clips` | List các folder trong `output_smplerx/` |
| `POST` | `/translate` | Submit câu văn bản → trả `task_id` ngay |
| `POST` | `/convert` | Submit convert thủ công → trả `task_id` ngay |
| `GET` | `/tasks` | List tất cả task đang có trong Redis |
| `GET` | `/tasks/{id}` | Poll kết quả của 1 task |
| `DELETE` | `/tasks/{id}` | Xóa task khỏi Redis |
| `GET` | `/fbx` | List file `.fbx` đã xuất |
| `GET` | `/fbx/{filename}` | Download file FBX |




