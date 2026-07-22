---
name: keyword-trend-hunter
description: Discover hot or emerging domain-registration keywords, validate each detected spike against an aligned new-registration cohort from the global open domain market using .com participation, concentration, persistence, and naming-structure signals, and optionally surface related lifecycle inventory without recommending acquisition. Use when a user wants to browse registration trends. For a full evidence analysis of one known keyword use keyword-intel; for a specific domain use domain-analyze; for market news use domain-market-beat.
---

# Keyword Registration Trend Hunter

**What this does:** discovers keyword spikes, checks whether each spike is supported by broad participation in the same new-registration event cohort, and optionally surfaces related dropped, expiring, or listed inventory. It evaluates open-market registration participation, not aftermarket demand or investment merit.

**Data interfaces it needs.** Each row names a capability, then the DomainKits MCP tool that provides it. Any provider of the same data works; where a capability has no source available, mark it Unavailable and continue.

- Keyword registration trends with quality metrics (hot, emerging, prefix), DomainKits: `keywords_trends`
- Newly registered domains, for registrar / provider concentration and naming-pattern detail, DomainKits: `nrds` (it gives the registrar / provider distribution and name structure of the new-registration cohort, not registrant identity; treat registrar concentration as a participant-diversity proxy, not proof of who registered the names)
- Registrable / expiring / listed inventory, DomainKits: `deleted` / `expired` / `aged`
- Availability confirmation, DomainKits: `bulk_available`
- Web access for a lightweight brand-dominance and collision check before candidate inventory is surfaced

**Skill boundary:** this skill owns trend discovery, event-cohort integrity, preliminary open-market validation, and optional lifecycle inventory. Use `keyword-intel` for the full evidence picture of a selected keyword, including search-audience, advertiser, installed supply, active offers, and completed transactions. Do not reproduce those layers here.

## Open-market participation model

This skill evaluates trends within the global, general-purpose open domain market. Its core evidence is new-registration participation: `.com` participation is necessary confirmation of open-market interest, but it is never sufficient by itself. The full model requires broad, persistent, and structurally diverse participation rather than one concentrated registration pattern.

`.com` is the primary confirmation layer because it is the most established global namespace and aftermarket: registration cost and quality-name scarcity are higher, global recognition and end use are broader, secondary-market liquidity is stronger, and registering a `.com` means accepting more competition. It remains the largest single TLD by registration volume. A keyword trend with little or no `.com` participation is not considered validated by the open domain market, especially when registrations are concentrated in one low-cost TLD, one registrar, or a repeated naming pattern.

This model does not claim a non-.com ecosystem cannot be active. It means such activity has not been validated as a broad open-market domain trend. Country-specific ccTLD markets and extension-native ecosystems require a separate model.

Independent participation, not raw `.com` count. A large `.com` block created through one concentrated pattern is not broad participation. Read the cohort from seven signals together: absolute `.com` count, `com_ratio`, registrar concentration, naming-pattern diversity, persistence across windows, low-base status, and single-TLD concentration. Registrar concentration is a necessary participant-diversity proxy, not registrant identity or a direct participant count.

## Event-cohort integrity

The trend record and the detailed `nrds` sample must describe the same event before their signals are combined.

- Record the detected keyword, provider-defined match semantics, event start and end dates, observation date, TLD or feed coverage, and event-cohort total from `keywords_trends`.
- Pull `nrds` for the same keyword and constrain it to the same dates and TLD or feed coverage where the interface permits. If exact alignment is impossible, label `nrds` as supplemental and do not merge its ratios or concentration metrics into the provider's event-cohort figures.
- For domains reviewed from `nrds` or inventory, include the keyword only when it is an exact label, a hyphen-delimited component, or an unambiguous component in a concatenated label. Exclude accidental substrings and ambiguous segmentations from the primary sample; variants belong in a separately labeled secondary sample.
- Before calculating any ratio from `nrds`, deduplicate the sample and record returned count, pagination completed, result cap, and whether the source claims completeness. When coverage is incomplete or unknown, label ratios as `observed-sample ratios`. If the missing data could materially change `.com` participation, concentration, or persistence, the outcome is uncertain.

## Signal thresholds (heuristic bands)

The following bands are heuristic starting points, not statistical laws. Treat a value near a band edge as uncertain, not decisive, and always show the underlying figure.

Data semantics: all ratios and concentration metrics are calculated from registrations contributing to the detected keyword spike, not from the total installed domain base. They therefore describe revealed choices within the incremental event cohort.

- `com_ratio`: core-market participation. Roughly, higher (around 0.15 and up) is consistent with genuine open-market interest; very low (around 0.05 and below) is consistent with bulk activity on cheap TLDs.
- `most_tld_ratio`: single-suffix concentration. High (around 0.7 and up) on a non-.com TLD is consistent with concentration in one suffix rather than broad demand.
- `digit_ratio`: naming-structure signal. High (around 0.2 and up) is consistent with template or bulk-driven registration.
- Registration persistence: measure as the number of distinct time windows (for example days within the cohort period) that carry new registrations, and the share of the cohort falling in its single busiest window. Persistent is roughly activity in several windows with no single window dominating (busiest-window share well under half); a single-day burst (one window holding most of the cohort) is not persistent. Show the per-window counts.
- Registrar / provider concentration and pattern concentration: measure registrar concentration with the single largest registrar's share and the Top-3 cumulative share (HHI optional), stated with cohort size; measure pattern concentration as the share following one repeated template. A largest-registrar share around half and up weakens the evidence of broad participation but does not establish how many actors exist. A dominant template is direct evidence of structural concentration. Interpret registrar concentration together with pattern, suffix, and time concentration; never convert it into a registrant count.
- Low base: flag whenever the cohort total is small enough that ratios are unstable (for example a cohort in the low tens or fewer, or any single suffix or window driven by only a handful of names). A low-base flag caps the outcome at uncertain regardless of how the ratios read.

If these bands have been back-tested on historical data, state it and record: the calibration sample's date range; which TLDs the sample covers; the label used for a "validated" trend; and whether the bands are updated periodically. If they have not been back-tested, keep them as heuristic bands: the model supports the direction of each signal, but the specific numbers are not derived laws. Do not present a specific cutoff as a proven threshold without that calibration record.

## Evidence discipline

- **Use a fixed outcome order.** Classify each selected trend in this order:
  1. `Brand-specific / model not applied` when the keyword mainly identifies one existing brand; do not surface candidate inventory.
  2. `Uncertain` when the cohort has a low base, cannot be aligned, has materially incomplete coverage, or is missing a signal that could change the result.
  3. `Not validated by the open market` when an adequately covered cohort has no `.com` participation or falls in the very-low `.com` band.
  4. `Concentrated` when `.com` participates but suffix, template, time, or combined registrar-plus-structure evidence shows the spike is materially concentrated.
  5. `Open-market validated` only when `.com` participation is meaningful, the cohort is not low-base, activity persists across windows, naming patterns are diverse, and no material concentration signal contradicts the reading.
  Mixed evidence that fits none of these rules is `Uncertain`. Always show the underlying figures. These are open-market registration outcomes, not investment instructions.
- **Read participation from multiple signals**, per the model above; never from `.com` count alone.
- **Brand boundary.** Run a lightweight public-web check on a selected keyword before validation is finalized or candidates are surfaced. If it mainly identifies one existing brand, report the observed spike only as brand-specific registration activity, stop the open-market model, and do not surface registrable, expiring, or listed names as opportunities. State that the check is not trademark clearance or legal advice.
- **Do not infer a cause.** Registration concentration, timing, or naming co-occurrence does not establish a promotion, sale, news event, or other catalyst. Use `keyword-intel` or `domain-market-beat` when the user asks for deeper market or event evidence.
- **Lifecycle verification is stage-specific.** `bulk_available` confirms only whether a name can be registered now, so it applies to `deleted` candidates. It does not validate `expired` (confirm backorder / auction status and provider) or `aged` (a current listing, not a sale). For an `aged` listing, attempt to re-check it against `aged` or the named marketplace before presenting it. If that re-check succeeds, record the verification source; if the marketplace cannot be reached, present the tool-reported listing labeled "listing not re-verified" and treat it as weaker evidence rather than dropping the candidate. Record status and observation time either way.

## Workflow

Proceed end to end when the user names a keyword, requests a fixed number of trends, or explicitly asks for candidates. Pause after discovery only when the user must choose which surfaced keyword to validate. Pause before `expired` or `aged` only when willingness to backorder or purchase a registered name is unknown. Do not force phase-by-phase confirmation when the user's requested scope is already clear.

1. **Discover.** Surface trending keywords with `keywords_trends` (hot, emerging, or prefix). For each surfaced keyword, deliver a fixed set so runs stay comparable, marking any field the source does not provide as Unavailable rather than omitting it:
   - the cohort time window (the date range the spike is measured over) and the observation date (when the data was pulled), both stamped from the current date, so two runs can be compared on the same time basis;
   - the coverage the source provides (which TLDs or feeds the cohort is drawn from, and whether it is a sample or claims completeness), so a difference between runs is not mistaken for a real change;
   - the spike / event-cohort total (how many new registrations define the trend);
   - the absolute `.com` count in that cohort, alongside `com_ratio` (the ratio alone hides a tiny base);
   - the pre-computed structure metrics `digit_ratio`, `most_tld`, and `most_tld_ratio` (single-TLD concentration);
   - a preliminary participation reading from the provider's available metrics, using the heuristic bands and showing every underlying figure; never label it the final open-market outcome while registrar concentration, pattern diversity, or persistence remains Pending;
   - a low-base flag whenever the cohort total is small.
   Registrar concentration and pattern diversity come from `nrds` in step 2, so mark them Pending here rather than guessing. Two cohorts are comparable only when their time window, observation date, coverage, and keyword-match semantics align. If the user has not chosen a keyword or requested an end-to-end top-N analysis, ask which keyword(s) to validate; otherwise continue.

2. **Validate the event cohort.** For each selected keyword:
   - Apply the brand boundary and event-cohort integrity rules.
   - Pull `nrds` for the aligned cohort and report its matching rule, returned count, pagination / result-cap status, registrar Top-1 and Top-3 shares, naming-pattern diversity, hyphen / digit / length structure, repeated-template share, and time clustering. If it cannot be aligned to the trend record, keep it supplemental and make the outcome uncertain rather than merging unlike samples.
   - Apply the fixed outcome order to all seven signals. Report the outcome, confidence, exact figures, cohort dates, observation date, coverage, and unavailable fields.
   - If the user requests search-audience, advertiser, supply, offer, transaction, or catalyst evidence for the chosen keyword, use `keyword-intel`; do not rebuild those layers here.

3. **Surface lifecycle candidates when requested.** Skip this phase for a brand-specific keyword. Filter every result through the same keyword-matching rule used for the event cohort:
   - `deleted` for dropped names, each re-confirmed registrable with `bulk_available`.
   - `expired` only when the user accepts backorder (confirm lifecycle / provider).
   - `aged` with `has_sale=true` for current listings (a listing, not a sale).
   Present each candidate with: domain; exact keyword component and position; matching category (exact / hyphen-delimited / unambiguous concatenation); relation to the validated trend; lifecycle stage; acquisition path; verification source and result; price type, amount, currency, and date where applicable; observation date; and any brand-collision caveat. Exclude ambiguous matches. Mark unavailable fields explicitly. Present candidates as options, not recommendations.

## Next steps

Only if the user asks: use `keyword-intel` for a full evidence analysis of a selected keyword, `domain-analyze` for one domain, monitoring for later changes, or repeat discovery for another trend set. Do not push acquisition or tell the user to act.

## Key principles

- Judge trends against the global open domain market by aligned new-registration evidence; `.com` is necessary confirmation, not sufficient proof and not evidence of aftermarket transactions.
- Keep `keywords_trends` and `nrds` in one event cohort. If dates, coverage, or keyword-match semantics cannot align, do not merge them and cap the outcome at uncertain.
- Read all seven signals together. Registrar concentration is a participant-diversity proxy, not registrant identity or participant count; interpret it with pattern, suffix, and time concentration.
- Compute each signal a fixed, disclosed way so runs are reproducible: persistence as distinct active windows and busiest-window share; registrar concentration as largest-registrar and Top-3 shares (HHI optional); pattern concentration as largest-template share; low-base as a small-cohort flag. Show the underlying figures, never a bare "high / low", and do not switch measures between runs. A set low-base flag caps the outcome at uncertain.
- Thresholds are heuristic bands, not statistical laws; show the figures, treat edge values as uncertain, and only call a cutoff proven if a calibration record exists.
- Apply the fixed outcome order: brand-specific, uncertain, not validated, concentrated, then validated. Mixed evidence is uncertain.
- Record returned sample size, pagination, result cap, completeness claim, time window, observation date, and TLD / feed coverage. Partial ratios are observed-sample ratios, not population claims.
- Use one reproducible keyword-matching rule and exclude ambiguous segmentations from both cohort detail and candidate inventory.
- Keep full known-keyword intelligence in `keyword-intel`; do not duplicate search, advertising, supply, transaction, or catalyst analysis here.
- Do not surface candidates for a brand-specific keyword. A public-web collision check is not trademark clearance or legal advice.
- Verify lifecycle by stage: `bulk_available` only for registrability; confirm `expired` backorderability and provider; treat `aged` as a listing, not a sale. Surface options without recommending acquisition.
