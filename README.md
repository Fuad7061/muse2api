# MUSE2API

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10%2B-blue?logo=python" alt="Python Version" />
  <img src="https://img.shields.io/badge/FastAPI-0.115%2B-009688?logo=fastapi" alt="FastAPI" />
  <img src="https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker" alt="Docker" />
  <img src="https://img.shields.io/badge/API-OpenAI%20Compatible-green" alt="OpenAI API Compatible" />
  <img src="https://img.shields.io/badge/License-MIT-orange" alt="License" />
  <a href="https://linux.do/" target="_blank"><img src="https://img.shields.io/badge/Community-LINUX%20DO-111827?logo=linux&logoColor=white" alt="LINUX DO" /></a>
</p>

<p align="center">
  Reverse proxy that turns <b><a href="https://muse.ai/">muse.ai</a></b> web free accounts into high-performance, <b>OpenAI-compatible RESTful APIs</b>.
</p>

Through headless browser CDP protocol tunneling, warm WebSocket connection reuse, and dynamic multi-account session management, **muse2api** natively supports text/code chat (with 2-3s streaming response), text-to-image, image editing, text-to-video, image-to-video, automated session renewals, zero-footprint cloud storage via TmpFiles.org, and a modern web dashboard.

---

## 🌟 Key Features

- 💬 **Standard Chat Completions & Responses API**
  - Fully compatible with `/v1/chat/completions` and OpenAI `/v1/responses` endpoints.
  - Native Server-Sent Events (SSE) streaming (`stream=true`) with first-token latency of only ~2–3 seconds.
  - Supports multi-turn conversations and custom system prompts.
  - Automatic model alias routing: models like `gpt-4o`, `claude-sonnet-4`, `deepseek-chat`, and `dall-e-3` are automatically routed.

- 🎨 **High-Quality Image Generation & Editing**
  - Fully compatible with `/v1/images/generations` and `/v1/images/edits`.
  - Supports aspect ratios: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`.
  - Supports reference image uploads, watermark-free raw outputs, and synchronous/asynchronous task polling.
  - Output formats: direct URLs or `b64_json`.

- 🎬 **Text-to-Video & Image-to-Video**
  - Connects to Muse's state-of-the-art video model with custom duration (5s / 6s / 8s / 10s) and aspect ratios (`16:9` widescreen or `9:16` vertical).
  - Supports starting frame reference images for image-to-video generation.
  - Asynchronous task architecture (`POST /v1/videos` to create task + `GET /v1/videos/{task_id}` to poll).

- ☁️ **TmpFiles.org Cloud Storage Integration**
  - Seamlessly uploads generated images and videos to [tmpfiles.org](https://tmpfiles.org) with 48-hour auto-expiration.
  - Eliminates VPS disk space issues and avoids heavy local media accumulation.
  - Configurable via the dashboard Settings tab with graceful automatic fallback to local disk.

- 🧹 **Automated Background Housekeeping**
  - Built-in background daemon purges old task logs and expired local media based on your configured retention period (default: 3 days).
  - Includes an instant manual purge button in the dashboard.

- 🔄 **Multi-Account Pool & Smart Dispatch**
  - Import unlimited Muse accounts.
  - Dispatches requests using warm connection affinity and Least-Recently-Used (LRU) round-robin.
  - Zero-downtime automatic failover if an account hits a quota limit or error.

- 🛡️ **Silent Keepalive & Session Maintenance**
  - Background keepalive automatically calls `/api/session` and `/api/hatch/vm/wake` every 30 minutes to keep cloud VMs warm and sessions active.

- 🧩 **One-Click Chrome Extension & Userscript**
  - Export cookies from your browser session without manual DevTools inspection. Supports HttpOnly cookie extraction with one click.

- 🖥️ **Modern Dark-Mode Dashboard & Online Hot Upgrade**
  - Complete English Web UI to manage accounts, quotas, generation tasks, media library, and system settings.
  - Real-time GitHub update detection with one-click seamless upgrades that preserve your `.env` configuration and account data.

---

## 🚀 Quick Start

### Method 1: Docker Compose (Recommended)

1. **Clone the repository**:
   ```bash
   git clone https://github.com/Fuad7061/muse2api.git
   cd muse2api
   ```

2. **Configure environment variables (optional)**:
   ```bash
   cp .env.example .env
   # Edit .env to set your MUSE2API_KEY
   ```

3. **Start container**:
   ```bash
   docker compose up -d
   ```

4. **Access the Dashboard**:
   Open `http://<YOUR_SERVER_IP>:18610/admin?key=<YOUR_MUSE2API_KEY>` in your browser.

---

### Method 2: Native Host / VPS (Ubuntu / Debian)

1. **Install system dependencies**:
   ```bash
   sudo apt-get update
   sudo apt-get install -y chromium fonts-wqy-zenhei python3 python3-pip python3-venv
   ```

2. **Setup virtual environment**:
   ```bash
   cd /opt/muse2api
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Enable systemd service**:
   ```bash
   sudo cp deploy/muse2api.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now muse2api
   ```

4. **Nginx Reverse Proxy Configuration (Recommended)**:
   ```nginx
   location / {
       proxy_pass http://127.0.0.1:18610;
       proxy_read_timeout 600s;
       proxy_send_timeout 600s;
       proxy_buffering off;
       client_max_body_size 64M;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }
   ```

---

## 📖 API Usage Examples

All requests must include your Bearer token in the header:
```http
Authorization: Bearer <YOUR_MUSE2API_KEY>
```

### 1. Chat Completions (`POST /v1/chat/completions`)

```bash
curl -X POST "http://localhost:18610/v1/chat/completions" \
  -H "Authorization: Bearer m2a_your_secret_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "muse-spark",
    "messages": [
      {"role": "user", "content": "Write a Python quicksort algorithm."}
    ],
    "stream": false
  }'
```

### 2. Image Generation (`POST /v1/images/generations`)

```bash
curl -X POST "http://localhost:18610/v1/images/generations" \
  -H "Authorization: Bearer m2a_your_secret_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "muse-image",
    "prompt": "A futuristic robotic Shiba Inu in space, cinematic lighting, 8k",
    "size": "16:9",
    "response_format": "url"
  }'
```

**Response**:
```json
{
  "created": 1790148495,
  "data": [
    {
      "revised_prompt": "A futuristic robotic Shiba Inu in space, cinematic lighting, 8k",
      "url": "https://tmpfiles.org/dl/12345/image.webp",
      "kind": "image",
      "bytes": 54210
    }
  ]
}
```

### 3. Video Generation (`POST /v1/videos`)

**Step 1: Create Video Task**
```bash
curl -X POST "http://localhost:18610/v1/videos" \
  -H "Authorization: Bearer m2a_your_secret_key" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A cute orange tabby kitten playing on grass in bright sunlight",
    "duration": 5,
    "size": "16:9"
  }'
```
Returns: `{"id": "task_xyz789", "status": "queued"}`

**Step 2: Poll Task Status**
```bash
curl "http://localhost:18610/v1/videos/task_xyz789" \
  -H "Authorization: Bearer m2a_your_secret_key"
```

When completed:
```json
{
  "id": "task_xyz789",
  "status": "succeeded",
  "result": {
    "url": "https://tmpfiles.org/dl/12345/video.mp4"
  }
}
```

---

## ⚙️ Environment Variables

| Variable | Default | Description |
|---|---|---|
| `MUSE2API_KEY` | Auto-generated | Dashboard and API Bearer authentication key |
| `MUSE2API_HOST` | `0.0.0.0` | Server listening host |
| `MUSE2API_PORT` | `18610` | Server port |
| `MUSE2API_MEDIA_STORAGE` | `local` | Media storage engine: `local` or `tmpfiles` |
| `MUSE2API_LOG_RETENTION_DAYS` | `3` | Retention days for old tasks and local media |
| `MUSE2API_PUBLIC_BASE` | Empty | Optional public base URL |
| `MUSE2API_CHROMIUM` | `chromium` | Path to Chromium executable |
| `MUSE2API_CDP_PORT` | `19210` | Internal CDP debugging port |
| `MUSE2API_IMAGE_TIMEOUT` | `240` | Max timeout for image generation (seconds) |
| `MUSE2API_VIDEO_TIMEOUT` | `600` | Max timeout for video generation (seconds) |
| `MUSE2API_CHAT_TIMEOUT` | `300` | Max timeout for chat completion (seconds) |

---

## 🔒 Security & Privacy

- **Data Privacy**: All account credentials and task state files are stored strictly inside your local `data/` directory. No telemetry or credentials are sent to external servers.
- **Compliance**: This project is intended solely for educational, testing, and operational research purposes. Please follow all relevant platform terms of service.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
