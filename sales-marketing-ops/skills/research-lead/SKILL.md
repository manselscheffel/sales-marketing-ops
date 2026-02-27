---
name: research-lead
description: Transform a LinkedIn URL into a complete research package with personalized outreach. Scrapes profile, researches company via Perplexity, runs AI analysis for pain points and DM sequences, stores results in Google Sheets. Run with /research-lead or ask to research a lead.
---

# Lead Research & Personalization

## Objective

Transform a LinkedIn URL into a complete research package with personalized outreach content — constrained to **relevant personalization only**.

Relevant = relates to a problem they're likely facing that we can solve.
Theater = personal but irrelevant (marathons, shared schools, hobbies).

Before including any fact, apply this test: "Does this relate to a problem they're facing that we can solve?" If no, discard it. See `context/outreach/relevance.md` for the full philosophy.

## Inputs Required

- LinkedIn URL (e.g., `https://www.linkedin.com/in/username/`)
- Google Sheets document ID (where to store results)

## Quick Start — Single Lead

Run the full pipeline with one command:

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/research_lead.py "https://www.linkedin.com/in/username/"
```

Add `--post-to-slack` to post the review card to Slack:
```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/research_lead.py "https://www.linkedin.com/in/username/" --post-to-slack
```

The orchestrator handles all steps below automatically, including parallel execution and error handling.

## Execution Steps (Pipeline Detail)

### Step 1: Scrape LinkedIn Profile

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/scrape_linkedin.py "LINKEDIN_URL"
```

**Output**: JSON with profile details, company info, recent posts (last 30 days)
**Dependencies**: Relevance AI API (`RELEVANCE_AI_PROJECT_ID`, `RELEVANCE_AI_API_KEY`)

### Step 2: Research Company & Person via Perplexity

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/research_with_perplexity.py \
  --company "Company Name" --domain "company.com" \
  --person "Full Name" --role "Job Title"
```

Extract company name, domain, person name, and role from Step 1 output.

**Output**: JSON with scale signals, growth signals, org structure, hiring patterns, tech stack, employee reviews, leader facts
**Dependencies**: Perplexity API (`PERPLEXITY_API_KEY`)

### Step 3: Run AI Analyses (Phase 1 — Parallel)

Run these two analyses simultaneously. Both use the combined output from Steps 1-2 as input.

**Analysis A — Lead Profile:**
```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/analyze_with_openai.py \
  --type lead_profile --input .tmp/combined_data.json
```

**Analysis B — Pain & Gain Operational:**
```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/analyze_with_openai.py \
  --type pain_gain_operational --input .tmp/combined_data.json
```

**Dependencies**: OpenAI API (`OPENAI_API_KEY`). Uses prompt templates from `${CLAUDE_PLUGIN_ROOT}/skills/research-lead/assets/prompts/`.

### Step 4: Run DM Sequence (Phase 2 — Sequential)

Requires Phase 1 results. The DM sequence uses only hooks where `allowed: true` from the pain & gain analysis — this filters out theater automatically.

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/analyze_with_openai.py \
  --type dm_sequence --input .tmp/combined_with_analysis.json
```

**Output**: 3-message DM sequence (each max 300 characters)

### Step 5: Generate Human Review Report

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/generate_review_report.py \
  --data .tmp/final_results.json
```

**Output**: HTML review report saved to `deliverables/review/` with quality assessment, DM preview, and source citations.

### Step 6: Store Results in Google Sheets

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/update_google_sheet.py \
  --linkedin-url "LINKEDIN_URL" --data .tmp/final_results.json
```

**Dependencies**: Google Sheets API (`GOOGLE_SHEETS_CREDENTIALS_FILE`, `GOOGLE_SHEETS_TOKEN_FILE`, `GOOGLE_SHEETS_DOCUMENT_ID`)

### Step 7: Post to Slack (Optional)

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/post_lead_review_to_slack.py \
  --data .tmp/final_results.json
```

Posts a review card with Approve/Reject buttons to Slack. Only runs when `--post-to-slack` is passed to the orchestrator.

**Dependencies**: Slack API (`SLACK_BOT_TOKEN` or `SLACK_LEAD_REVIEW_BOT_TOKEN`)

## Process Flow

```
1. Input LinkedIn URL
   ↓
2. Scrape LinkedIn Profile → Profile data + Posts
   ↓
3. Research Company & Person (Perplexity) → Scale/Growth/Org signals
   ↓
4. Phase 1: Run 2 AI Analyses in Parallel
   → Lead Profile (operational focus + achievements)
   → Pain & Gain Operational (hooks with allowed flag)
   ↓
5. Phase 2: Run DM Sequence (uses Phase 1 results)
   → 3-message sequence using allowed hooks only
   ↓
6. Generate HTML Review Report
   ↓
7. Upload to Google Sheets → Mark as "Analysed: Yes"
   ↓
8. Post to Slack (optional) → Approve/Reject buttons
```

## Batch Processing

For automated daily processing of multiple leads from Airtable:

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/batch_research_leads.py --limit 25
```

Dry run (show what would be processed):
```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/batch_research_leads.py --limit 25 --dry-run
```

The batch processor:
1. Reads Airtable for leads where Status = "Not Analyzed"
2. Processes up to 25 leads per run (30-second delay between leads)
3. Runs each lead through the full research pipeline
4. Posts each lead to Slack for review/approval
5. Updates Airtable status to "Awaiting Review"
6. Stops on first failure (fix and re-run to continue)

**Additional dependencies**: Airtable API (`AIRTABLE_PERSONAL_ACCESS_TOKEN`, `AIRTABLE_BASE_ID`), Slack API (`SLACK_BOT_TOKEN`)

**Cron scheduling** (8am EST, weekdays):
```bash
0 8 * * 1-5 cd /path/to/project && python3 ${CLAUDE_PLUGIN_ROOT}/skills/research-lead/scripts/batch_research_leads.py >> logs/cron.log 2>&1
```

## Deliverables

Each lead produces:

- **Person Profile**: Single paragraph professional summary
- **Company Profile**: Single paragraph company overview
- **Operational Focus**: What operational challenges they discuss/post about
- **Operational Achievements**: Verifiable career accomplishments (scaling, building, transforming)
- **Pain Points**: Operational gaps using 4 Engines framework (Acquisition, Delivery, Support, Cross-Functional)
- **Pattern Interrupt Hooks**: Relevance-filtered hooks with `allowed` flag
- **LinkedIn DM Sequence**: 3 messages (DM1: 300 chars, DM2: 300 chars, DM3: 300 chars)
- **Google Sheet Update**: All data stored for sales team

**NOT delivered** (theater — removed from workflow):
- ~~Similarities~~ (shared schools, hobbies = irrelevant)
- ~~Email personalization~~ (podcasts, marathons = theater)
- ~~Connection request~~ (use HeyReach templates instead)

## Expected Output Structures

Detailed JSON output structures for each analysis type are documented in `${CLAUDE_PLUGIN_ROOT}/skills/research-lead/references/output-structures.md`.

## Edge Cases & Error Handling

### LinkedIn Scraping
- **Rate limits**: Relevance AI has limits. Wait 60 seconds and retry once.
- **Profile not found**: Return error immediately if profile doesn't exist or is private.
- **No recent posts**: Continue — analysis works with profile data alone.

### Perplexity Research
- **Company not found**: Proceed with LinkedIn data. Mark research sections as "Limited data available."
- **Domain verification**: Only uses sources matching company domain. Empty arrays are valid.

### OpenAI Analysis
- **Timeout**: Retry once with 5-second backoff.
- **Invalid JSON**: Log raw response and retry with explicit JSON formatting instruction.
- **Character limit violations**: All DMs must be ≤300 chars. Auto-truncate with "..." if violated.

### Google Sheets
- **Sheet not found**: Create it if it doesn't exist.
- **Permission denied**: Ensure service account has edit access.
- **Duplicate entries**: Use LinkedIn URL as matching column. Update existing row.

## Environment Variables Required

```bash
# Relevance AI (LinkedIn scraping)
RELEVANCE_AI_PROJECT_ID=your_project_id
RELEVANCE_AI_API_KEY=your_key_here

# Perplexity AI (web research)
PERPLEXITY_API_KEY=your_key_here

# OpenAI (all analyses)
OPENAI_API_KEY=your_key_here

# Google Sheets
GOOGLE_SHEETS_CREDENTIALS_FILE=credentials.json
GOOGLE_SHEETS_TOKEN_FILE=token.json
GOOGLE_SHEETS_DOCUMENT_ID=your_sheet_id

# Slack (review posting)
SLACK_BOT_TOKEN=xoxb-...
SLACK_LEAD_REVIEW_CHANNEL=lead-review

# Batch processing (optional)
AIRTABLE_PERSONAL_ACCESS_TOKEN=your_token
AIRTABLE_BASE_ID=your_base_id
```

## Performance

- **Total time**: ~45-60 seconds per lead
- **Cost per lead**: ~$0.30-$0.50
- **Phase 1 analyses run in parallel** (lead_profile + pain_gain_operational)
- **Rate limits**: Relevance AI 100/min, Perplexity 50/min, OpenAI tier-dependent, Google Sheets 100/100s
