---
name: clipbait
description: Turn long videos (YouTube, Twitch VODs, Rumble, Ganjing) into short vertical viral clips with AI captions — and auto-clip live Twitch streams. Use when the user wants to make short-form clips from a video URL, check clip jobs, or start live auto-clipping.
---

# Clipbait — AI clip generation for agents

Clipbait turns long videos into ready-to-post vertical clips (captions, hooks, watermark), and can continuously auto-clip a live Twitch stream. This skill drives it over Clipbait's REST API.

## Setup (one time)
1. The user gets their **API key** at https://app.clipbait.ai → **Me → API key** (or `POST /api/users/me/api-key` while logged in). It looks like `cbk_live_...`.
2. Store it as `CLIPBAIT_API_KEY`.
3. All requests go to base URL `https://app.clipbait.ai/api` and authenticate with header:
   `X-API-Key: $CLIPBAIT_API_KEY`  (or `Authorization: Bearer $CLIPBAIT_API_KEY`)

Generating clips **spends the user's credits** (1 credit ≈ 1 minute of source video), and live auto-clipping requires a Pro plan.

## Actions

### Generate clips from a video
```bash
curl -s -X POST https://app.clipbait.ai/api/clips/generate \
  -H "X-API-Key: $CLIPBAIT_API_KEY" -H "Content-Type: application/json" \
  -d '{"videoUrl":"<URL>","aspectRatio":"9:16","contentType":"talking_head","maxClips":9}'
# → { "jobId": "..." }
```
- `aspectRatio`: `9:16` (default) or `16:9`
- `contentType`: `gaming` | `talking_head` | `irl` | `music` | `documentary`
- `maxClips`: 1–20 (default 9)
- Optional `clipRangeStartSec` / `clipRangeEndSec` to clip only a slice of a long VOD.

### Check a job / get finished clip URLs
```bash
curl -s https://app.clipbait.ai/api/clips/status/<jobId> -H "X-API-Key: $CLIPBAIT_API_KEY"
# → { status, progress, clips: [{ hook, url, ... }] }
```
Poll until `status` is `completed`; each clip's `url` is the finished, captioned video.

### List recent jobs
```bash
curl -s https://app.clipbait.ai/api/clips/jobs -H "X-API-Key: $CLIPBAIT_API_KEY"
```

### Check a video's duration before clipping
```bash
curl -s -X POST https://app.clipbait.ai/api/clips/probe-url \
  -H "X-API-Key: $CLIPBAIT_API_KEY" -H "Content-Type: application/json" \
  -d '{"url":"<URL>"}'
# → { durationSeconds }
```

### Start live auto-clipping a Twitch stream (Pro)
```bash
curl -s -X POST https://app.clipbait.ai/api/live/start \
  -H "X-API-Key: $CLIPBAIT_API_KEY" -H "Content-Type: application/json" \
  -d '{"videoUrl":"https://twitch.tv/<channel>","cadenceMin":7}'
# → { liveSessionId }
```
Clipbait then watches the live broadcast and produces clips automatically as viral moments happen — no further calls needed.

## Prefer MCP? (ChatGPT / claude.ai / Claude Code)
Instead of raw HTTP, connect Clipbait's MCP server:
`https://app.clipbait.ai/api/mcp/<CLIPBAIT_API_KEY>`
Tools: `generate_clips`, `get_job`, `list_recent_clips`, `start_live_autoclip`, `probe_video`.

## Typical flow
1. `probe_video` to confirm length (and warn if it'll cost a lot of credits).
2. `generate_clips` with the URL → get `jobId`.
3. Poll `get_job` until `completed` → return the clip URLs to the user.
