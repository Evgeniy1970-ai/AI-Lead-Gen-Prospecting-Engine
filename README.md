# NeolithAI Sales Intelligence System

**A three-agent B2B outreach automation built on n8n, GPT-4o, and Supabase.**

From a single Telegram command to a ready-to-send Gmail draft — fully automated, human-approved at every critical step. Built for EU & UK markets. Works with any business type in any city.

> ~$0.25 per full lead cycle · Any niche · Any location · Human-in-the-loop by design

---

## What It Does

You type one command in Telegram:

```
restaurants Jordaan Amsterdam Netherlands
```

The system does the rest:

1. **Finds** relevant businesses via Google Maps with precise geo-filtering
2. **Enriches** each lead — website, email, LinkedIn, phone, Google rating
3. **Scores** leads and surfaces only the best ones for your review
4. **Researches** approved leads in depth — site content, customer reviews, social signals
5. **Generates** a personalized outreach email grounded in real business intelligence
6. **Creates** a Gmail draft ready for your final send

You make two decisions. The system handles everything else.

---

## Architecture

![NeolithAI LeadGen Architecture](NeolithAI_LeadGen_Architecture_v3.png)

The system runs as three specialized n8n workflows, each triggered by the previous one. All three share a single Telegram bot and a Supabase database.

---

## The Three Agents

### Agent 1 — Lead Scout
*GPT-4o-mini · discovery → enrichment → scoring*

- Parses your Telegram command, geocodes the location, builds a bounding box for accurate district-level search
- Queries Google Maps Places API and deduplicates results (within-batch and across previous runs)
- Runs a waterfall email enrichment: Serper snippets → Firecrawl `/contact` → Firecrawl `/about`
- Searches LinkedIn via Serper with false-positive filtering
- Scores each lead 0–100 (HOT ≥ 75, WARM ≥ 30, COLD < 30)
- Runs Chain Detection and Contact Domain checks — flags suspicious leads visibly, never auto-blocks
- Sends a HOT lead card to Telegram with **[Research]** / **[Skip]** buttons
- Routes your choice: Research triggers Agent 2, Skip logs and ends

### Agent 2 — Deep Research
*GPT-4o · manual trigger via [Research] · HOT leads only*

Three parallel branches run simultaneously:

- **Branch A — Site Analysis:** Firecrawl sitemap crawl → keyword-filtered URLs → full page scrape → GPT-4o content analysis
- **Branch B — Review Analysis:** Google Places API reviews → GPT-4o pattern recognition (service issues, recurring complaints, operational signals)
- **Branch C — Social Signals:** Serper search for Instagram + Facebook with defensive false-positive filtering

Results merge and feed into GPT-4o for a unified pain points / automation gaps analysis. Output is stored in Supabase with separate confidence scores for site and review data.

Sends a full dossier to Telegram with **[Approve]** / **[Reject]** buttons.

### Agent 3 — Proposal Generator
*GPT-4o · runs after Approve*

- Checks suppression list and existing proposals before generating anything
- Reads the full lead + research record from Supabase
- Generates a 100–150 word personalized outreach email grounded in the research dossier
- No generic language, no banned words, English only, GDPR-compliant footer included
- Sends the proposal to Telegram with **[Send to Gmail]** / **[Discard]** buttons
- Creates a Gmail draft — the Send button stays with you

---

## Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| AI Models | GPT-4o-mini (Agent 1) · GPT-4o (Agents 2 & 3) |
| Database | Supabase (PostgreSQL) |
| Business Search | Google Maps Places API v1 (textQuery + bbox) |
| Geocoding | Google Geocoding API |
| Web Scraping | Firecrawl |
| Contact & Social Search | Serper |
| Operator Interface | Telegram Bot API (inline keyboards + callbacks) |
| Email Output | Gmail API (draft creation only) |

---

## Database Schema

Three linked tables. Each agent writes to its own table and updates lead status on completion.

```sql
-- leads: created by Agent 1
id, company_name, website, email, phone, linkedin, address,
lead_score, lead_tier, status, place_id,
chain_signal, contact_signal, error_message, created_at

-- research: created by Agent 2
id, lead_id, content, pain_points, automation_gaps,
review_patterns, social_signals, google_rating, review_count,
site_confidence, review_confidence, analysis_notes, analyzed_at

-- proposals: created by Agent 3
id, lead_id, research_id, kp_text, gmail_draft_id,
status, sent_at, created_at, approved_at

-- suppression_list: managed manually
id, email, domain, reason, added_at
```

**Lead status machine:** `discovered` → `researched` → `proposal_ready` → `sent` (or `rejected` / `discarded` at any stage)

---

## Economics

| Scope | Cost |
|---|---|
| Agent 1 — 10 leads | ~$0.07 |
| Agent 2 — 2 HOT leads | ~$0.12 |
| Agent 3 — 2 proposals | ~$0.06 |
| **Full cycle, one lead** | **~$0.25** |
| 100 full cycles/month | ~$25 |
| 500 full cycles/month | ~$125 |

---

## Setup

### Prerequisites

- n8n instance (cloud or self-hosted)
- Supabase project with the schema above (see `supabase_schema_neolith_v2.sql`)
- API credentials: Google Maps, Serper, Firecrawl, OpenAI, Telegram Bot, Gmail OAuth2

### Credentials

Each workflow uses named credentials. After importing, reconnect the following in n8n:

| Credential name | Type | Used in |
|---|---|---|
| `Lead Generator Suite` | Supabase | All agents |
| `Google_Maps_Key` | HTTP Query Auth | Agent 1 |
| `Serper_API` | HTTP Header Auth | Agents 1 & 2 |
| `Firecrawl_API` | HTTP Header Auth | Agents 1 & 2 |
| `OpenAi account` | OpenAI API | Agents 1, 2 & 3 |
| `Telegram_Mass_Scout` | Telegram API | All agents |
| `Gmail account` | Gmail OAuth2 | Agent 1 |

### Installation

1. Import all three workflow JSON files into n8n
2. Reconnect credentials (credential IDs are replaced with `YOUR_*_CREDENTIAL_ID` placeholders)
3. In Agent 1, set your Telegram chat ID in the `ALLOWED_CHAT_IDS` constant (`YOUR_TELEGRAM_CHAT_ID`)
4. Create the Supabase tables using the provided SQL schema
5. Publish all three workflows
6. Send your first command to the Telegram bot

### First Command

```
restaurants De Pijp Amsterdam Netherlands
```

For best results, use district-level queries rather than city-wide ones. Google Maps Places API returns ~60 results per query — granular targeting gives better precision.

---

## Design Principles

**Human decides, agent prepares.** Two mandatory approval gates (after Agent 1 and after Agent 2) ensure no email is ever sent without explicit operator sign-off.

**Null is better than wrong data.** Missing information is flagged and stored as null rather than filled with assumptions. Chain Detection and Contact Domain signals are always shown to the operator, never used to auto-block.

**Full workflow execution, not step-by-step.** Always test by running the complete workflow — n8n can return stale cached data when testing individual nodes.

**Cost of error determines depth of fix.** Lightweight heuristics for signals that don't affect output quality; robust validation for anything that touches email generation or sending.

---

## GDPR Compliance

- Suppression list checked before every proposal generation
- GDPR-compliant footer included in every outreach email (identity, location, unsubscribe mechanism)
- No auto-send at any stage — Gmail draft only
- Contact names deliberately excluded from proposals (GDPR Art. 14 / Art. 4(1))

---

## Built by

**NeolithAI Agency** · Kraków, Poland · [neolithai.agency](https://neolithai.agency)

Boutique AI automation studio specialising in EU & UK markets. If you'd like this system deployed and configured for your outreach operation, [get in touch](https://neolithai.agency).
