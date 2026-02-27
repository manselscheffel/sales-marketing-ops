# Connectors

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~CRM` might mean Salesforce, HubSpot, or any other CRM with an MCP server.

Plugins are **tool-agnostic** — they describe workflows in terms of categories (CRM, chat, email, etc.) rather than specific products. The `.mcp.json` pre-configures specific MCP servers, but any MCP server in that category works.

## Connectors for this plugin

| Category | Placeholder | Included servers | Other options |
|----------|-------------|-----------------|---------------|
| Email | `~~email` | Gmail | Microsoft 365 |
| Chat | `~~chat` | Slack | Microsoft Teams |
| Docs | `~~docs` | Notion | Google Docs |

## API key-based services

| Service | Env var | Used by |
|---------|---------|---------|
| Relevance AI | `RELEVANCE_AI_PROJECT_ID`, `RELEVANCE_AI_API_KEY` | research-lead |
| Perplexity | `PERPLEXITY_API_KEY` | research-lead |
| OpenAI | `OPENAI_API_KEY` | research-lead, email-digest |
| Google Sheets | `GOOGLE_SHEETS_CREDENTIALS_FILE`, `GOOGLE_SHEETS_TOKEN_FILE`, `GOOGLE_SHEETS_DOCUMENT_ID` | research-lead |
| Airtable (optional) | `AIRTABLE_PERSONAL_ACCESS_TOKEN`, `AIRTABLE_BASE_ID` | research-lead (batch mode) |
