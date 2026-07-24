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

Slack does not render markdown tables — use Slack mrkdwn: `*bold*` section headers with a leading
emoji, and tables as fixed-width aligned text inside triple-backtick code blocks (pad columns with
spaces so they align in monospace). Stay under ~4000 chars; if needed, trim the notes columns from
tables, never the numbers.

Message shape (fill in real numbers):

```
🌙 *Social Listening — Night Sync Report (<DATE>)*
window: cron 04:00–05:00 UTC · job logs 03:30–07:10 UTC

*Summary*
✅ Clean run — zero failures        ← or 🚨 verdict if anything failed
• Keywords synced: N
• Jobs spawned + completed: N
• Jobs failed / cancelled: N
• New mentions inserted: N
• API spend (today, UTC): $X.XX

*Needs attention*
• Nothing — no failed/cancelled/timed-out jobs, no keywords stuck in `syncing`
  ← or list every anomaly: failed jobs, stuck keywords, missing costs, cron misses

📊 *Mentions per platform*
```(code block: Platform | Gathered | Inserted | Rate | Kw | Calls + Total row)```

💰 *Cost breakdown*
```(code block: Source | Cost | Share | $/mention + Total row)```

⚙️ *Job pipeline*
```(code block: Job type | Spawned | Completed | Failed | Cancelled)```

🕐 *Cron runs*
• socialListeningTopicSync — 04:00 UTC, X.Xs, { keywordsProcessed: N }
• socialListeningBrandSync — 05:00 UTC, X.Xs, { keywordsProcessed: N }

🏦 *Spend by vendor*
• Bright Data $X.XX (NN%) · X API $X.XX (NN%) · OpenAI $X.XX (N%)

🔗 Full visual report: https://albert-andrei.github.io/social-listening-reports/<YYYY>/<mmm>/<DATE>/
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
4. Slack DM sent to U09A81M0N07 with emoji headers, aligned code-block tables, and the Pages link.
