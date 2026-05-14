---
name: Absorb the Borg Pipeline State
description: AI-assisted YouTube content pipeline — current build state, key paths, how to run
type: project
originSessionId: 59f1e2db-729c-4b77-8f51-a652cd4fc2a3
---
Scout → Script → Review loop is live as of 2026-05-11.

**Why:** Passive income via YouTube AdSense + affiliate links in AI tools + personal finance niche. Mitch reviews every script before anything renders (required by YouTube policy after Jan 2026 enforcement wave).

**Where code lives:**
- Source (git-tracked): `/home/mitch/workspace/absorb-the-borg/` on PVE
- Runtime: `/home/mitch/absorb-the-borg/` on ArcAiVM (192.168.50.196)
- Storage: `/ai-storage/absorb-the-borg/` (ZFS, NFS-mounted, 216GB)
- GitHub: `git@github.com:mitchmclellan/absorb-the-borg.git`

**Secrets:** OpenBao `secret/absorb-the-borg` — youtube creds, anthropic_api_key, telegram_bot_token, telegram_chat_id. VAULT_TOKEN in `/home/mitch/absorb-the-borg/.env` on ArcAiVM.

**To run (from ArcAiVM):**
```bash
cd /home/mitch/absorb-the-borg && set -a && source .env && set +a
python3 run.py --scout              # scrape + Telegram report
python3 run.py --generate <id>      # draft script for topic_id + Telegram notify
python3 run.py --status             # show queue
python3 run.py --render <queue_id>  # render APPROVED script to audio
python3 run.py --render <id> --voice af_nova --speed 1.05  # custom voice/speed
```

**Approve/reject via Telegram:** `/approve <id>` or `/reject <id> [reason]` — arcai-bot handles these directly (queue.db updated immediately).

**To run (from ArcAiVM):**
```bash
python3 run.py --render <id>    # TTS → audio.wav + metadata.json
python3 run.py --assemble <id>  # audio + subtitles → episode.mp4
python3 run.py --render <id> --voice af_nova --speed 1.05
```

**What's built:**
- Topic Scout, Script Generator, Review Queue (all working)
- TTS Renderer (`tts/render.py`) — kokoro-onnx 0.5.0. ~85s CPU render for ~2min audio. Saves `audio.wav` + `metadata.json` (per-section timing). Default voice `af_heart`.
- Video Assembler (`assembler/assemble.py`) — 1920x1080 H.264+AAC, dark branded background, title card, SRT subtitle burn-in, thumbnail.jpg extracted. Output: `/ai-storage/absorb-the-borg/video/<id>/episode.mp4`.
- Orchestrator (run.py), Systemd timers

**GPU note:** onnxruntime-gpu installed but needs CUDA 12 toolkit + cuDNN 9 on ArcAiVM for actual GPU inference. Falls back to CPU automatically. Future task: `apt install` CUDA toolkit.

**What's next:** YouTube Uploader (`uploader/upload.py`) — needs Google OAuth2 flow run once by Mitch, then it's just code.

**How to apply:** When Mitch says anything about Absorb the Borg, channel, videos, scripts, or YouTube pipeline — load CONTEXT.md first and work from ArcAiVM.
