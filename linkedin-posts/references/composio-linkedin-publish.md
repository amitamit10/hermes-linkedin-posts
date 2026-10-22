# LinkedIn Publish via Composio (Managed Auth)

Use when the LinkedIn Developer Portal blocks page/app creation or you need programmatic publish without handling OAuth yourself.

## When to Use

- Developer portal returns `/null` or blocks Company Page creation.
- You need to publish a post + carousel (PNGs) + first comment from the agent without asking the user for a password.
- Composio is available (`composio` SDK / CLI installed; resolve the CLI local-first, container path `/root/.composio/composio` only as fallback).

## Procedure

1. **Ensure Composio auth:** `composio login --no-browser --no-wait` → open the printed `https://dashboard.composio.dev/?cliKey=...` URL in a human browser → `composio login --poll` (verify with `composio whoami`).
2. **Link LinkedIn:** `composio link linkedin --no-browser --no-wait` → open the printed `https://connect.composio.dev/link/lk_...` URL and click **Allow**. Verify with `composio link linkedin --list` (status `ACTIVE`).
3. **Derive author URN:** `composio execute LINKEDIN_GET_MY_INFO -d '{}'` → `id` field → `urn:li:person:<id>`.
4. **Prepare assets:** Build carousel slides as 1080x1350 PNGs under `$HERMES_HOME` (PIL in `hermes` container: `docker exec hermes /opt/hermes/.venv/bin/python3 $HERMES_HOME/<script>.py`). Do not publish the PDF directly — Composio `images` expects file paths to PNGs.
5. **Publish:**
   ```bash
   composio execute LINKEDIN_CREATE_LINKED_IN_POST -d '{"author":"urn:li:person:XXXX","commentary":"...","images":["$HERMES_HOME/pic1.png", ...],"visibility":"PUBLIC","lifecycleState":"PUBLISHED"}'
   ```
   Save the returned `urn:li:ugcPost:<id>` (also visible as `https://www.linkedin.com/feed/update/<urn>`).
6. **First comment with report link:**
   ```bash
   composio execute LINKEDIN_CREATE_COMMENT_ON_POST -d '{"actor":"urn:li:person:XXXX","object":"urn:li:ugcPost:YYYY","target_urn":"urn:li:ugcPost:YYYY","message":{"text":"Full report: https://..."}}'
   ```
   The `target_urn` + `object` must both be the `ugcPost` URN; `message.text` ≤1250 chars.

## Raw API via `composio proxy`

For anything the managed tools lack (video, status checks, deletes), call LinkedIn REST directly through the Composio-managed token:

```bash
composio proxy "https://api.linkedin.com/<path>" -t linkedin -X GET|POST \
  -H "Content-Type: application/json" \
  -H "X-Restli-Protocol-Version: 2.0.0" \
  [-H "LinkedIn-Version: 202609"] [-d '{...}']
```

- **Always send `X-Restli-Protocol-Version: 2.0.0`** on `/v2/` calls; add `LinkedIn-Version: 202609` (or current) on `/rest/` calls. Missing version headers cause silent schema mismatches.
- URL-encode URNs in paths (`urn:li:video:X` → `urn%3Ali%3Avideo%3AX`).

## Video posts (raw API — Composio managed tool is images-only)

`LINKEDIN_CREATE_LINKED_IN_POST` accepts images only; `POST /rest/posts` with `content.video` returns 422. Use the Videos API upload flow, then attach the video in a UGC post:

1. **Initialize:** `POST /rest/videos?action=initializeUpload` with `{"initializeUploadRequest":{"owner":"<author-urn>","fileSizeBytes":<exact-size>,"uploadCaptions":false,"uploadThumbnail":false}}` → returns `value.video` URN + `value.uploadInstructions[0].uploadUrl` (+ `uploadToken`).
2. **PUT the bytes** to `uploadUrl` with `Content-Type: video/mp4` **and capture the response `etag` header** (curl `-D headers.txt`). A 200 without the ETag is useless — the upload cannot be completed without it.
3. **Finalize:** `POST /rest/videos?action=finalizeUpload` with `{"finalizeUploadRequest":{"video":"<video-urn>","uploadToken":"<token>","uploadedPartIds":["<etag>"]}}`. Omitting `uploadedPartIds` leaves the video stuck at `WAITING_UPLOAD` forever (finalize then returns 400 "no part upload response").
4. **Poll** `GET /rest/videos/<url-encoded-urn>` until `status` is `AVAILABLE` (single-part uploads under ~4MB need no chunking; expect ~1–3 min processing).
5. **Publish the UGC post** referencing the video URN. Note: `urn:li:video:` URNs are rejected by `v2/ugcPosts` with `INVALID_CONTENT_OWNERSHIP` — attach per the current UGC Post API video-media shape in the LinkedIn docs, and verify with a read-back before announcing success.
6. **Run long uploads in background** (initialize → PUT → poll → finalize → publish spans minutes): background process with logs to `/tmp`, then read the log — never block the turn on the poll loop.

## Pitfalls

- **401 Invalid API key from Composio SDK:** SDK key `ck_...` may be scoped without `Sessions: Read+Write`. Create a **Full access** Project API key in the dashboard; the CLI `composio login` flow avoids this entirely (preferred).
- **`images` must be filesystem paths the hermes container can read** (`$HERMES_HOME/...`), not Host `/home/ubuntu/...` paths — the bind mount is `$HERMES_HOME` inside the container.
- **Comment length limit:** 1250 chars. Keep the first comment to a single link line; the post body holds the narrative (≤3000 chars).
- **Do not embed the external link in the post body** — LinkedIn's algorithm penalizes body links. Body = story + hashtags, comment = link.

## Video creative (feed video, e.g. Playwright-rendered motion demo)

- **Render at feed resolution (1080x1080), not higher.** Downscale in ffmpeg (`-vf "...scale=1080:1080"`); high-res renders waste time and LinkedIn re-encodes anyway. ~12s / ~600KB–1MB at crf 18–19 is a proven envelope.
- **Strip ads and banners from news screenshots before compositing.** Crop to the headline + dek only, then `vision_analyze` the crop to confirm no ads, cookie banners, or sidebars survived — anything extraneous in frame reads as sloppy on feed.
- **Brand the frames:** subject logos/icons (e.g. Meta ∞, product glyph) as large translucent background marks plus small solid badges near captions; plain gradient backgrounds look generic.
- **Verify rendered frames with vision before swapping into the schedule:** inspect 3–4 frames across scenes for cut-off labels, overlapping captions, and unreadable text; fix layout and re-render rather than shipping glitches.
