---
name: domain-name-advisor
description: Run a naming and acquisition-path consultation and present a small verified shortlist in either of two modes. Project mode turns vague or open-ended requirements into a generator-ready brief, delegates creative generation to domain-generator, and adds lifecycle inventory when appropriate. Taken-target mode extracts what the user values in an unavailable target, delegates creative variants to domain-generator, and searches dropped, expiring, for-sale, or brokered alternatives. Use when a user needs naming direction or recommendations across acquisition paths. Do not use for a simple availability check (use available / bulk_available), creative variations from an already-defined seed with no lifecycle comparison (use domain-generator), or analysis of a domain itself (use domain-analyze).
---

# Domain Name Advisor

**What this does:** consults on naming and presents a small verified shortlist, in one of two modes:

- **Project mode** (no target yet): briefly diagnose needs, prepare a generator-ready project brief, delegate all creative candidate work to domain-generator, and add lifecycle options when appropriate.
- **Taken-target mode** (a specific target is already taken): confirm the target is actually unavailable, extract what the user values in it, delegate all creative variant work to domain-generator, and search relevant lifecycle inventory without re-running an open naming consultation.

Both modes converge on the same cross-channel comparison and delivery. Most candidates are registrable names verified as available at check time; when the user accepts the relevant acquisition paths, the shortlist may also include already-registered candidates (expired / backorder or for-sale / aged), which are verified by lifecycle and acquisition status, not as "available".

**Data interfaces it needs.** Each row names a capability, then the DomainKits MCP tool that provides it. Any provider of the same data works; where a capability has no source available, mark it Unavailable and continue.

- Availability check with pricing, DomainKits: `bulk_available`
- Cross-TLD registration data (a lightweight crowding signal), DomainKits: `tld_check`
- Web access for a lightweight public-web check on obvious brand collisions; for confirming a named broker before a contact-owner candidate is offered (a domain with a public marketplace listing is a purchase candidate, handled via `aged`, not contact-owner); and, optionally, for a past sale price only from a named verifiable sales record. There is no dedicated sales-history source here, so the historical-sale field is omitted when no named source is found, never estimated.

**Skill dependency:** use `domain-generator` as the sole creative-generation component. This advisor owns project or target diagnosis, generator handoff, lifecycle sourcing, cross-channel comparison, and final recommendation. Domain-generator owns the creative frame, semantic distances, naming territories, construction methods, intrinsic filtering, and creative iteration. Do not reproduce or maintain any of those rules here.

Lifecycle acquisition capabilities (a primary source in taken-target mode; in project mode used only as expansion when the registrable shortlist is thin, and scoped to what the user will accept):
- `deleted`: dropped names that may be registrable; re-confirm every candidate with `bulk_available`.
- `expired`: only when the user accepts backorder; verify lifecycle, backorder / auction status, and provider (not `bulk_available`).
- `aged`: only when the user accepts purchasing a registered domain; verify current listing and asking price (not `bulk_available`).

## Evidence discipline

- **Availability is point-in-time, and applies only to registrable candidates.** Present a registrable candidate as "available at check time" with an observation timestamp; it may not be available when the user actually registers. An already-registered candidate (expired / backorder, or for-sale / aged) is never labeled "available": verify and label it by its own status (current lifecycle and backorder / auction status for expired; current listing and asking price for aged), each with an observation timestamp. Do not run `bulk_available` on an already-registered candidate; it measures registrability only.
- **Separate facts from judgment.** Source factual claims (availability, price, registration data). Present naming-quality judgments as reasoned creative assessments, not objective facts.
- **Price is data, and its shape depends on the acquisition path.** For standard registration, report standard vs premium status, first-year and renewal price, currency, provider, and check date. For a backorder candidate, report the backorder fee and current bid where shown (first-year / renewal applies only once caught). For an aged listing, report the asking price and listing type. Never carry standard-registration first-year / renewal fields onto an already-registered candidate.
- **A lightweight public-web check may catch obvious existing brands, but it is not trademark clearance. Never describe a candidate as legally safe.**

## Workflow

Use what the user has already told you (project, audience, style, TLD, or a specific target) and proceed. Ask at most one concise question at a time, and only when a missing detail would materially change the naming direction. Do not force a full diagnosis when the brief is already clear.

**Determine the mode first.** If the user has a specific taken target they want to replace, use the taken-target entry (step 1T). Otherwise use the project entry (steps 1 to 2). Both entries then hand creative work to domain-generator and use this advisor for lifecycle sourcing, comparison, and delivery.

1. **Brief (project mode).** Establish the essentials: project or industry, target audience, and what the name is for. If those are already given, move on. Ask a single question only when the answer changes direction (for example, tone, or whether the user is committed to `.com`).

1T. **Understand the target (taken-target mode).** Confirm the target is actually unavailable, unless a fresh availability result already exists in the conversation; do not search for alternatives to a domain that may still be registrable. Extract plausible keyword and pattern interpretations (for example, `getflow.com` may be prefix + root) and record what the user values in it: literal word, meaning, length, sound, rhythm, image, construction, or brevity. Record TLD scope and willingness to backorder or buy a registered domain. Then go to step 3; domain-generator will turn this target brief into a creative frame.

2. **Prepare the generator handoff (project mode).** Turn the diagnosis into a compact brief containing the project or category, audience, use case, positioning or desired outcome, desired brand qualities, user-supplied words or concepts, literal constraints, TLD scope, language or regional constraints, and what may transform. Preserve any direction the user has chosen. Do not invent candidates, semantic distances, naming territories, construction methods, or filtering rules; domain-generator determines those from the brief.

3. **Use domain-generator for the creative channel.** In project mode, pass the generator-ready brief; its project or category, positioning, or user-supplied concept serves as the conceptual input. In taken-target mode, pass the full target plus the valued literal, semantic, phonetic, visual, and structural features. Domain-generator owns creative framing, generation, intrinsic filtering, brand-collision checks, initial availability verification, and creative rationale. Resume this workflow only after it returns its curated candidates. Do not create, extend, filter, or iterate creative candidates inside this skill. If the request is only for creative variants and no lifecycle comparison or advisory synthesis is needed, hand off directly to domain-generator and let it deliver the result.

4. **Gather lifecycle channels.** Scope every channel to what the user accepts. In taken-target mode, lifecycle sources are a primary channel; in project mode, use them when the registrable creative shortlist is thin or the user explicitly wants purchase / backorder options. Search the selected keyword or seed at both start and end where the source supports it:
   - `deleted` for dropped names that may be registrable; re-confirm every retained candidate with `bulk_available`.
   - `expired`, only when the user accepts backorder; verify lifecycle stage, backorder / auction status, and provider. Do not use `bulk_available` on a name that remains registered.
   - `aged`, only when the user accepts purchasing a registered domain; verify the current marketplace listing, asking price, and listing type. A public listing is a purchase path, never contact-owner.
   - Contact-owner only as a last-resort path for a registered candidate the user specifically wants, usually the original target, when no public listing exists. Confirm a named broker via the public web. Exclude the candidate if neither a listing nor a confirmable broker exists; never bypass WHOIS privacy or contact a private registrant directly.

5. **Verify, compare, and shortlist across channels.** Preserve domain-generator's naming territory, semantic distance, formation, creative concept, and seed connection. Re-run `bulk_available` only when a creative candidate's availability result is no longer fresh. Run the lightweight public-web collision check on any lifecycle candidate not already checked. Use `tld_check` only as a crowding / distribution signal, never as demand or quality. Compare all candidates on user fit, what they preserve from the brief or taken target, pronunciation, spelling, language risk, current acquisition status, cost basis, and acquisition complexity. Present up to 5 to 10 verified candidates, grouped from easiest to most involved: standard registration, backorderable, purchasable, then contact-owner. Availability alone does not make a candidate worth recommending.

6. **Iterate and follow up.** Route creative feedback back through domain-generator by semantic distance, image, sound, or formation; do not create a second prefix-swapping loop here. Adjust lifecycle channels when the user's acquisition tolerance changes. If the current best is already strong, do not force weaker options. Tailor next steps to the selected candidate: monitoring, deeper analysis (domain-analyze or expired-domain-analysis), or substitution-based current-market positioning (domain-cma-valuation).

## Output format

Present each shortlisted candidate with:

- **Domain**: the candidate.
- **Origin**: domain-generator creative candidate, deleted find, expired opportunity, aged listing, or brokered target; include the project territory or relation to the taken target.
- **Creative concept / formation** (for domain-generator candidates): preserve its naming territory, semantic distance, formation, and intended creative rationale.
- **Why it matches**: in project mode, its relation to the project's positioning and creative brief; in taken-target mode, what it preserves from the original target (literal word, meaning, pattern, length, sound, rhythm, image, or brevity).
- **Pronunciation / spelling notes**: readability and the radio test.
- **Current status, matched to the path**, with observation timestamp: for a registrable candidate, the `bulk_available` result as "available at check time"; for expired, the current lifecycle and backorder / auction status; for aged, the current listing status; for contact-owner, the confirmed public channel. Never label an already-registered candidate "available".
- **Acquisition path**: standard registration, backorder, purchase, or contact-owner.
- **Status / cost basis, matched to the acquisition path** (do not force first-year + renewal onto every path):
  - Standard registration: standard vs premium status, first-year price, and renewal price, with currency.
  - Backorder (expired): backorder fee and current bid where shown, with currency; note that securing the name is not guaranteed and renewal pricing applies only after it is caught.
  - For-sale purchase (aged): the listing / asking price and listing type (BIN, minimum offer, current bid), with currency; an asking price is not a sale price.
  - Contact-owner: "Price unknown / negotiation required" (do not estimate).
- **Historical sale price** (optional, include only when a named verifiable source provides it): a past confirmed sale from a named sales record or marketplace announcement, with that source and the sale date, kept as a separate field and never presented as the current asking or registration price. This skill has no dedicated sales-history data source, so omit this field when no such source is at hand; never estimate or infer a past price. For current market pricing, hand off to domain-cma-valuation, which positions a domain against current for-sale listings (it does not build a range from past sales); for a domain's sale history as part of due diligence, use domain-analyze.
- **Source and observation date**: where the price came from and when checked.
- **Language / brand caveats**: cross-language meaning issues or possible brand collisions.

Contact-owner and purchase are mutually exclusive. A domain with any public marketplace listing is a purchase candidate, so route it there. Reserve contact-owner for a registered candidate with no public listing but a named broker confirmed by a web check. If a candidate has neither a listing nor a confirmable broker, exclude it rather than presenting it, and never attempt to bypass WHOIS privacy or contact a private registrant directly. State the confirmed broker in the candidate's acquisition path.

## Key principles

- Treat domain-name-advisor as the orchestration and recommendation layer: it diagnoses the project or taken target, forms the handoff brief, gathers lifecycle inventory, compares acquisition paths, and delivers the final shortlist.
- Use domain-generator as the sole creative-generation component in both modes. Do not maintain a second naming-generation or creative-filtering system here.
- Determine the mode first: project mode turns open requirements into a generator-ready brief; taken-target mode confirms unavailability and records what the user values in the target. Both then delegate all creative work to domain-generator and use this advisor for lifecycle comparison and delivery.
- Diagnose briefly, then act; ask one question at a time and only when it changes direction. Do not force a full multi-turn diagnosis on a clear brief.
- In project mode, make the handoff brief specific enough for domain-generator to frame the creative work; in taken-target mode, anchor it to what the user valued in the original. Preserve domain-generator's creative rationale in the final delivery.
- Present a small cross-channel verified shortlist (up to 5 to 10), not the full creative or lifecycle inventory.
- Availability is point-in-time and applies only to registrable candidates: label those "available at check time" with a timestamp. An already-registered candidate is verified by its own lifecycle / listing status, never labeled "available", and never checked with `bulk_available`.
- Lifecycle sources are primary in taken-target mode and expansion-only in project mode, always scoped to what the user accepts: `deleted` for dropped names (re-confirm with `bulk_available`); `expired` only when the user accepts backorder (verify lifecycle, backorder / auction status, and provider, not `bulk_available`); `aged` only when they accept buying a registered domain (verify current listing and asking price, not `bulk_available`); `tld_check` is a crowding signal only.
- Match the cost basis to the acquisition path: first-year / renewal for standard registration, backorder fee / current bid for backorder, asking price / listing type for aged; never carry first-year / renewal onto an already-registered candidate; contact-owner prices are unknown, not estimated.
- Keep any historical sale price as a separate field, filled only from a named verifiable source; this skill has no dedicated sales-history source, so omit it when none is found and never estimate. Never present a past sale as a current asking or registration price.
- Group the shortlist by acquisition method, easiest to most involved: standard registration, then backorderable, then purchasable, then contact-owner.
- Contact-owner and purchase are mutually exclusive: any domain with a public marketplace listing is a purchase candidate. Reserve contact-owner for a registered candidate the user specifically wants that has no listing but a named broker confirmed by a web check; if it has neither a listing nor a confirmable broker, exclude it, and never bypass WHOIS privacy.
- Source factual claims; present naming-quality judgments as reasoned creative assessments, not objective facts.
- A public-web check is not trademark clearance; never call a candidate legally safe to use.
