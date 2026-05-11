---
name: slack-triage
description: >-
  Pull recent Slack messages from a specific person (boss, teammate, customer)
  and cross-reference Rafael's own replies/commits to surface what's still open
  vs already addressed. Output is a precise, terse action list (P0/P1/Open Q/
  Already handled). Use when Rafael says "what did X say", "check feedback from",
  "what do I need to tackle", "boss feedback", or any review of someone's Slack
  activity over a time window. Bullet-only, no prose, no recap, no praise.
---

# Slack Triage (personalized for Rafael)

Pull person's recent Slack messages, cross-reference Rafael's own replies, output terse action list with status.

## Rafael's identity

- Name: Rafael Fonseca
- Slack ID: `U05NCSS3PKQ`
- Email: rafael@sandstormapp.com
- Use this ID to filter Rafael's own replies in the same threads/channels.

## Inputs needed

Ask only if missing:
- person (name or user ID) — common: Matt Hoffman = `U07CALH0BLP`
- channel (name, ID, or "all") — common: `#engineering` = `C08Q66EHHFC`
- window (e.g. "last 3 days" → resolve to absolute date)

## Steps

1. Resolve target user ID via `slack_search_users` if name given.
2. Resolve channel ID via `slack_search_channels` if name given. Skip if ID provided or "all".
3. Pull target's messages:
   - Channel: `from:<@TARGETID> in:<#CHANNELID> after:YYYY-MM-DD`
   - All: `from:<@TARGETID> after:YYYY-MM-DD`
   - Use `slack_search_public_and_private`, `sort=timestamp`, `sort_dir=desc`, `response_format=concise`, `include_context=false`
4. Paginate until messages older than window OR 5 pages.
5. Pull Rafael's messages same window+channel: `from:<@U05NCSS3PKQ> in:<#CHANNELID> after:YYYY-MM-DD`. Same params.
6. Pull threads where both appear — for top P0 candidates, use `slack_read_thread` to see if Rafael already responded/resolved.
7. Cross-reference signals for "already handled":
   - Rafael's reply contains "fixed", "shipped", "merged", "done", "PR #", or commit SHA
   - Rafael's reply links to a PR/commit
   - Optional: `git log --author=rafael --since=YYYY-MM-DD --oneline` in cwd to match Rafael's commits to topics
8. Files: target's `content_types=files` query. Note count + dates only. Don't fetch bytes (Slack MCP auth wall).
9. Triage into buckets:
   - **P0** = bugs, blockers, ban-risk, repeated complaints — NOT yet handled
   - **P1** = UX asks, perf, polish — NOT yet handled
   - **Open Q** = unresolved questions target asked Rafael
   - **Already handled** = items Rafael's replies/commits show resolved

## Output rules

- Bullet only. Fragments OK. No grammar formality.
- One line per task. ≤ 10 words ideal.
- No "Summary", no "Top 3", no "Want me to...", no recap.
- Group under `## P0` / `## P1` / `## Open Q` headers only.
- Skip empty sections entirely.
- If screenshots exist: one line at bottom: `Screenshots: N (auth wall — paste key ones if you want OCR)`

## Anti-patterns

- ❌ Verbose triage with explanations per item
- ❌ Quoting full message text — paraphrase to action
- ❌ Adding meta-commentary, offers, follow-ups
- ❌ Repeating same item across buckets
- ❌ Listing every "lol" / reaction / one-word reply

## Example output

```
## P0
- persist chats across deploys
- Meta API rate limit (acct ban risk)

## P1
- 20s cold start

## Open Q
- billing hooked up?

## Already handled
- chat UUID in URL — shipped (PR #1027)
- copy button for result — committed c818a971b

Screenshots: 20 (auth wall — paste key ones if you want OCR)
```
