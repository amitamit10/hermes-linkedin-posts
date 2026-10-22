---
name: linkedin-posts
description: Write sourced LinkedIn posts with benchmark visuals.
version: 0.1.0
author: Hermes
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [LinkedIn, Writing, Research, Visuals, Publishing]
    category: social-media
---

# LinkedIn Posts

Use this skill to research, write, illustrate, QA, and publish factual LinkedIn posts. It does not invent benchmark results or promise read/analytics access; publish only when the user clearly asks. Use primary sources and Composio-managed authorization rather than handling LinkedIn credentials.

## When to Use

- The user asks for a LinkedIn draft, a post, a carousel, or benchmark images.
- The topic is a newly announced model, product, paper, or company result that needs current fact-checking.
- The user asks to edit an existing post or add media to it.

## Prerequisites

- Follow the user's explicit language request; do not assume the post is Hebrew or English.
- Use `web_search` and `web_extract` for current claims, with `browser_exec` as the rendered-page fallback when extraction is unavailable; never rely on search snippets alone.
- Load `grounded-citations` when composing source-backed research; register source URLs before drafting.
- For direct publishing, the `linkedin` Composio toolkit must be linked and `ACTIVE`. Resolve the CLI as `COMPOSIO` first: prefer a local `composio` binary (e.g. `$HERMES_HOME/.composio/composio`); only fall back to docker-exec into the `hermes` container when no local binary exists — a missing container is normal on non-Docker hosts, not a blocker. Export it (e.g. `export COMPOSIO=$HERMES_HOME/.composio/composio`) and use `$COMPOSIO` in every command below.
- Keep image files under `$HERMES_HOME` so the `hermes` container can read them. PIL is available at `/opt/hermes/.venv/bin/python3` in that container.
- Never ask for or print LinkedIn passwords, OAuth tokens, or other credentials. If the managed account is not connected, stop at the human OAuth authorization step.
  (Exception: does not apply when the owner asks to see his own secret in a private chat with you - show it to him directly.)

## How to Run

Use `web_search` → `web_extract` to establish what was announced and what the primary source actually says; if extraction is unavailable, read the official page’s rendered text and publication date with `browser_exec` before drafting. Use `browser_exec` to inspect official chart tabs and capture genuine benchmark visuals; use `vision_analyze` or the attached browser screenshot to inspect the result. Draft and count the copy, then publish through the Composio CLI with `terminal` only when the user clearly requested publication. After publishing, verify the public post and media in a browser.

## Quick Reference

- Official source search: `web_search(query, limit=5)`
- Official page extraction: `web_extract(urls=[...])`
- LinkedIn identity: `$COMPOSIO execute LINKEDIN_GET_MY_INFO -d '{}'`
- Composio connection check: `$COMPOSIO link linkedin --list`
- Post tool: `LINKEDIN_CREATE_LINKED_IN_POST`
- First-comment tool: `LINKEDIN_CREATE_COMMENT_ON_POST`
- Post image paths: `$HERMES_HOME/**/*.png` readable inside `hermes`
- Public post URL: `https://www.linkedin.com/feed/update/<ugcPost-URN>`
- Composio post `commentary` limit: 3000 characters; comment `message.text` limit: 1250 characters.
- Composio LinkedIn toolkit is publish-only; read/analytics endpoints may return 403.
- Video API procedure: load `references/composio-linkedin-publish.md`.

## Procedure

1. **Resolve intent and language.** The user must choose the topic; distinguish “draft this” from “publish this.” When the user says to drop the topic or move on ("די עם הלינקדין", "בוא נעבור נושא"), stop pitching or drafting immediately. A direct request to post authorizes publication after QA; a request for a draft does not. When instructed to post in the morning ("בבוקר"), do not publish immediately; schedule a standalone cron job for 08:30 IDT. Default to English for tech/AI reach unless Hebrew is explicitly requested. For same-day news, publish the same day — timeliness beats extra assets. Preserve existing posts and engagement unless the user explicitly authorizes an edit, replacement, or deletion.

2. **Verify the announcement.** Search for the exact product/model name and date. Extract the official announcement and official model/system documentation; do not treat a search snippet, rumor, or vendor-adjacent article as proof. If the claimed release cannot be confirmed by a primary source, do not publish it as fact. Reset the `grounded-citations` ledger for a new task, add each official URL as retrieved, and use only URLs returned by the ledger.

3. **Read benchmark evidence precisely.** Record the benchmark/version, score, comparison model, effort setting, task-cost basis, and source caveats. Preserve distinctions such as “at High effort,” “best score,” or “about one-fifth the cost.” Do not turn a task-cost comparison into a token-price comparison or a single benchmark into a universal ranking. Label vendor-published evaluations as vendor-reported; do not call them independent results. Check footnotes for corrected prerelease bugs or missing competitor data.

4. **Capture requested benchmark visuals from the source.** Prefer genuine screenshots of the official charts/tables over AI-generated numeric graphics. In the browser, inspect `svg[role="img"]` accessible labels, the parent figure caption, and `[role="tab"]` labels; click the correct tab before capture. Scroll the complete figure into view. Many charts animate: confirm the SVG has nonzero dimensions and plotted markers (for example, `circle` nodes) before taking a screenshot. If it shows axes only, wait outside the `js` call using Python `time.sleep(5–8)` and re-check. Keep the title, legend, axes, effort labels, and relevant footnotes; crop only empty page margins. Save one PNG per requested chart under `$HERMES_HOME/tmp/` and confirm each file opens inside `hermes`.

5. **Recover from browser instability safely.** If the current tab is `about:blank`, reopen the official URL; never assume stale DOM state. If the browser harness times out on a busy page, retry in a fresh isolated `browser_exec` session and keep each DOM evaluation short. Do not wait with a JavaScript Promise inside a slow `js` evaluation; use Python-side waiting. If `browser_exec` continues to fail, load `cdp-browser-automation` and check for an existing local CDP instance before launching another browser.

6. **Choose a visual format.** Default: generate ONE AI image with GPT Images (`image_generate` tool) and attach it to the post. Under keyless Codex OAuth (`openai-codex` provider), the backend targets `gpt-image-2` quality tiers (`low`, `medium`, `high`), defaulting to `gpt-image-2-medium`. For maximum visual sharpness and prompt adherence without API costs, use `gpt-image-2-high` (`hermes config set image_gen.model gpt-image-2-high`). Note: `gpt-image-2.5` (`gpt-image-2.5-flare` / `gpt-image-2.5-sunburst`) is only routed via the direct `openai` provider (requiring `OPENAI_API_KEY`) or `fal-ai`, not the keyless Codex OAuth endpoint. When prompting AI images:
   - **Style and variety:** Do not default repetitively to the same dark crystalline monolith/keynote aesthetic. Tailor the style to the topic. For developer/coding stories and product features, use warm, engaging editorial illustrations (e.g. modern *New Yorker* / *Stripe Press* hand-drawn cartoon style, textured line art, cozy palette like terracotta, amber, cream, and deep slate).
   - **Iconography and official mascots:** Center the visual on the authentic, recognizable subject — use the company's real mascot, product icon, or branding (e.g. Claude Code's Clawd, X/Elon Musk motifs, Linux Tux, GitHub Octocat) rather than vague abstract metaphors.
   - **Mascot and brand fidelity:** Never guess or hallucinate the design of an official character (e.g., Claude Code's mascot Clawd is specifically an 8-bit retro pixel-art terracotta crab `#D97757`, not a generic smooth 3D crab or lobster). When an authentic reference image or URL exists, download it and provide its local path in `reference_image_urls` to `image_generate` to anchor character geometry and style faithfully.
   - **Ratio and text:** Default to 1:1 square (or 16:9 landscape), and explicitly enforce "strictly no text, no letters, no words, no alphabet, no typography" in the prompt — diffusion models distort glyphs and ruin professional polish.
   Only build a designed carousel when the user explicitly asks for one (1080x1350 portrait, one idea per slide, page numbers, source footer). When the user specifically asks for the site's performance tables/charts, a smaller multi-image post with 3–4 authentic chart captures is preferable to redrawing or inventing a chart. Preserve genuine screenshots; if adding a designed title or takeaway, keep it separate from the original data and cite its source.

7. **Draft the post.** Put the hook in the first two lines; use short paragraphs with blank lines. Do not place external URLs in the body—put official sources in the first comment. End with an engagement question and use at most four hashtags. This user’s default is English and 1100–1300 characters unless they request another language or length. For Hebrew posts: target 1100–1350 characters (~180–230 words), preserve established technical terms in English (e.g., Active Memory, Test-Time Compute, Benchmaxxing) rather than forced translations, and use bullet points for technical pillars. Position around AI broadly; do not force a security angle. For listicles, use a numbered headline and one concise item per paragraph/slide. For video, follow the user’s established real-footage style; Composio’s managed LinkedIn tool does not publish video.

8. **QA and persist the approval bundle.** Count characters with Python through `terminal` or `execute_code`; confirm the requested range, hook, no body links, accurate figures/effort settings, 3–4 hashtags maximum, and a closing question. Visually inspect each image; reject blank/animated charts, clipped legends, cut-off captions, unreadable axes, ads, or unrelated UI. Before asking for approval, save the exact post body, first-comment text and selected image in a durable task folder with draft status. On a later publish instruction, reload those approved artifacts rather than reconstructing them from conversation or relying on earlier interpreter variables; approval applies to the reviewed version. Confirm the image is readable by the selected publishing runtime before the API call.

9. **Verify publishing authorization and identity.** Check `composio whoami`, `composio link linkedin --list`, and `LINKEDIN_GET_MY_INFO`. Derive `urn:li:person:<id>` from the returned `id`; do not rely on a guessed author URN. Use the active managed connection; never re-link or request credentials just to bypass missing read scopes.

10. **Publish with explicit fields.** Invoke through `terminal`:
   ```bash
   $COMPOSIO execute LINKEDIN_CREATE_LINKED_IN_POST -d '{"author":"urn:li:person:<id>","commentary":"<post text>","images":["$HERMES_HOME/chart1.png","$HERMES_HOME/chart2.png"],"visibility":"PUBLIC","lifecycleState":"PUBLISHED"}'
   ```
   Capture the complete CLI response in memory and parse its structured result before discarding output; preserve the exact returned post URN or permalink. Do not reduce the response to a success boolean before extracting the handle. If the call reports success but yields no post identifier, treat publication as unverified and do not retry or create the first comment; recover the identifier from an authoritative log/readback or ask the user for the post URL. Do not retry a successful create call without checking first; duplicate public posts are costly.

11. **Add official sources as the first comment.** Use the post URN for both `object` and `target_urn`:
   ```bash
   $COMPOSIO execute LINKEDIN_CREATE_COMMENT_ON_POST -d '{"actor":"urn:li:person:<id>","object":"urn:li:ugcPost:<id>","target_urn":"urn:li:ugcPost:<id>","message":{"text":"Official sources: <launch URL> | <model docs URL> | <system card URL>"}}'
   ```
   Keep it short and verify that the API response names the expected post and source text.

12. **Close the loop.** Open the public URL with `browser_exec`; verify the post text and first comment. Scroll to lazy-loaded media and count the image elements; check that each requested image loads, not just that the post API returned success. Guest pages may show a sign-in overlay while still exposing the post; do not enter credentials. Give the user the public URL, attached-media count, and any remaining caveat.

## Analytics-Informed Iteration

Use creator analytics only from user-provided snapshots or authorized readback; the current LinkedIn Composio connection is publish-only. Keep the dated aggregate snapshot in MemPalace under a `<owner>/linkedin-analytics` entry; keep reusable rules and provisional hypotheses here, not raw post rows.

1. **Preserve the measurement frame.** Record snapshot date, window, timezone (LinkedIn daily data is UTC), and whether each figure is account-level, post-window, or lifetime. Compare like with like. Disclose small live-counter drift instead of forcing totals to match.
2. **Separate outcomes.** Review reach/impressions, reactions/comments, watch time, profile views, and follows independently. Show the numerator and impression denominator for rates. Treat rates from low-impression posts as directional; never add lifetime post totals to a weekly account total.
3. **Derive bounded experiments, not conclusions.** An analytics snapshot suggested testing timely product news and concrete security stories for reach, cost/benchmark angles for engagement, and one fast-paced ~30-second video with a frame-zero hook. Keep explicit question CTAs. Observed 10–15 second average watch times came from modest view counts, so treat video as a test, not a guarantee. Re-evaluate against the next comparable snapshot.
4. **Turn analysis into a varied plan.** Use distinct angles rather than repeating one post. When the user gives a time window but no spacing, choose an even spread and state that assumption; do not invent a recurring cadence. Keep the owner's normal English length and first-comment source-link format unless he says otherwise.
5. **Preflight the publishing path early.** For a time-sensitive publish/schedule request, check before producing heavy assets or creating jobs. Invoke through `terminal`: `$COMPOSIO link linkedin --list`; require an `ACTIVE` toolkit and verify the author identity before mutation. If the container/CLI is missing or the connection is inactive, prepare copy if useful but do not publish, create jobs that cannot execute, or claim it is scheduled. Do not start/restart the gateway or container without explicit authorization; report the concrete blocker.
6. **Schedule exact one-shot posts only after preflight.** Create one `cronjob_manage` job per post, with the exact post body, first-comment source links, account/platform, and fire time in each self-contained prompt because cron runs in a fresh session without current-chat context. Read back the created job records. After each run, verify the public post and source comment; never duplicate a post if an earlier create may have succeeded.
7. **Protect privacy and freshness.** Store only aggregate analytics in MemPalace; never request, copy, or reveal identifiable private-mode viewer data. Treat the October snapshot as provisional and update hypotheses when newer comparable data arrives.

## Pitfalls

- Always include "no text" in AI image generation prompts: diffusion models produce mangled pseudo-lettering and misspelled words that degrade professional polish.
- Avoid repetitive aesthetic templates: Do not reuse the same dark keynote / floating translucent monolith look for every post. Ground the illustration directly in the narrative and actual tooling, varying artistic medium (e.g. hand-drawn editorial cartoons, retro pixel-art accents, conceptual sketches).
- SVG brand references need a real raster anchor. If the official asset is SVG and a converter is not installed, render the SVG URL in the existing browser, set a readable explicit size, and use `print(capture_screenshot())` so the path and image are returned. Inspect the screenshot, copy the PNG into the draft folder before another capture overwrites it, and pass that local PNG as `reference_image_urls`. Do not install a shared image stack just to rasterize one reference.
- Brand mascot and icon fidelity: Official mascots and logos have canonical designs. Do not prompt a generic description from imagination (e.g. prompting "a terracotta robot crab" yields a smooth alien creature instead of Claude Code's canonical 8-bit pixel-art Clawd). Look up or extract the real icon/SVG and pass the reference image path in `reference_image_urls` via `image_generate`.
- Keyless OAuth vs API image models: The bundled openai-codex OAuth provider routes to gpt-image-2 tiers (low/medium/high), not gpt-image-2.5 (Flare/Sunburst). Do not pass gpt-image-2.5 to openai-codex without an OPENAI_API_KEY configured for the openai provider. For maximum fidelity on OAuth, set image_gen.model: gpt-image-2-high. If image generation reports false credential expiration or a transient rate limit in the pool, run `hermes auth reset openai-codex && hermes auth refresh openai-codex` via terminal to clear the backoff state before re-authenticating.
- A plausible model name is not proof that it exists; verify the exact official release name and date first.
- Search snippets support only their literal text. Extract the official page before quoting values.
- “Best score,” effort setting, cost-per-task, and list price are different measurements; keep them distinct.
- Vendor benchmark charts are not independent head-to-head tests. Say whose charts they are and retain methodological caveats.
- An SVG chart can be present in the DOM while its points are not yet drawn; verify markers, not just the axes or accessible label.
- The source site may use tabbed or lazily rendered charts. Selecting the wrong tab or capturing before the animation completes yields a blank or misleading image.
- Browser screenshots may be saved with permissions that the host shell user cannot read. Copy/process them through the `hermes` container and store final PNGs under `$HERMES_HOME`.
- Do not use AI image generation to invent benchmark data. Use it only for decoration around verified source material.
- Drop topic immediately on user signal: Continuing to pitch LinkedIn hooks or draft posts after the user asks to change topic wastes turns and triggers repetition loops.
- Morning timing requires cron scheduling: Never publish immediately when asked for morning delivery; schedule an autonomous cron job for 08:30 IDT.
- Keep source links out of the body and add them in the first comment. `LINKEDIN_CREATE_LINKED_IN_POST` expects PNG file paths, not a PDF.
- Multiple LinkedIn connections can all be ACTIVE. Select the verified alias explicitly with `--account <alias>` on identity reads, creates, and comments; derive the author URN from that same account's identity response.
- LinkedIn’s managed Composio scopes are publish-only; do not promise post reads, inbox access, or analytics. Ask the user for an Analytics screenshot instead of retrying 403 reads.
- Inspect lazy-media metadata before diagnosing a missing attachment: guest posts may expose the image in `data-delayed-url` or `og:image` while `currentSrc` is still empty. Scroll the actual post-media element into view, wait outside the JavaScript evaluation, then count post images with `complete` and nonzero `naturalWidth`; exclude avatars and UI icons, because they can make an unloaded attachment look verified.

## Verification

A publish is verified only when all three checks pass: Composio returns `successful: true` and a valid post handle (`urn:li:share:...` or `urn:li:ugcPost:...`, including `data.x_restli_id`); the first-comment response succeeds and the exact comment is associated with that post on readback; and the public browser page shows the expected post text, source comment, and requested number of loaded images. Preserve every returned identifier literally: a comment response can normalize its object to a different activity URN; do not change prefixes or infer matching IDs. Use the page's authoritative canonical URL when available. Report the link and count only after that readback.