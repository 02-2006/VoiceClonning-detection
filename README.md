# VoiceShield AI — Skeleton

A minimal, **honest** end-to-end skeleton for an AI voice-clone / anti-spoofing
detector: real audio preprocessing, real RawNet2 + AASIST model architectures,
a real weighted-fusion ensemble, a real FastAPI backend, and a real Next.js
frontend (mic recording + upload + result dashboard + history).

## What's real vs. what you still need to add

**Real / working:**
- Audio preprocessing pipeline (decode → mono → 16kHz → trim silence → normalize)
- RawNet2 and AASIST model *architectures* (PyTorch, real forward passes)
- Ensemble fusion math, risk-level thresholds, CAPTCHA-trigger logic
- FastAPI routes: `/predict`, `/upload`, `/history`, `/health`, `/models`
- SQLite scan history
- Next.js dashboard: mic recording (MediaRecorder API), file upload, result UI, history table
- Docker + Netlify config

**You still need to add:**
- **Pretrained checkpoints** for RawNet2 and AASIST (e.g. trained on ASVspoof2019 LA).
  This sandbox has no internet access, so none are bundled. Without them, `/predict`
  returns **HTTP 503** — it refuses to invent a confidence score rather than faking one.
  Get official weights from the ASVspoof baseline repos, or train your own, and drop
  them at `backend/checkpoints/rawnet2.pth` and `backend/checkpoints/aasist.pth`.
- WebSocket streaming endpoint (`/stream`) for live mic analysis — the frontend already
  chunks audio every 250ms via MediaRecorder, ready to be wired to a streaming route.
- Voice CAPTCHA endpoint (`/voice-captcha`) — the ensemble already flags
  `requires_captcha` when confidence is in the 0.40–0.70 uncertain band; the frontend
  surfaces that flag; the actual challenge-generation + re-verification round trip
  isn't implemented yet.
- Mel-spectrogram / waveform visualization on the result page (Recharts is installed,
  not yet wired to a chart).

## Local development

```bash
# backend
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload

# frontend (separate terminal)
cd frontend
npm install
npm run dev
```

Frontend expects the API at `NEXT_PUBLIC_API_URL` (defaults to `http://localhost:8000`).

## Deployment

**Frontend → Netlify.** `netlify.toml` is set up with the official Next.js plugin.
Push this repo, connect it in the Netlify UI, and set the environment variable
`NEXT_PUBLIC_API_URL` to wherever you deploy the backend.

**Backend → NOT Netlify.** PyTorch + model checkpoints don't fit in Netlify's
serverless function limits (package size + execution time). Use a real container
host instead:
- [Render](https://render.com) — easiest, has a free tier, native Dockerfile support
- [Railway](https://railway.app)
- [Fly.io](https://fly.io)
- [Hugging Face Spaces](https://huggingface.co/spaces) (Docker SDK) — good fit since
  this is an ML model anyway

```bash
cd backend
docker build -t voiceshield-backend .
docker run -p 8000:8000 voiceshield-backend
```

## API reference (current)

| Method | Path | Description |
|---|---|---|
| POST | `/predict` | Upload audio, get full ensemble result (503 if checkpoints missing) |
| POST | `/upload` | Store raw audio, get back a file id |
| GET | `/history` | Recent scan history |
| GET | `/health` | Service + model load status |
| GET | `/models` | Model checkpoint paths, device, ensemble weights |
| GET | `/docs` | Auto-generated OpenAPI docs |
