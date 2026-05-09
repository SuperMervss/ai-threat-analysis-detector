# Threat Intelligence Feed Dashboard

## Project Overview
A capstone project that ingests security threat alerts from multiple sources, processes them through AI models to assess relevance to the organization's tech stack, assigns risk scores and severity levels, and outputs prioritized threat intelligence to security teams.

**Owner:** Mervin Laforteza  
**Component:** Component 3 (Relevance Scorer)  
**Status:** Checkpoint 2 complete — end-to-end scoring workflow operational

## What This Project Does
1. **Ingestion (Component 1):** Collects threat alerts from SIEM/security tools via API, parses threat details, stores in Airtable
2. **AI Processing (Component 2):** Summarizes data and extracts key indicators (IPs, domains, file hashes)
3. **Scoring (Component 3):** Evaluates threat relevance using Groq Llama 3.1 + HuggingFace NER + RAG knowledge retrieval
4. **Integration (Component 4):** Formats scored threats for dashboards, sends high-priority alerts to Slack, creates follow-up tasks

## Tech Stack
- **n8n Cloud** — workflow automation and component orchestration
- **Flowise Cloud** — LLM chains and RAG security knowledge assistant
- **Groq API** — llama-3.3-70b-versatile LLM inference
- **HuggingFace Inference API** — NER, zero-shot classification, entity extraction
- **Airtable** — shared database (Alerts, Scoring_Results, Actions tables)
- **GitHub** — repository, documentation, portfolio

## Checkpoint 2 Status: ✅ READY
**Core requirement:** One alert record flows from Airtable through n8n + Flowise scoring and writes back to Airtable automatically.

**Completed:**
- Airtable schema (Alerts & Scoring_Results tables) live and linked
- n8n workflow with automatic trigger, Flowise scoring call, and result writeback
- Status update from `pending_scoring` → `scored`
- End-to-end test (TEST001) successful

**See:** [docs/checkpoint2-audit.md](docs/checkpoint2-audit.md) for full readiness assessment

## Component Details
Each component has its own README:
- [component-1-data-ingestion/](component-1-data-ingestion/)
- [component-2-ai-processing/](component-2-ai-processing/)
- [component-3-scoring/README.md](component-3-scoring/README.md) — Relevance Scorer setup and testing guide
- [component-4-integration-output/](component-4-integration-output/)

## Quick Start (Component 3 - Relevance Scorer)
See [component-3-scoring/README.md](component-3-scoring/README.md) for full setup instructions.

Required:
- Airtable account with Alerts and Scoring_Results tables
- n8n workspace
- Flowise Cloud deployment with threat scoring chatflow
- Groq API key

Test with one alert record:
1. Add to Airtable Alerts table: `alert_id=TEST001`, `threat_description=...`, `status=pending_scoring`
2. Trigger n8n workflow
3. Verify new row appears in Scoring_Results with linked alert_id
4. Verify Alerts status changed to scored

## Project Structure
```
component-1-data-ingestion/
component-2-ai-processing/
component-3-scoring/
  ├── README.md (setup and testing guide)
  ├── READ3.md (component summary)
  ├── week-04-model-comparison/ (model evaluation)
  ├── week-05-automl-training/ (fine-tuned classifier)
  └── week-07/ (RAG knowledge assistant)
component-4-integration-output/
docs/
  ├── checkpoint2-audit.md (readiness assessment)
  └── proposal.md
copilot-instructions.md (full project context)
prompt-log-mervin.md (AI tool interaction log)
```

## Known Limitations
- Scoring quality depends on Flowise prompt tuning and model behavior
- Confidence values may need scaling (0-1 vs 0-100) before Airtable write
- Current milestone simulates upstream components through manual Airtable input
- NER model accuracy is lower on cybersecurity terminology
- Phishing classifier has ~20% false positive rate

## Next Steps (Post-Checkpoint 2)
1. Improve phishing classifier accuracy with more training data
2. Enhance NER for cybersecurity-specific entity extraction
3. Build full Components 1, 2, 4 workflows
4. Implement real Slack alerting for high-priority threats
5. Add persistent Flowise knowledge source management
- Flowise Cloud account and a deployed prediction endpoint
- Groq API key (configured in Flowise model credentials)

## 2) Airtable Configuration
Create/verify these tables and fields:

### Alerts
- alert_id (single line text)
- threat_description (long text)
- source (single line text)
- timestamp (date)
- status (single select: pending_scoring, scored, dismissed)
- created_at (date)

### Scoring_Results
- score_id (single line text)
- alert_id (link to Alerts)
- relevance_score (number)
- severity_level (single select: critical/high/medium/low/informational)
- extracted_entities (long text)
- threat_category (single line text)
- confidence (percent)
- recommended_actions (long text)
- scored_at (date/time)

Important: alert_id in Scoring_Results is a linked-record field, so n8n must send an array with the Airtable record ID.

## 3) Flowise Configuration
Create a chatflow for threat scoring and expose the prediction URL.

Recommended output contract (JSON only):
- relevance_score
- severity_level
- extracted_entities
- threat_category
- confidence
- recommended_actions

Recommended prompt behavior:
- phishing/spoofing/targeted credential attack => high or critical
- routine maintenance => low or informational
- ambiguous indicators => medium
- do not return all zeros unless input is empty/malformed

## 4) n8n Configuration
Build a workflow with this order:
1. Airtable Trigger (Alerts table, trigger field alert_id)
2. Set node (map alert_id, input_text, source, timestamp, record_id)
3. HTTP Request node (POST to Flowise prediction endpoint)
4. Normalizing Code/Set node (ensure fields and data types match Airtable)
5. Airtable Create Record (Scoring_Results)
6. Airtable Update Record (Alerts.status = scored)

Key mapping rules:
- Scoring_Results.alert_id must be [record_id] array, not TEST001 string
- extracted_entities should be converted to a string before write
- confidence percent may need scaling depending on Airtable formatting

## How To Test

## Test 1: High-Risk Phishing
1. Add a new Alerts record:
   - alert_id: TEST001
   - threat_description: Phishing email with spoofed Amazon domain detected targeting finance@acmecorp.com. Sender IP traced to Moscow.
   - source: email_gateway
   - status: pending_scoring
2. Run/trigger workflow.
3. Verify:
   - New Scoring_Results row exists
   - Linked alert_id points to TEST001 alert record
   - Alerts.status changed to scored

## Test 2: Ambiguous Routine Event
1. Add a second alert:
   - alert_id: TEST002
   - threat_description: Routine firewall rule update completed during scheduled maintenance.
   - source: firewall_log
   - status: pending_scoring
2. Trigger workflow.
3. Verify lower severity/relevance and successful writeback.

## Test 3: Bad/Missing Input
1. Add malformed or empty description alert:
   - alert_id: TEST003
   - threat_description: (empty)
   - source: unknown
   - status: pending_scoring
2. Trigger workflow.
3. Verify graceful handling (fallback values and no workflow crash).

## Known Limitations
- Scoring quality depends heavily on prompt tuning and model behavior.
- Flowise may return array/format variants; normalization is required before Airtable writes.
- Confidence formatting can be inconsistent across systems (0-1 vs 0-100).
- Current milestone simulates upstream components through Airtable input instead of full live ingestion.
- RAG retrieval quality in Flowise can vary if chunking/top-k/knowledge sources are not tuned.

## Current Milestone Status
Checkpoint 2 core requirement is satisfied for this component path:
- one record enters Alerts
- n8n triggers automatically
- Flowise scoring executes
- Scoring_Results is written
- Alerts status updates to scored
