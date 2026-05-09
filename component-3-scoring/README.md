# Component 3: Relevance Scorer

## Overview
The Relevance Scorer analyzes incoming threat alerts and assigns a relevance score, severity level, and recommended response actions. It uses Flowise (LLM chain with Groq) to classify threat context, then writes structured scoring output back to Airtable.

For Checkpoint 2, this component is integrated with n8n so one alert record can be processed end-to-end automatically: trigger, scoring, result writeback, and status update.

## What This Component Does
- Reads threat alert text and metadata from Airtable records.
- Produces a structured scoring payload:
  - relevance_score (0-100)
  - severity_level (critical/high/medium/low/informational)
  - extracted_entities
  - threat_category
  - confidence
  - recommended_actions
- Writes output to the Scoring_Results table and marks the source alert as scored.

## How It Connects to Other Components

### Input From Upstream
- Primary source: Airtable Alerts table (simulated ingestion/AI Core handoff for current milestone)
- Required input fields:
  - alert_id (text)
  - threat_description (long text)
  - source (text)
  - timestamp (date/time)
  - status (single select, should be pending_scoring before processing)

### Output To Downstream
- Writes scored result to Airtable Scoring_Results table:
  - score_id
  - alert_id (linked record to Alerts)
  - relevance_score
  - severity_level
  - extracted_entities
  - threat_category
  - confidence
  - recommended_actions
  - scored_at
- Updates Alerts.status from pending_scoring to scored.
- Output can then be consumed by integration/dashboard workflows (Component 4).

## Setup Instructions

### 1) Required Accounts and Keys
You need the following:
- Airtable account with a base containing Alerts and Scoring_Results tables
- Airtable Personal Access Token (PAT)
- n8n account/workspace
- Flowise Cloud deployment with threat scoring chatflow
- Groq API key

### 2) Airtable Configuration
Create two tables with the schema defined in [copilot-instructions.md](../copilot-instructions.md):

**Alerts table:**
- alert_id (text)
- threat_description (long text)
- source (text)
- timestamp (date/time)
- status (single select: pending_scoring / scored / failed)

**Scoring_Results table:**
- score_id (primary key)
- alert_id (linked record to Alerts table)
- relevance_score (number)
- severity_level (single select: critical / high / medium / low / informational)
- extracted_entities (long text)
- threat_category (text)
- confidence (number)
- recommended_actions (long text)
- scored_at (date/time)

### 3) Flowise Configuration
Deploy a threat scoring chatflow with the following contract:
```json
{
  "relevance_score": 0,
  "severity_level": "critical|high|medium|low|informational",
  "extracted_entities": "",
  "threat_category": "",
  "confidence": 0,
  "recommended_actions": ""
}
```

Input the threat description and expect JSON output with all fields populated.

### 4) n8n Workflow Configuration
Build a 6-node workflow:
1. **Airtable Trigger** — Listen for new records with status=pending_scoring
2. **Flowise HTTP Request** — Call scoring chain with threat_description
3. **Code Node** — Normalize output (convert arrays to strings)
4. **Airtable Create** — Write scored result to Scoring_Results
5. **Airtable Update** — Mark source alert as scored
6. **Success Response** — Log completion

Critical expressions:
- Linked record field: `{{ [ $item(0).$node["Airtable Trigger"].json["id"] ] }}`
- Array to string: `{{ Array.isArray($json.extracted_entities) ? $json.extracted_entities.join(", ") : ($json.extracted_entities ?? "") }}`

## How to Test

### TEST001: Phishing Alert (Should Score High)
1. Create Airtable record:
   - alert_id: TEST001
   - threat_description: "Phishing email detected from domain spoofing paypal-secure.tk. Email body contains credential harvesting links disguised as PayPal login. Recipient: finance@company.com"
   - source: email_gateway
   - status: pending_scoring

2. Trigger n8n workflow manually or wait for auto-trigger.

3. Expected outcome:
   - Scoring_Results row created with relevance_score 75-100, severity_level=high/critical
   - Alerts.status changed to scored
   - Extracted entities include domain, email address

### TEST002: Routine Maintenance (Should Score Low)
1. Create Airtable record:
   - alert_id: TEST002
   - threat_description: "Routine firewall rule update completed on fw-01 during scheduled maintenance window. Change ticket: CHG-12345. Approved by network team."
   - source: firewall_log
   - status: pending_scoring

2. Trigger workflow.

3. Expected outcome:
   - relevance_score 0-30, severity_level=informational/low
   - Demonstrates accurate filtering of routine ops

### TEST003: Bad Data (Should Handle Gracefully)
1. Create Airtable record:
   - alert_id: TEST003
   - threat_description: "" (empty)
   - source: unknown
   - status: pending_scoring

2. Trigger workflow.

3. Expected outcome:
   - Workflow completes without crashing
   - Scoring_Results created with safe default values or error logged
   - Demonstrates resilience

## Known Limitations
- Scoring quality depends on Flowise prompt tuning and model behavior
- Confidence values may need scaling (0-1 vs 0-100) before Airtable write
- NER model accuracy is lower on cybersecurity terminology (industry-specific entities underdetected)
- Phishing classifier has ~20% false positive rate
- Current milestone simulates upstream components through manual Airtable input

## Checkpoint 2 Status
✅ **READY** for demonstration
- Airtable schema live with two linked tables
- n8n workflow all nodes turn green (verified successful execution)
- End-to-end TEST001 validated
- Status transitions working (pending_scoring → scored)
- Linked record relationship confirmed

**Remaining for production:**
- TEST002 and TEST003 repeatability runs
- Flowise prompt tuning to prevent all-zero defaults
- n8n workflow JSON exported to repo
- Integration with Components 1, 2, 4

## Next Steps
1. Run TEST002 and TEST003 for edge case validation
2. Improve Flowise prompt with explicit threat category rules
3. Export n8n workflow to component-3-scoring/n8n-workflow.json
4. Build Components 1 (ingestion) and 2 (AI processing) data sources
5. Implement Component 4 (Slack notifications, dashboard output)
