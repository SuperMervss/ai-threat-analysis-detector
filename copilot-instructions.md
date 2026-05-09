# Capstone Project Context

## Project
- **Name:** Threat Intelligence Feed Dashboard
- **Team:** Mervin Laforteza (Component 3: Scoring/Relevance Scorer)
- **What it does:** Ingests security threat alerts from multiple sources, processes them through AI models to assess relevance to the organization's tech stack, assigns risk scores and severity levels, and outputs prioritized threat intelligence to security teams. Benefits: Security teams get ranked, actionable threat intelligence; reduces alert fatigue; faster response times.
- **Project type:** Threat Intelligence Hub / Security Alert Prioritization System

## Architecture
- **Ingestion (Component 1):** Threat alerts ingested from SIEM/security tools via API, parsed for threat details, stored in Airtable Alerts table with status "pending_scoring"
- **AI Core (Component 2-3):** Groq Llama 3.1 LLM evaluates threat relevance to org tech stack; HuggingFace NER extracts entities; RAG system retrieves MITRE ATT&CK/NIST context; zero-shot classification categorizes threat types; results written to Airtable with relevance_score, severity_level, recommended_actions
- **Specialist (Component 4):** Integration layer formats scored threats for dashboards, sends high-priority alerts to Slack, creates follow-up tasks for security team

## Tech Stack
- n8n Cloud (workflow automation and component orchestration)
- Flowise Cloud (RAG chatbot and security knowledge assistant)
- Groq API (LLM inference — llama-3.3-70b-versatile)
- Hugging Face Inference API (NER, zero-shot classification, entity extraction)
- Airtable (shared database — 3-4 tables: Alerts, Threats, Scoring_Results, Actions)
- GitHub (repo, documentation, portfolio)
- Teachable Machine (image classification for phishing detection)

## Airtable Schema

### Alerts Table
| Field | Type | Written By | Status Values |
|-------|------|-----------|---------------|
| alert_id | Text | Component 1 | N/A |
| threat_description | Long Text | Component 1 | N/A |
| source | Text | Component 1 | N/A |
| timestamp | Date | Component 1 | N/A |
| status | Single Select | All | pending_scoring, scored, actioned, dismissed |
| created_at | Date | Component 1 | N/A |

### Scoring_Results Table
| Field | Type | Written By | Status Values |
|-------|------|-----------|---------------|
| score_id | Text | Component 3 | N/A |
| alert_id | Link to Alerts | Component 3 | N/A |
| relevance_score | Number | Component 3 | N/A |
| severity_level | Single Select | Component 3 | critical, high, medium, low, informational |
| extracted_entities | Long Text | Component 3 | N/A |
| threat_category | Text | Component 3 | N/A |
| confidence | Percent | Component 3 | N/A |
| recommended_actions | Long Text | Component 3 | N/A |
| scored_at | Date | Component 3 | N/A |

## Conventions
- Field names: snake_case
- Status values: lowercase
- Date fields end in _at
- Boolean fields use is_ prefix
- Severity levels: critical > high > medium > low > informational
- Confidence scores: 0-100 percent

## Current State
- **What's working:** 
  - Component 3 (Scoring) model selection complete and tested (Llama 3.1, NER, RAG system)
  - Week 4 model comparison validated Groq as optimal primary model
  - Week 5 fine-tuned phishing classifier (80% accuracy)
  - Week 7 RAG security knowledge system operational
  - All scoring logic and AI pipelines designed and tested
- **What's in progress:** 
  - Integration of scoring component with n8n workflow
  - Connection to Airtable Alerts/Scoring_Results tables
  - End-to-end workflow testing with sample alerts
- **Known issues:** 
  - NER model accuracy on cybersecurity terminology (e.g., "SSH" extracted as "SS")
  - Legitimate email false positive rate in phishing classifier (~20%)
  - RAG system needs tuning for better chunk size and Top-K retrieval
- **Next milestone:** Checkpoint 2 (Week 9) — one record end-to-end through all 4 components with successful scoring and action

## Repository Structure
```
component-1-data-ingestion/
component-2-ai-processing/
component-3-scoring/
  ├── READ3.md
  ├── week-04-model-comparison/
  │   ├── report.md (Model testing results)
  │   └── results/
  ├── week-05-automl-training/
  │   ├── report.md (Fine-tuning results)
  │   └── teachable-machine/ (Phishing classifier)
  └── week-07/
      ├── week-07-report.md (RAG system evaluation)
      └── screenshots/
component-4-integration-output/
docs/
```