# Sales & Marketing Ops Plugin

Complete sales and marketing operations system — lead research, content creation, email triage, meeting prep, and content writing in one installable package.

## Install

**Claude Code:**
```
/plugin marketplace add manselscheffel/sales-marketing-ops
/plugin install sales-marketing-ops@sales-marketing-ops
```

**Claude Cowork:**
Upload the .zip file via Organization Settings > Plugins, or drag into the Cowork plugin upload UI.

## Skills

| Skill | What It Does | Required API Keys |
|-------|-------------|-------------------|
| **research-lead** | LinkedIn URL → full research package + DM sequence | Relevance AI, Perplexity, OpenAI, Google Sheets |
| **content-pipeline** | YouTube transcript → LinkedIn posts + carousel PDFs | None (uses MCP connectors) |
| **email-digest** | Gmail triage → prioritized briefing + draft replies | OpenAI |
| **meeting-prep** | Name/company → research brief + talking points | None (uses web search) |
| **content-writer** | Topic → content in your voice (posts, emails, proposals) | None |

## Commands

| Command | Description |
|---------|-------------|
| `/research-lead` | Research a lead from their LinkedIn URL |
| `/content-pipeline` | YouTube transcript to LinkedIn posts + carousels |
| `/email-digest` | Process inbox and deliver executive briefing |
| `/meeting-prep` | Research brief and talking points for a meeting |
| `/content-writer` | Write content in your voice |

## Setup

1. **Install the plugin** (see above)
2. **Connect MCP services** in the Cowork UI: Gmail, Slack, Notion (or your equivalents)
3. **Add API keys** to your `.env` file (see CONNECTORS.md for full list)
4. **Add context files** — create `context/my-voice.md` and `context/my-business.md` in your workspace

## Requirements

- Claude Code or Claude Cowork
- Python 3.10+ (for research-lead and email-digest scripts)
- API keys listed in CONNECTORS.md

## License

MIT
