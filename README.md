# runpod-videogen-hermes

End-to-end 3:4 video generation via RunPod serverless endpoint for Hermes Agent.

## What it does

- Submit video generation jobs to a RunPod endpoint
- Poll until completion
- Download base64-encoded MP4
- Verify video properties

## Endpoint

`cbrzbzlinjhsc0`

## ⚠️ Cold start / warm-up (required)

Video generation is heavy. The first request after idle triggers worker boot (~1–2 min) and **will** fail with `HTTP 503` / `queue_error`. The correct pattern is:

```bash
# Step 1: Warm up the worker with a tiny request
python3 scripts/rp_submit.py --prompt "hi"

# Step 2: Wait until status is COMPLETED (or IN_PROGRESS, then poll)
python3 scripts/rp_status.py --run-id <RUN_ID>

# Step 3: NOW submit your actual video generation job
python3 scripts/rp_submit.py --prompt "Your real video prompt"
```

Or use the one-shot helper with `--warm-up`:

```bash
bash scripts/rp_complete_video.sh --prompt "Your real prompt" --output ./video.mp4 --warm-up
```

### Cold-start timeline

| Time | What happens |
|------|--------------|
| 0s | Submit `"hi"` → `IN_QUEUE` |
| ~30s | Worker boots, model loads → `IN_PROGRESS` |
| ~60–120s | Warm-up completes → `COMPLETED` |
| +5s | Submit real prompt → normal speed (2–4s) |

**Do not skip the warm-up step.** Without it, your first real video job will queue behind the boot process and appear to hang or fail.

## Quick start

```bash
python3 scripts/rp_submit.py --prompt "Your prompt"
python3 scripts/rp_status.py --run-id <RUN_ID>
python3 scripts/rp_save_video.py --run-id <RUN_ID> --output ./video.mp4
```

Or all-in-one:
```bash
bash scripts/rp_complete_video.sh --prompt "Your prompt" --output ./video.mp4
```

## Prerequisites

- Python 3.11+ (stdlib only)
- RunPod API key at `/tmp/rp.key`

## Verified output

- Codec: H.264
- Resolution: 480×720 (portrait, 3:4)
