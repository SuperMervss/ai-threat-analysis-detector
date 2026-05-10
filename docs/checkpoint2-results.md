# Checkpoint 2 Results
**Date:** 2026-05-09  
**Team:** Solo Capstone (Mervin Laforteza)

**Test record:** Alert records were sent through Airtable -> n8n -> Flowise -> Airtable writeback using three test cases:
- TEST001: phishing email (high-risk)
- TEST002: routine firewall maintenance (low/informational)
- TEST003: empty/bad input (safe default handling)

## End-to-End Status: PASSED (Component 3 scope)

## Component-by-Component Results

### Ingestion
- **Status:** Partially Working (Simulated for Component 3)
- **What happened:** Ingestion is done through manually entered test samples in Airtable `Alerts` (not a live upstream feed). Records with `status=pending_scoring` were successfully picked up by the scoring workflow.
- **Screenshot:** `checkpoint2-ingestion-alerts-table.png`

### AI Core
- **Status:** Working
- **What happened:** Flowise/Groq scoring chain received the alert text and returned structured JSON outputs (`relevance_score`, `severity_level`, `extracted_entities`, `threat_category`, `confidence`, `recommended_actions`).
- **Screenshot:** `checkpoint2-aicore-flowise-output.png`

### Specialist
- **Status:** Working
- **What happened:** Component 3 (Relevance Scorer) normalized the AI output, preserved Airtable record IDs via Merge, and wrote correctly mapped fields into `Scoring_Results`.
- **Screenshot:** `checkpoint2-specialist-scoring-results.png`

### Integration Dashboard
- **Status:** Partially Working
- **What happened:** Completed records are visible in Airtable table views and linked correctly, but full dashboard/slack downstream integration (Component 4) is not yet implemented.
- **Screenshot:** `checkpoint2-integration-airtable-view.png`

## Gaps Found
- Confidence scale inconsistency (some outputs can be interpreted as 0-1 instead of 0-100)  
  Owner: Component 3 (Mervin)
- Flowise cloud prediction limit can block requests during heavy testing  
  Owner: Component 3 (Mervin)
- Exported n8n workflow JSON not yet committed under `component-3-scoring`  
  Owner: Component 3 (Mervin)
- Components 1, 2, and 4 are still simulated/manual for this checkpoint scope  
  Owner: Project scope (solo)

## Fix Plan
1. Standardize confidence handling to 0-100 before Airtable writeback (Owner: Mervin, Effort: 0.25h)
2. Add retry/backoff or fallback handling for Flowise prediction-limit errors (Owner: Mervin, Effort: 0.5h)
3. Export and commit final n8n workflow JSON and screenshot evidence (Owner: Mervin, Effort: 0.25h)
