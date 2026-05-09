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
