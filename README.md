# Instagram Downloader × Rio

Flask app: paste a link to a **public** Instagram reel or video, then save the video (with or without
the Rio ghost watermark), an MP3, the original file, the cover image or a details TXT.

Built the same way as the Facebook / TikTok Downloader × Rio projects: same pages, same tabs, same
Render setup. Theme: hot pink, black and light yellow.

## Folder layout (Flask needs exactly this)

```
app.py
requirements.txt
render.yaml
templates/index.html
static/app.js
static/style.css
static/watermark.png     <- ghost logo burned into videos
static/ghost.png         <- logo in the header / hero
static/favicon.png
```

## Run locally

```bash
pip install -r requirements.txt
python app.py            # http://localhost:5000
```

## Deploy on Render

Push this folder to GitHub, create a **Web Service** from the repo (render.yaml is picked up).

## Links that work

`instagram.com/reel/...`, `instagram.com/reels/...`, `instagram.com/p/...` (video posts),
`instagram.com/tv/...`, `instagram.com/<user>/reel/...`, `instagr.am/...`

Only **public** videos work. Stories, private accounts, close-friends posts, photo-only posts and anything
that needs a login cannot be saved. For a post with several items, the first video is used.

## Downloads, in short

| Tab | What you get | How |
|---|---|---|
| Video | MP4 in the chosen quality. Watermark switch + corner picker. | Watermark on: ffmpeg job. Off: direct stream. |
| Audio | Real **MP3** with title/creator tags. | ffmpeg job, using the video's own sound. |
| Document | **Original video file** (no watermark, no re-encode), **cover image**, **details TXT**. | Direct stream (or a quick join job). |

Some qualities are served as two separate streams (picture only + sound only). For those the server
downloads both and joins them with ffmpeg **without re-encoding** (`-c copy`), so quality is not lost.
Every download starts with `POST /api/prepare`, so any failure shows as a message in the page.

## Environment variables (all optional)

| Name | Default | Meaning |
|---|---|---|
| `PROVIDERS` | `ytdlp` | Lookup providers (yt-dlp does the work for Instagram). |
| `WATERMARK` | `1` | `0` switches the watermark feature off. |
| `WATERMARK_POSITION` | `br` | Default corner: `br`, `bl`, `tr`, `tl`. |
| `WATERMARK_SIZE` | `0.30` | Logo width as a share of the video width. |
| `WATERMARK_OPACITY` | `0.92` | 0.1 to 1. |
| `WATERMARK_IMAGE` | `static/watermark.png` | Use another PNG (transparent background). |
| `RATE_LIMIT` / `RATE_WINDOW` | `12` / `60` | Requests per visitor per window (seconds). |
| `GLOBAL_RATE_LIMIT` | `240` | Requests per window for the whole site. |
| `PROXY_HOPS` | `2` | Proxies in front of the app (see below). |
| `MAX_SOURCE_MB` | `100` | Largest video that ffmpeg will process. |
| `MAX_ACTIVE_JOBS` | `5` | ffmpeg jobs allowed at once. |
| `FFMPEG_PRESET` | `ultrafast` | `veryfast` gives smaller files but uses more CPU. |
| `PROXY_URL` | - | Proxy for Instagram requests (residential works best). |
| `COOKIES_FILE` | - | Path to a Netscape `cookies.txt` (see the note below). |
| `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` | - | Keeps the download counter across restarts. |
| `COUNTER_KEY` | `ig_downloads` | Counter name (separate from the other sites' counters). |
| `DEBUG_ERRORS` | - | `1` shows the real error in the UI and enables `/api/debug/ip`. |

### Rate limit and `PROXY_HOPS`

`X-Forwarded-For` can be forged by visitors, so the app trusts only the entry added by your own
proxies, counted from the right. On Render this is normally 2. To check: set `DEBUG_ERRORS=1`,
open `/api/debug/ip` and confirm `seen_as` is **your** IP, then remove `DEBUG_ERRORS`.

## Notes (please read: Instagram is stricter than Facebook / TikTok)

* **Instagram blocks many datacenter IPs** (Render, AWS...) and often answers "login required" or
  "rate-limit reached". If lookups fail even for public reels, set `PROXY_URL` to a residential proxy.
* `COOKIES_FILE` is an **optional last resort**. Cookies come from a real Instagram login, so use a
  throw-away account, never your main one, and never commit the file to GitHub (put it in Render's
  Secret Files). Instagram may restrict an account that is used this way.
* **Instagram changes often.** If lookups suddenly start failing, redeploy so `yt-dlp` updates
  (it is not version-pinned on purpose). Render free plan: Manual Deploy > "Clear build cache & deploy".
* Render's free plan sleeps after 15 min idle (first request is slow) and wipes temp files on restart.
* Replace `static/watermark.png` with any transparent PNG to rebrand.
* Only save videos you made or have permission to use.
