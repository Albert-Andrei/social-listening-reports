# Task: Social Listening Night Sync Report → GitHub Pages + Slack message

Gather all the information using mongodb mcp, mezmo mcp, sentry, and all possible logs about how
the social listening jobs performed this night (cron window 04:00–05:00 UTC; job logs
03:30–07:10 UTC). Compute and verify against production data:

- Cron runs: `socialListeningTopicSync` (04:00 UTC) and `socialListeningBrandSync` (05:00 UTC) —
  duration, keywords picked up, result payloads, staleness rules (topics 7d, brands 24h), keywords
  skipped and why, staging behavior.
- Headline stats: keywords synced, jobs spawned + completed, jobs failed / cancelled, new mentions
  inserted (KeywordMentions createdAt ≥ 03:30 UTC), API spend today UTC (KeywordDailyMetrics
  `costs.<platform>` buckets).
- Per-platform table: gathered (billed API records) vs inserted (new mentions), insert rate,
  keywords with new mentions, API calls, notes.
- Cost table per platform: cost, share, cost per inserted mention, pricing model.
- Spend by vendor: Bright Data / X API / OpenAI.
- Job pipeline: spawned / completed / failed / cancelled per job type, from Mezmo `[MiniJob]`
  Job started / Job done / Job failed counts, 03:30–07:10 UTC.
- Keyword digest (for the Slack message):
  - Total keywords tracked, and active keywords (have ≥1 KeywordListeners).
  - New keywords added in the last 24h: keyword name + estimated API spend so far (perCost).
  - Keywords with no apiCosts yet (not synced / zero spend) — list them by name.
  - Monthly spend: month-to-date total vs previous month total.

Let `<DATE>` be the report date in UTC (`YYYY-MM-DD`), `<YYYY>` the year, and `<mmm>` the
lowercase 3-letter month (e.g. `jul`).

## Artifact 1 — HTML report published to GitHub Pages

Reports repo: `https://github.com/Albert-Andrei/social-listening-reports` (branch `main`),
deployed via GitHub Pages at `https://albert-andrei.github.io/social-listening-reports/`.

1. Clone the reports repo (do NOT put the report inside the planable-app repo, and do not commit
   anything to planable-app).
2. Create the report at `<YYYY>/<mmm>/<DATE>/index.html` (e.g. `2026/jul/2026-07-24/index.html`).
3. Rendering requirements (self-contained single file):
   - Same sections in the same order: title + subtitle → success/failure callout → stat strip
     (5 big numbers) → "Cron runs" (2 cards + caption) → "Mentions: gathered vs inserted per
     platform" (chart + caption + table + caption) → "Cost breakdown per platform" (chart +
     table + caption) → "Spend by vendor" (chart + caption) → "Job pipeline" (table + caption).
   - Dark theme, flat design: near-black background (#1a1a1a-ish), light text, subtle 1px
     borders, rounded corners. NO gradients, NO box-shadows, NO external CSS/JS frameworks,
     no CDN links — everything inline or in a `<style>` block so the file works offline.
   - Charts rendered with inline SVG (no chart libraries):
     1. Grouped vertical bar chart "Mentions: gathered vs inserted per platform" (gathered =
        blue/info tone, inserted = green/success tone, with value labels).
     2. Horizontal bar chart "Cost breakdown per platform" (USD values as labels).
     3. Horizontal bar chart "Spend by vendor" (3 bars, USD values as labels).
   - Include the stat strip, the verdict callout, the two cron cards, all three data tables, and
     every small gray caption line.
   - Use the existing report `2026/jul/2026-07-07/index.html` in the repo as the visual/structural
     template — copy its markup and styles, replace the data.
4. Add a link to the new report at the top of the list in the root `index.html` (newest first),
   format: `<li><a href="<YYYY>/<mmm>/<DATE>/"><DATE> — Night Sync Report</a></li>`.
5. Commit with message `Add night sync report <DATE>` and push to `main`.
6. Verify deployment: poll
   `https://albert-andrei.github.io/social-listening-reports/<YYYY>/<mmm>/<DATE>/` until it
   returns HTTP 200 (Pages usually deploys in under 2 minutes; give up after 10 minutes and note
   the failure in the Slack message instead of the link).

## Artifact 2 — Slack message

Send via the `user-slack` MCP using `slack_send_message` with `channel_id: "U09A81M0N07"` (DM).
Do NOT upload or paste the HTML file — the report is delivered as the GitHub Pages link.

The message must be SHORT and scannable — a quick pulse check, not a report. All detail
(per-platform mentions, cost breakdown, job pipeline, cron runs, vendor split, keyword tables)
lives ONLY in the HTML report behind the link. Hard rules:

- Max ~8 lines, target under 700 characters. No code-block tables. No section headers.
- One line per idea, Slack mrkdwn (`*bold*`, `•` bullets, emoji).
- The "Needs attention" line appears ONLY when something actually needs a human look (failed
  jobs, stuck keywords, keywords with no apiCosts, odd cron timing, skipped/invalid vendor
  records). On a clean run, omit it entirely.
- Do NOT append any footer after the report link (no plan-file paths, no "Sent using …",
  no signatures).

Message shape (fill in real numbers):

```
🌙 *Social Listening — Night Sync (<DATE>)*
✅ Clean run — zero failures        ← or 🚨 one line saying exactly what broke
• Synced N of N tracked keywords (N active) · N jobs, 0 failed
• N new mentions · spend today $X.XX · MTD $X.XX (last month $Y.YY)
• New keywords (24h): N — name ($X.XX), name ($X.XX)        ← or "none"
⚠️ N keywords with no apiCosts yet: kw1, kw2, kw3        ← only when needed, else omit
🔗 Full report: https://albert-andrei.github.io/social-listening-reports/<YYYY>/<mmm>/<DATE>/
```

## Constraints

1. Do not post to any public/private channel (DM only). Do not publish anything to Linear.
2. No database writes.
3. Nothing committed to the planable-app repo — only the social-listening-reports repo.

## Acceptance checklist (verify before finishing)

1. Report exists in the repo at `<YYYY>/<mmm>/<DATE>/index.html`, pushed to `main`, dark flat
   theme, all 3 charts as inline SVG, all 3 tables present, numbers match the gathered data.
2. Root `index.html` links to the new report.
3. The GitHub Pages URL returns HTTP 200 and renders the report.
4. Slack DM sent to U09A81M0N07: max ~8 lines, no code-block tables, verdict + key numbers +
   new keywords + needs-attention (only if real) + the Pages link, no footer after the link.
