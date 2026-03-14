---
name: domain-keyword-intelligence
description: "Discover potential domain registration and trading opportunities by analyzing emerging keyword spikes. Uses DomainKits MCP to assess market liquidity and separate real multi-party signals from single-operator noise. Use when the user asks about trending keywords, domain market trends, emerging keywords, registration hotspots, or domain investment opportunities. Different from trend_hunter (interactive multi-turn)."
---
 
# Domain Keyword Intelligence
 
## Overview
 
The domain market is a multi-party ecosystem — investors, end-users, and speculators all participate independently. A real trend shows a natural mix of these participants. Registration spikes are predominantly driven by technology news, major domain sales, and internet industry events.
 
This skill transforms raw `keywords_trends(emerging)` data into actionable market intelligence. Core value chain:
 
```
Discover trends → Multi-dimensional profiling (market vs junk) → NRDS position analysis → Find domain opportunities
```
 
## Prerequisites
 
- Requires DomainKits MCP connection (endpoint: `https://api.domainkits.com/v1/mcp`)
- If tools are unavailable, guide the user to connect via Settings → Connectors (Claude.ai) or MCP config (OpenClaw/Claude Code)
 
## Workflow
 
### Step 1: Fetch Emerging Keyword Data
 
Call `keywords_trends(type="emerging")` to get keywords with registration volume spikes in the last 7-14 days. The tool returns per-keyword data including registration volume (w3/w4), com_ratio, forsale_pct, might_use_count, top_registrar, and other dimensions needed for Step 2 analysis.
 
### Step 2: Multi-Dimensional Profile Analysis (Core Logic)
 
#### How the domain market works
 
The domain market is a multi-party ecosystem. After registering a domain, a person can only do one of three things with it:
 
1. **Sell it** — list on aftermarket platforms and wait for buyers. This is investor behavior. Their presence shows up in `forsale_pct`.
 
2. **Use it (maybe)** — point it to infrastructure (Cloudflare, AWS, Vercel). Their presence shows up in `might_use_count` — but this only indicates the domain was configured beyond default parking, not that it is genuinely in use. It could be a real project or a site farm. Only meaningful when registrar distribution is diverse.
 
3. **Unknown** — the domain sits on default NS, neither listed for sale nor pointed to any infrastructure. The registrant's intent is unclear. This is the remainder after subtracting forsale and might_use from total registrations.
 
These three states account for every registered domain. Like any financial market, a healthy keyword market requires **liquidity** — active trading, not just ownership.
 
`forsale_pct` is the market's trading volume. If forsale is very low, market participation is low — this is not a healthy market signal. A keyword with high com_ratio, dispersed registrars, and high might_use_count but near-zero forsale has registrations but no market.
 
Two hard rules:
- **If forsale is around 1%, this is very likely not a healthy market.** Do not explain it away with "end-user driven" or "terminal demand" — low forsale means low market participation, period.
- **Never make recommendations or judgments without data.** Labels like "NFL-related", "Chinese pinyin demand", "gaming keyword" are speculation unless confirmed by web_search. If you have not searched, do not guess. Output data only, not narratives. Potential healthy keywords must be verified through NRDS registration analysis (Step 3) before being presented as opportunities.
 
Two cross-cutting dimensions apply to all three participant types:
 
- **`com_ratio`** — the "blue chip ratio." High com_ratio means participants are investing in .com — the most expensive TLD. Low com_ratio means activity is concentrated on cheap TLDs.
 
- **`top_registrar.pct`** — the "exchange concentration." Registrars are channels, not identity labels. High concentration (e.g., above 80%) on a single registrar reduces confidence that many independent parties are involved. A real multi-party market almost always shows distributed registrar usage.
 
**The core question for every keyword is: Is this data from "one person" or "a market"?**
 
When analyzing a keyword, check whether each participant type is present. When a type is absent, ask why — the answer tells you what's really happening. When a type overwhelmingly dominates, ask whether that makes sense for a real market or whether it points to a single operator.
 
#### Filtering Process
 
1. Calculate w4/w3 growth rate for each keyword
2. Profile each keyword across all dimensions — check whether all three participant types are present and whether the two cross-cutting dimensions are reasonable
3. Classify as `junk` (single operator or missing participant roles) or `healthy` (genuine multi-party market)
4. Sort healthy keywords by w4/w3 growth rate
5. Take Top 5-8 for deep analysis
 
#### Output Format
 
Summarize filtering results concisely:
- N total emerging keywords
- Excluded X junk signals — each in under 10 words (e.g., "ethereum: single registrar, all forsale, no .com")
- Identified Z healthy market signals
 
List healthy keywords with key profile data:
 
```
llm — W3: 794 → W4: 979 (↑23%)
  com_ratio: 82.6% | forsale: 36.8% | might_use: 63 | top_registrar: Unstoppable Domains 29.7%
  Profile: Multi-party participation, .com dominant, mix of investment and usage. Healthy signal.
```
 
### Step 2.5: Catalyst Research (web_search MANDATORY)
 
For each healthy keyword identified in Step 2, use `web_search` to investigate what is driving the registration spike. Domain registration spikes are predominantly driven by technology and internet industry events — product launches, AI model releases, platform announcements, viral open-source projects, regulatory changes, major acquisitions, or high-profile domain sales.
 
Profile analysis tells you WHETHER a signal is healthy. Catalyst research tells you WHY — and "why" determines whether the opportunity is worth pursuing.
 
For each healthy keyword, search with a technology lens. Prioritize the last 3 days — emerging spikes are almost always driven by very recent events. If nothing is found within 3 days, extend to 10 days maximum:
1. `{keyword} news` — look for product launches, funding rounds, open-source projects, platform announcements, regulatory changes
2. `{keyword} domain sold price` — a high-value domain sale is the single strongest catalyst for registration spikes
 
#### Classify each keyword's catalyst:
 
- **Confirmed catalyst**: News event, product launch, domain sale, etc. found via search → high confidence signal
- **Probable catalyst**: Indirect evidence (e.g., related industry growth) → moderate confidence
- **No catalyst found**: Web search returned nothing relevant → say "cause unidentified" — never fabricate. A healthy profile with no identifiable catalyst is still actionable, but flag the uncertainty. Common English words may have natural baseline demand that doesn't need a specific trigger.
 
#### Output Format
 
```
llm — Catalyst: CONFIRMED
  AI industry continues rapid expansion. Multiple LLM product launches in March 2026.
  Domain sale: llm.ai sold for $XX,XXX on [date] (source)
 
nemo — Catalyst: UNIDENTIFIED
  No relevant news found. Possibly crypto/memecoin related but unconfirmed.
  ⚠️ High growth (↑1730%) with no identifiable cause — treat with caution.
```
 
### Step 3: NRDS Registration Position Analysis
 
**This step bridges "macro trend" to "micro execution."**
 
For each healthy keyword, call `nrds` to examine actual registration patterns:
 
```
nrds(keyword="<keyword>", position="start", tld="com", no_hyphen="true", sort="reg_date_desc", days_range="0-10")
nrds(keyword="<keyword>", position="end", tld="com", no_hyphen="true", sort="reg_date_desc", days_range="0-10")
```
 
#### Analysis Points
 
1. **Position distribution**:
   - `position=start` (e.g., llmtools.com, llmagent.ai) → keyword as category anchor
   - `position=end` (e.g., myllm.com, bestllm.io) → keyword as modifier
   - Comparing volumes reveals how the market positions this keyword
 
2. **Popular combinations**: Extract high-frequency combination words from registrations. E.g., llm + agent, llm + chat, llm + tools. These represent the market's view on the keyword's most valuable applications
 
3. **Registration quality**:
   - Length distribution (short names taken = fierce competition; short names available = window open)
   - `period` (registration term): 6+ years = serious project, 1 year = speculative trial
   - `prefix_tld_count`: high = prefix registered across many TLDs = strong recognition
 
4. **Investor vs end-user behavior**:
   - NS: afternic.com / sedo.com / thisdomain.forsale = investor listing
   - NS: cloudflare / aws / vercel = domain configured for use (could be real project or site farm — check if naming patterns are diverse or template-like)
   - Ratio reveals speculation phase vs adoption phase
 
#### Output Format
 
```
llm — NRDS Registration Analysis
  position=start: 287 domains (llmtools, llmagent, llmchat, llmcode...)
  position=end: 143 domains (myllm, bestllm, openllm, smartllm...)
  Popular combos: agent (42), tools (28), chat (23), code (19), hub (15)
  Market direction: Heavy llm+agent combinations suggest bullish sentiment on LLM Agent space
  Quality: Short domains (<10 chars) largely taken, 10-15 char range still has room
```
 
### Step 4: Next Steps
 
After presenting the trend analysis and NRDS findings, let the user know they can use DomainKits' other tools to explore domain opportunities for any keyword that interests them — such as `deleted` for recently dropped domains, `expired` for backorderable domains, `aged` for domains listed for sale, `keyword_intel` for deep keyword analysis, or `domain_generator` for creative name ideas. Let the user choose which keywords and directions to pursue.
 
## Output Style
 
- **Language**: Follow user's language
- **Concise**: Profile judgments in one sentence. Skip junk quickly, expand on healthy signals
- **Data-driven**: Every judgment cites specific numbers
- **Honest**: No catalyst found → say "cause unidentified" — never fabricate
 
## Notes
 
- Emerging data typically returns 10-100 keywords, most are junk or noise. This is normal — the skill's value is extracting signal from noise
- Step 3 consumes many tool calls. If user has quota limits, show Step 2 results first and let them pick 2-3 keywords before continuing