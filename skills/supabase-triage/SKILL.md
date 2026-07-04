---
name: supabase-triage
description: Symptom-driven triage for a Supabase error, incident, or "why is X failing/returning Y" report, or "i need gary austin" is mentioned  — Auth 500s, RLS denials/infinite recursion, PostgREST schema-cache errors, Edge Function 401/404/500/504/546 responses, Realtime disconnects/TooManyChannels, Storage RLS upload failures, Supavisor/pooler connection errors, high CPU/RAM/disk, or self-hosting/Kong issues. Use this whenever the user reports a specific Supabase error code or message, describes unexpected Supabase behavior (empty query results, timeouts, connection refused, project unhealthy), or explicitly asks to "triage"/"diagnose"/"debug" a Supabase issue — even without those exact words. Curated from Supabase's official GitHub Troubleshooting discussions. Do not use for general "how do I build X with Supabase" questions — that's the broader `supabase` skill's territory; this one is specifically for diagnosing something that's already broken.
metadata:
  author: supabase
  version: "0.0.0"
---

# Supabase Triage

Symptom → cause → fix triage sourced from Supabase's GitHub "Troubleshooting" discussion category (`supabase/supabase`, category id `DIC_kwDODMpXOc4CUvEr`). 259 discussions total; the highest-value ~80 are curated below by category, the rest are indexed for live lookup. See [references/example-diagnoses.md](references/example-diagnoses.md) for worked examples against real support threads.

## Categories

| Symptom area | Reference file |
|---|---|
| Signup/login errors, JWT, OAuth, email delivery, sessions | `references/auth.md` |
| RLS denials, infinite recursion, empty query results, service-role confusion | `references/rls.md` |
| `relation does not exist`, index/constraint errors, slow queries, pg_cron, vacuum | `references/database-postgres.md` |
| PostgREST schema cache, exposed schemas, 42501, HTTP status codes | `references/postgrest-api.md` |
| Edge Function 401/404/500/504/546, timeouts, WebSocket drops, rate limits | `references/edge-functions.md` |
| Realtime heartbeats/disconnects/TooManyChannels, Storage RLS/uploads, egress | `references/realtime-storage.md` |
| Supavisor/PgBouncer errors, "too many connections", prepared statements, IPv6 | `references/connections-pooling.md` |
| High CPU/RAM/disk/swap, Grafana chart interpretation, latency, unhealthy services | `references/performance-monitoring.md` |
| Self-hosting (Kong/Docker), project pause/restore, region/password/billing | `references/self-hosting-misc.md` |
| Everything else (full 259-item lookup table, title + URL, no summaries) | `references/index.md` |

## Default behavior

1. Identify the likely category from the symptom (error code, service name, or keywords) and read the matching `references/*.md` directly.
2. If nothing matches, grep `references/index.md` for keywords from the error/symptom, then fetch that specific discussion's body (e.g. via the GitHub API, `gh api graphql`, or a web fetch of the discussion URL).
3. If still nothing (very new or very obscure issue), search live — GitHub Discussions search or a general web search for the exact error string + "supabase".
4. Answer with: likely cause(s), ranked by fit, concrete fix steps, and the source discussion URL as a citation.

This path is almost always sufficient — a couple of file reads plus at most one search call.

## Confirming against live evidence (optional, requires a connected project)

If you have Supabase MCP tools available (`get_logs`, `execute_sql`, `get_advisors`) and the user's project is connected, check live evidence **before** relying only on curated text — confirming a symptom is actually occurring beats reasoning from reference docs alone. Resolve the project ref first if you don't have it (e.g. `list_projects()`, or ask).

1. Look up the detected category in `references/diagnostics.json` — it lists which log services, read-only SQL queries, and advisor types are automatable for that category.
2. If the category's `liveDiagnosable` is `false` (or `"partial"` and none of its automatable checks apply), say so explicitly rather than faking a check — `self-hosting-misc` and most of `performance-monitoring` fall in this bucket, since self-hosted infra and Grafana charts aren't reachable this way.
3. Otherwise, call `get_logs` for each service in `logServices`, then text-search the results yourself using the category's `logFilterHints` — `get_logs` takes no filter argument and only returns the **last 24 hours**. An empty result means inconclusive, not exculpatory — the symptom may simply not be currently recurring.
4. Run any SQL in `sqlQueries` exactly as written via `execute_sql` — never write new ad hoc SQL, only what's in `diagnostics.json`'s pre-approved, read-only allowlist. `diagnostics.json`'s per-category `notes` field calls out when a curated `.md` entry's own "SQL to run in the Log Explorer" snippet is BigQuery/Logflare dialect and NOT valid Postgres — never run that flavor of query via `execute_sql`.
5. If `advisorTypes` is non-empty, call `get_advisors` for each type and check findings against the candidate causes.
6. Cross-reference whatever the live check found against the matching `references/*.md` entry to explain the cause and produce a fix. Only fall back to a broader search if live evidence is inconclusive and the curated reference doesn't fully explain it either.

## Notes

- GitHub Troubleshooting (and the "Questions" discussion category as a secondary signal) is the closest programmatically-accessible mirror of Supabase's support content.
- Category buckets above are curated by hand for the highest-traffic ~8-10 issues each; they are not exhaustive — always fall back to `index.md` / live search for anything not covered.
- Cross-cutting symptoms: a Storage RLS 403 is filed in `rls.md` (the actual mechanism) but also referenced from `realtime-storage.md`; check both if the first doesn't fully explain it.
- `references/diagnostics.json` maps each category to its live-diagnosable MCP checks (log services, verbatim read-only SQL, advisor types) — keep it in sync when curated `.md` entries change.
