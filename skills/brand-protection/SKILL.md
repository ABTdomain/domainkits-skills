---
name: brand-protection
description: Scan for typosquats and lookalike registrations around a brand, assess each domain individually, surface unconfirmed defensive candidates, and offer monitoring. Not for single-domain analysis (use domain-analyze), general discovery, or keyword trends.
---

# Brand Protection

Scan the domain landscape around a brand, evaluate each similar registration individually, and surface unconfirmed defensive candidates.

**Registration is not infringement.** A domain matching a typosquat pattern is a structural similarity, not proof of abuse. Many such domains are legitimate (common words, unrelated businesses, the brand's own portfolio). This skill surfaces domains worth reviewing. It does not determine legality or intent. Use neutral language ("registered variant", "matches pattern") throughout.

## Tools

**Core:** `typosquat` (requires full domain), `nrds`, `tld_check`, `monitor`, `preferences`, `usage`, `safety`

**Supporting (per-domain, conditional):** `whois`, `dns`, `available`, `bulk_available`

## Input

- Domain (e.g. `acmebank.com`): prefix = brand term, domain used directly for `typosquat`.
- Keyword only (e.g. `acme`): use for `nrds` and `tld_check`. Ask user to confirm primary domain before calling `typosquat`. Do not guess.

## Evidence rules

- `typosquat` returns variants with an optional `registered_date`. WITH `registered_date` = confirmed registered. WITHOUT = unknown status (not "available", not confirmed registered). For unknown-status variants worth investigating, call `whois` or `available` to resolve.
- `prefix_tld_count` is a current count, not a time series. Cannot prove bulk registration alone.
- `tld_check` returns `might_available`, not confirmed availability. Verify with `available` or `bulk_available` before recommending defensive registration.
- WHOIS privacy is neutral, not a risk signal.
- `nrds` returns domain and registration date. TLD is parsed from the domain string, not a separate field. `nrds` does NOT return registrar. Registrar data requires `whois`.
- `nrds` returns paginated results. The first page is not the full dataset. Note `total_found` from the response and state how many pages were checked vs. total available. Do not present one page as the complete picture.
- Mutation type (omission, transposition, etc.) is a fact from `typosquat`. Visual similarity to the brand is a model judgment, not tool output. Classify it as inference, not fact.
- `safety` returns `safe.is_safe` and `safe.threat_types`. A hit is a detection signal, not a legal conclusion; a flagged domain may be benign (false positive, shared hosting, stale blocklist). `is_safe: true` is not confirmed safe (coverage gaps, new threats, evasion). Report hits with the specific `threat_types` returned.

## Workflow

One tool failing marks that section Unavailable. Continue with the rest.

### Phase 1: Surface scan (parallel)

1. **Typosquat.** Call `typosquat` with the primary domain. Collect variants that have `registered_date` (confirmed registered). Note variants without `registered_date` but with high `prefix_tld_count` as unknown-status for possible follow-up.

2. **Recent registrations.** Call `nrds` with brand keyword, `position: contain`, sort `reg_date_desc`. Note `total_found` and how many results were reviewed (page 1 = first 10). If total_found is large, state that only the first page was checked. Note any clusters of registrations on the same date as context (date clustering alone is weak evidence).

3. **Cross-TLD footprint.** Call `tld_check` with brand prefix. Returns per-TLD status for core TLDs only (com, net, org, io, ai, de): `registered`, `expiring`, `for_sale`, or `might_available`. Aggregate counts (`count`, `gtlds_count`, `cctlds_count`) cover a wider TLD set but carry no per-TLD detail.

### Phase 2: Per-domain evaluation

Typosquat can return hundreds of variants. Before any lookups, determine budget.

**Step 1: Check quota.** Call `usage` first. For each tool group used in this phase (`whois`, `dns`, `available`, `safety`, plus `bulk_available` if defensive candidates need verification), read `daily_remaining` and `rate_per_minute`. When a group shows `daily_unlimited: true`, `daily_remaining` is absent; treat that group as uncapped. Investigation budget = `min(daily_remaining)` across `whois` and `dns` (the two mandatory per-domain tools), or 10 when both are uncapped. Cap at 10 without user confirmation.

**Step 2: Triage.** Select confirmed-registered variants by priority: recent `registered_date`, high `prefix_tld_count`, close mutation type (e.g. single-character omission or homoglyph substitution are closer mutations than TLD-swap). Visual similarity is an inference, not a fact; label it accordingly. State total found. Limit selection to the budget from Step 1.

**Step 3: Resolve unknowns (within budget).** For unknown-status variants that look high-priority, call `whois` or `available` to determine registration status. These calls count against the same budget. Only resolve unknowns if budget remains after reserving slots for confirmed-registered variants.

- If a call is rate-limited (error response), stop further calls for that tool. Report what was completed and list remaining domains as not investigated.
- State the quota situation in the report. If the user wants deeper coverage, mention that higher tiers allow more lookups.

For each selected domain:

1. **WHOIS.** Call `whois`. Extract registration date, registrar, expiry, nameservers. Report as facts.
2. **DNS.** Call `dns`. Extract A/AAAA/CNAME records, MX, NS. Report as facts. DNS alone cannot reliably distinguish parking pages from active sites (a domain with no A record may have AAAA or CNAME; parking IPs are not enumerated here). Treat DNS as context, not classification.
3. **Safety.** Call `safety` if available on the user's tier. Report the returned `threat_types` as detection signals. A hit does not confirm malice; no hit does not confirm safety. If `safety` is unavailable (guest tier, quota exhausted), note it in the report and continue.
4. **Assessment.** Note mutation type as fact. Note visual similarity as inference. Note registration timing as fact. Note safety flags as detection signals. Do not infer intent.

### Phase 3: Report and next steps

**Report structure:**

1. **Summary.** Variants scanned, confirmed registered, investigated, defensive candidates. One paragraph.
2. **Registered variants.** Per-domain data: mutation type, registered_date, registrar (from whois), nameservers, DNS records, safety flags (if checked). Neutral framing. These are domains worth monitoring, not confirmed infringers.
3. **Recent registrations.** NRDs containing brand term from `nrds`, with dates. State total_found and pages checked.
4. **Unconfirmed defensive candidates.** TLDs where `tld_check` returned `might_available` (core TLDs only). Ranked by importance (.com/.net/.org first). Availability is unconfirmed; offer to verify with `available`/`bulk_available`.
5. **Not investigated.** How many variants were skipped, why (quota, unknown status, lower priority).
6. **Unavailable evidence.** Any tools that failed or returned errors during the scan. State which section is affected and what data is missing.

**Monitoring offer.** After the report, offer to set up `monitor` on registered variants the user wants to track.

- Check `tier` from the Phase 2 `usage` response (call `usage` if it was not called). If tier is `guest`, monitoring is not available. The report is the full deliverable. Mention that a registered account unlocks monitoring. Do not call `preferences` for guests.
- For registered users (any non-guest tier): call `preferences action:get` to check memory status. Current monitor count is `monitors_count` (0 if absent). The tier limit comes from the `usage` response field `monitor.max_items` (`monitors_summary.max` in preferences appears only when at least one monitor exists). Remaining slots = `monitor.max_items` - `monitors_count`. If 0 slots remain, do not offer monitoring; inform the user their monitor slots are full and mention upgrade options.
- If memory is not enabled, ask the user: "Would you like me to enable memory so I can set up monitoring? I'll call `preferences action:set memory_enabled:true`." Only proceed with their consent.
- Create: `monitor` with `action: set`, `domain: <domain>`, `tools: whois,dns`, `note: <context>`. Only offer to add up to the number of remaining slots. If the user selects more domains than available slots, add up to the limit and list the rest as "not added, monitor slots full".
- Default monitor tools are `whois,dns` only.
- Page tracking (`web_fetch`) requires separate user consent. Before enabling: explain that the AI client will fetch the page content and store a text snapshot. The fetch must be done in a read-only manner: no form submissions, no credential input, no file downloads. Before any `web_fetch`, run `safety` first. If `safety` flags the domain, skip `web_fetch` and note why. If `safety` is unavailable (quota exhausted, guest tier), skip `web_fetch` and note that safety could not be verified.
- Monitor is on-demand: `action: get` triggers checks. It does not run in the background.

## Limitations

- Typosquat covers common mutations (omission, transposition, keyboard-adjacent replacement, insertion, repetition, hyphenation, homoglyph, vowel-swap, plural, TLD-swap). IDN homograph and multi-language variants are not covered.
- `tld_check` returns per-TLD status for six core TLDs (com, net, org, io, ai, de). Aggregate counts cover more TLDs, but per-TLD status beyond the core set is not returned.
- This skill evaluates domains individually. It cannot determine whether multiple domains belong to the same registrant.
- Visual similarity is a model inference, not a measured value. Different models may assess it differently.
