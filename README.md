# domainkits-skills

Agent skills powered by [DomainKits MCP](https://domainkits.com/mcp). Each skill is a standalone SKILL.md that gives your AI agent specialized domain industry knowledge and workflows.

Works with Claude Code, OpenClaw, and any agent that supports the [Agent Skills](https://code.claude.com/docs/en/skills) open standard.

## Why Skills?

DomainKits MCP gives your agent tools. Skills teach it **how to think**.

An agent with DomainKits tools can look up WHOIS or check availability. An agent with DomainKits skills can analyze market trends, profile registration spikes, research catalysts, and find opportunities — the way an experienced domainer would.

## Skills

| Skill | What it does |
|---|---|
| [brand-protection](./skills/brand-protection/) | Scan typosquats and lookalike registrations around a brand, evaluate each domain individually on registration facts, surface defensive gaps, and offer monitoring |
| [domain-analyze](./skills/domain-analyze/) | Build a complete evidence-based picture of one domain: registration, DNS, safety, backlinks, cross-TLD footprint, and market context |
| [domain-cma-valuation](./skills/domain-cma-valuation/) | Comparative Market Analysis using verified for-sale substitutes to position a domain within the current asking-price market |
| [domain-generator](./skills/domain-generator/) | Generate brandable domain variations from a keyword, concept, or taken domain across semantic distances, then verify availability |
| [domain-market-beat](./skills/domain-market-beat/) | Time-bounded domain market briefing covering notable sales, registration trends, and recent domain movements |
| [domain-name-advisor](./skills/domain-name-advisor/) | Naming consultation that turns vague requirements into a verified shortlist across registration, aftermarket, and lifecycle paths |
| [keyword-intel](./skills/keyword-intel/) | Map search, advertiser, registration, supply, and transaction evidence around one known keyword |
| [keyword-trend-hunter](./skills/keyword-trend-hunter/) | Discover hot or emerging registration keywords, validate each spike against new-registration cohort signals |

## Prerequisites

All skills require a **DomainKits MCP** connection. Some skills also use platform built-in tools like `web_search`.

### Claude Code

```bash
claude mcp add domainkits https://api.domainkits.com/v1/mcp
```

With API key (for higher limits):

```bash
claude mcp add domainkits https://api.domainkits.com/v1/mcp --header "X-API-Key: YOUR_KEY"
```

### Claude.ai

Connect DomainKits via **Settings → Connectors**. No manual configuration needed.

### Other Agents (OpenClaw, Cursor, etc.)

Add to your MCP config:

```json
{
  "mcpServers": {
    "domainkits": {
      "baseUrl": "https://api.domainkits.com/v1/mcp"
    }
  }
}
```

Get your API key at [domainkits.com](https://domainkits.com). Guest access (no key) is also available with limited daily usage.

## Install

Pick the skill you want and copy its folder:

```bash
# Clone the repo
git clone https://github.com/ABTdomain/domainkits-skills.git

# Copy a skill to your agent's skills directory
cp -r domainkits-skills/skills/keyword-intel ~/.claude/skills/
```

Or download a single skill folder directly from GitHub.

Claude Code will auto-detect the skill and use it when relevant. No restart needed.

## Skill Structure

Each skill lives under `skills/` as a self-contained folder with one required file:

```
skills/
└── skill-name/
    └── SKILL.md    # Instructions, workflow, and metadata
```

Some skills may include additional supporting files (templates, scripts, examples) in the same folder.

## Contributing

Have a domain industry workflow that could be a skill? Contributions welcome.

1. Create a new folder with a descriptive name
2. Add a `SKILL.md` following the [Agent Skills format](https://code.claude.com/docs/en/skills)
3. Make sure it declares its DomainKits tool dependencies in the Prerequisites section
4. Submit a PR

## Links

- [DomainKits](https://domainkits.com) — Domain intelligence platform
- [DomainKits MCP](https://domainkits.com/mcp) — MCP endpoint documentation
- [GitHub](https://github.com/ABTdomain/domainkits-mcp) — DomainKits MCP source

## License

MIT
