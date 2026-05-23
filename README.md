# SEO Blog Agent

An end-to-end automated SEO content generation system built with n8n. Takes a keyword and topic as input and outputs a fully researched, strategically structured, and published SEO blog post — with human-in-the-loop validation at every key stage.

---

## Overview

This project automates the entire content production pipeline: from keyword research and competitor analysis, through AI-powered content planning, to final blog generation and publishing. It integrates DataForSEO APIs, multiple LLMs, Google Sheets, GitHub Gists, and Notion into a single coherent workflow.

```
User Input → SERP Analysis → Competitor Keyword Extraction → Keyword Enrichment
     → AI Keyword Selection → Checklist Generation → User Validation
          → Blog Generation → User Approval → GitHub + Notion Publishing
```

---

## Features

- **Conversational input** — chat-based interface for collecting keyword, audience, intent, and location
- **Live SERP data** — fetches real-time Google organic results via DataForSEO
- **Competitor content scraping** — extracts and cleans HTML from top-ranking pages
- **Multi-stage keyword research** — KFK expansion, keyword suggestions, difficulty scoring, intent classification, and trend analysis
- **AI-driven keyword selection** — GPT-4.1 mini selects 10–15 H2-ready keywords with strategic justification
- **SEO checklist generation** — structured content plan with keyword hierarchy, TOC, CTAs, and tone guidelines
- **Human-in-the-loop validation** — review and edit both the checklist and the blog before publishing
- **Full blog generation** — 1500–1700 word, SEO-structured blog written by GPT-4.1
- **Dual publishing** — final content published as a public GitHub Gist and a Notion page

---

## Tech Stack

| Layer | Tools |
|---|---|
| Workflow Engine | n8n |
| LLMs | GPT-4.1, GPT-4.1 mini (OpenAI), LLaMA 3.1 8B (Groq) |
| SEO Data | DataForSEO (SERP, KFK, KS, KD, Intent, Trends APIs) |
| Storage | Google Sheets |
| Publishing | GitHub Gists, Notion |
| Language | JavaScript (Code Nodes) |

---

## Workflow Stages

### 1. Conversational Input (AI Layer 1)
Collects and validates user inputs via chat:
- Primary keyword
- Blog title
- Target audience
- Location (normalized to full country name)
- Search intent distribution (must sum to 100%)

### 2. Google Sheets Initialization
Creates a new spreadsheet with four worksheets:
- `SERP Data` — raw search engine results
- `Competitor Keywords` — keywords extracted from competitor pages
- `All Keywords` — enriched and filtered keyword pool
- `Selected Keywords` — final shortlisted keywords

### 3. SERP Data Retrieval
Calls the DataForSEO SERP API for the top 10 organic results. Filters out PDF links and stores structured results (rank, URL, title, snippet) in the SERP Data sheet.

### 4. Competitor Content Extraction
Fetches HTML from the top 5 competitor URLs. Extracts titles, headings (H1–H3), and paragraphs, then cleans and truncates to ~2000 tokens before passing to the AI.

### 5. Competitor Keyword Extraction
LLaMA 3.1 8B (via Groq) analyzes competitor content and classifies keywords as:
- Primary
- Secondary
- Long-tail

### 6. Keyword Enrichment & Expansion
Expands and scores the keyword pool using multiple DataForSEO endpoints:
- **KFK API** — keyword-for-keyword expansion (up to 100 suggestions)
- **Keyword Suggestions API** — seed-based expansion
- **Bulk KD API** — keyword difficulty scores
- **Search Intent API** — intent classification
- **Google Trends API** — 3-month trend scores

Applies two rounds of AI filtering (GPT-4.1 mini) and rule-based filters (min search volume > 150, CPC ≥ 0.5, word count ≥ 2).

### 7. Final Keyword Selection (AI Layer 2)
GPT-4.1 mini acts as an *Elite SEO Keyword Strategist* and selects 10–15 keywords from the enriched pool. Each keyword includes:
- Relevance (LOW / MEDIUM / HIGH)
- Growth Trend (Growing / Stable / Declining)
- Priority (LOW / MEDIUM / HIGH)
- Strategic justification

### 8. Checklist Generation & User Validation
GPT-4.1 generates a structured SEO blog checklist covering:
- Keyword hierarchy (H1 → H2 → H3)
- Table of contents
- Target audience definition
- Blog objectives, tone, and voice
- Industry context, use cases, and CTAs

The checklist is sent to the user for review. Edits can be requested (routes back to Edit Mode) or it can be approved as-is. Approved checklists are published as a GitHub Gist and saved to Notion.

### 9. Blog Generation & Publishing
GPT-4.1 writes a 1500–1700 word SEO blog based on the approved checklist. The draft can be modified in a loop until satisfied. Final blog is:
- Published as a public GitHub Gist (Markdown)
- Saved to a Notion page with access link

---

## Data Flow

```
Selected Keywords Sheet
        ↓
Checklist Generation (GPT-4.1)
        ↓
User Review → [Edit] → loop back
        ↓ [Approve]
GitHub Gist + Notion (Checklist)
        ↓
Blog Generation (GPT-4.1)
        ↓
User Review → [Modify] → loop back
        ↓ [Accept]
GitHub Gist + Notion (Blog)
        ↓
User Notification
```

---

## Setup

### Prerequisites
- n8n (self-hosted or cloud)
- API keys for: OpenAI, Groq, DataForSEO
- OAuth credentials for: Google Sheets, GitHub, Notion

### Credentials to Configure in n8n
| Service | Credential Type |
|---|---|
| OpenAI | API Key |
| Groq | API Key |
| DataForSEO | HTTP Basic Auth |
| Google Sheets | OAuth2 |
| GitHub | Personal Access Token (Gist scope) |
| Notion | Integration Token |

### Import & Run
1. Import the workflow JSON into your n8n instance.
2. Configure all credentials under **Settings → Credentials**.
3. Activate the workflow.
4. Open the chat trigger URL and start a conversation.

---

## Example Input

```
Primary keyword: AI development services
Blog title: Top AI Development Services for Mid-Market Businesses in 2025
Target audience: CTOs and CFOs at US mid-market companies
Location: United States
Intent: 80% informational, 20% commercial
```

---

## Output

- ✅ Keyword research spreadsheet (Google Sheets)
- ✅ SEO blog checklist (GitHub Gist + Notion)
- ✅ Final SEO blog post (GitHub Gist + Notion)

---

## Project Structure

```
/
├── workflow.json        # n8n workflow export
├── README.md
└── docs/
    └── system-report.pdf   # Full system documentation
```

---

## Key Design Decisions

- **Google Sheets as the data backbone** — acts as a shared, inspectable store across all stages, making the pipeline debuggable without a separate database.
- **Multi-model strategy** — fast/cheap models (LLaMA 3.1 8B, GPT-4.1 mini) handle extraction and filtering; GPT-4.1 handles strategic and generative tasks.
- **Human-in-the-loop at two checkpoints** — checklist approval and blog approval keep the output aligned with intent despite being fully AI-generated.
- **Modular node architecture** — each stage (SERP, competitor, keyword, selection, content) is self-contained and can be extended independently.
