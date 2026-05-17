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
- status (single select: pending_scoring / scored / failed / error)
- error_reason (long text)

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
Deploy a threat scoring chatflow with this prompt template:

```text
You are a cybersecurity threat scoring assistant.

Given a threat description and a source, return ONLY valid JSON. Do not include markdown, explanations, or extra text.

Use the threat description as the primary evidence. Use the source as supporting context. Different inputs must produce different scores and labels when the evidence differs.

Respond with ONLY a JSON object in this exact shape:
{
  "relevance_score": 50,
  "severity_level": "medium",
  "extracted_entities": [],
  "threat_category": "suspicious_activity",
  "confidence": 50,
  "recommended_actions": ["investigate further"]
}

Rules:
- Return JSON only.
- Do not include markdown.
- Do not include explanations.
- Use the threat description and source to choose the score and labels.
- The source can provide useful context, but do not invent evidence that is not in the description.
- If the alert is clearly malicious, set relevance_score high and severity_level to critical or high.
- If it is routine maintenance or a benign operational note, set relevance_score low and severity_level to low or informational.
- If the alert is ambiguous, choose the safest moderate label and a mid-range score.

Output rules:
- relevance_score must be an integer from 0 to 100
- confidence must be an integer from 0 to 100
- extracted_entities must be an array of short strings; use [] if none. If the input contains names, IPs, domains, file names, systems, or tools, extract them.
- recommended_actions must be an array of short action strings; use [] if none. The action should match the severity level.
- threat_category must be a short lowercase label like phishing, malware, unauthorized_access, suspicious_activity, routine_maintenance, benign
- If the input is empty or blank, return:
   {
      "relevance_score": 0,
      "severity_level": "informational",
      "extracted_entities": [],
      "threat_category": "benign",
      "confidence": 0,
      "recommended_actions": ["no action required"]
   }

Decision logic:
- phishing, spoofed login, fake MFA, or credential harvesting -> high or critical
- unauthorized login from unusual location, IP, or device -> high unless the text clearly says it is approved
- scheduled maintenance, patching, firewall changes, or routine admin work -> low
- suspicious but unconfirmed behavior -> medium
- only use informational when the text is clearly harmless or unrelated to security

Examples:
- Input: "Unauthorized login from IP 198.51.100.4 traced to Moscow targeting user John Miller"
   Output severity should be high or critical, with entities like the IP and user name.
- Input: "Routine firewall rule update completed on fw-01 during scheduled maintenance window"
   Output severity should be low, with entities like fw-01 and firewall rule update.
- Input: "User reported a suspicious email with a link asking for password verification"
   Output severity should be medium or high, depending on how strong the phishing indicators are.

Return JSON only.
```

Use a low temperature setting such as 0.1 or 0.2 for more consistent results. Keep the output parser connected to the LLM Chain.

### 4) n8n Workflow Configuration
Build the workflow with record ID preservation:
1. **Airtable Search/Trigger** — Read pending alerts from Alerts table
2. **Set Node** — Keep `record_id`, `alert_id`, `input_text`, `source`
3. **Code Node (Validate Required Fields)** — Build a missing-field list before calling Flowise
   - Required fields: `alert_id`, `threat_description`, `source`, `timestamp`
   - Output fields: `has_missing_fields`, `missing_fields`, `error_reason`, `status`
   - If any are missing, set `status = "error"`
4. **IF Node** — Route by `has_missing_fields`
   - True path: send to Airtable update for the error branch
   - False path: continue to Flowise
5. **Airtable Update (Error Branch)** — Write the invalid record back to Alerts
   - Set `status = "error"`
   - Set `error_reason = {{$json.error_reason}}`
   - Do not send this item to Flowise or Scoring_Results
6. **Flowise HTTP Request** — Send `input_text` to chatflow
7. **Merge Node (Combine by Position)** — Merge Set output + HTTP output so Airtable record IDs are not lost
8. **Code Node (Normalizing Output)** — Parse model JSON and normalize arrays to text
9. **Airtable Create** — Write scored output to Scoring_Results
10. **Airtable Update** — Mark source alert as `scored`

Critical expressions:
- Linked record field (`Scoring_Results.alert_id`): `{{ [$json.record_id] }}`
- score_id: `{{ $json.record_id + "-score" }}`
- extracted_entities (long text): `{{ $json.extracted_entities_text }}`
- recommended_actions (long text): `{{ $json.recommended_actions_text }}`

Missing-field error branch behavior:
- Build `error_reason` from the first missing required field or a joined list of missing fields
- Update the original Alerts row with `status = "error"`
- Write the reason into `Alerts.error_reason`
- Do not call Flowise or create a scoring result when the input is incomplete

Code node example for the validation step:

```javascript
const requiredFields = ['alert_id', 'threat_description', 'source', 'timestamp'];

for (const item of items) {
   const json = item.json || {};
   const missingFields = requiredFields.filter((field) => {
      const value = json[field];
      return value === undefined || value === null || String(value).trim() === '';
   });

   item.json = {
      ...json,
      missing_fields: missingFields,
      has_missing_fields: missingFields.length > 0,
      status: missingFields.length > 0 ? 'error' : (json.status || 'pending_scoring'),
      error_reason: missingFields.length > 0
         ? `Missing required field(s): ${missingFields.join(', ')}`
         : ''
   };
}

return items;
```

IF node setup:
- Condition: `{{$json.has_missing_fields}}` is true
- True output: connect to the Airtable Update error branch
- False output: connect to the Flowise HTTP Request

Normalizing Output (Code node):
```javascript
const results = [];

for (const it of items) {
   const raw = it.json || {};
   let parsed = {};

   try {
      if (typeof raw.text === 'string') parsed = JSON.parse(raw.text);
   } catch {}

   try {
      if (!Object.keys(parsed).length && typeof raw.answer === 'string') parsed = JSON.parse(raw.answer);
   } catch {}

   try {
      if (!Object.keys(parsed).length && typeof raw.response === 'string') parsed = JSON.parse(raw.response);
   } catch {}

   const extractedEntities = Array.isArray(parsed.extracted_entities)
      ? parsed.extracted_entities
      : Array.isArray(parsed.entities)
         ? parsed.entities
         : [];

   const recommendedActions = Array.isArray(parsed.recommended_actions)
      ? parsed.recommended_actions
      : Array.isArray(parsed.recommendations)
         ? parsed.recommendations
         : [];

   const normalized = {
      record_id: raw.record_id || raw.id || raw.alert_record_id,
      relevance_score: Number(parsed.relevance_score ?? 0),
      severity_level: String(parsed.severity_level ?? 'informational').toLowerCase(),
      extracted_entities: extractedEntities,
      extracted_entities_text: extractedEntities.join(', '),
      threat_category: String(parsed.threat_category ?? 'unknown').toLowerCase(),
      confidence: Number(parsed.confidence ?? 0),
      recommended_actions: recommendedActions,
      recommended_actions_text: recommendedActions.join(', ')
   };

   results.push({ json: { ...it.json, ...normalized } });
}

return results;
```

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
   - Verified sample output: relevance_score=80, severity=high, threat_category=phishing

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
   - Verified sample output: relevance_score=5, severity=informational, threat_category=routine_activity

### TEST003: Bad Data (Should Handle Gracefully)
1. Create Airtable record:
   - alert_id: TEST003
   - threat_description: "" (empty)
   - source: unknown
   - status: pending_scoring

2. Trigger workflow.

3. Expected outcome:
   - Workflow completes without crashing
   - Scoring_Results created with safe default values
   - Verified sample output: relevance_score=0, severity=informational, threat_category=routine_activity
   - Demonstrates resilience

## Known Limitations
- Scoring quality depends on Flowise prompt tuning and model behavior
- Confidence values may need scaling (0-1 vs 0-100) before Airtable write
- NER model accuracy is lower on cybersecurity terminology (industry-specific entities underdetected)
- Phishing classifier has ~20% false positive rate
- Current milestone simulates upstream components through manual Airtable input
- Flowise Cloud may return prediction quota/rate-limit errors during heavy testing

## Checkpoint 2 Status
✅ **READY** for demonstration
- Airtable schema live with two linked tables
- n8n workflow all nodes turn green (verified successful execution)
- End-to-end TEST001, TEST002, and TEST003 validated
- Status transitions working (pending_scoring → scored)
- Linked record relationship confirmed
- Merge + Normalize pipeline confirmed (record IDs preserved through HTTP)

**Remaining for production:**
- Export final n8n workflow JSON to repo
- Improve confidence standardization (0-1 vs 0-100)
- Integration with Components 1, 2, 4

## Next Steps
1. Export n8n workflow to `component-3-scoring/n8n-workflow.json`
2. Standardize confidence to a single scale (0-100) before Airtable write
3. Build Components 1 (ingestion) and 2 (AI processing) data sources
4. Implement Component 4 (Slack notifications, dashboard output)
5. Add retry/error handling for Flowise prediction-limit responses

## Week 10 Error Handling Implementation

Use the scoring workflow to fail softly when Flowise is rate-limited or returns an invalid response.

1. Retry the Flowise HTTP request before giving up.
2. If the request still fails, write a fallback scoring record instead of stopping the workflow.
3. Set fallback values such as `status = error`, `severity_level = informational`, `relevance_score = 0`, and `threat_category = unknown`.
4. Log the error message to n8n console or Airtable so the record can be reviewed later.
5. Keep the `record_id` and linked Airtable fields intact so the downstream merge and writeback still succeed.

Example fallback behavior for the normalization step:

```javascript
let parsed = {};
let errorMessage = '';

try {
   if (typeof raw.text === 'string') parsed = JSON.parse(raw.text);
} catch (error) {
   errorMessage = error.message;
}

if (!Object.keys(parsed).length) {
   parsed = {
      severity_level: 'informational',
      relevance_score: 0,
      threat_category: 'unknown',
      status: 'error',
      error_message: errorMessage || 'Flowise response was empty or invalid',
   };
}
```

## Week 10 Status: Error Handling, Confidence Routing & Dashboard Views

Summary:
- Validation & Error Handling: Added a `Code` validation node to detect missing fields and produce `error_reason`; invalid items are routed by an `IF` node to an Airtable Update that writes `status = error` and `error_reason` without calling Flowise.
- Record Preservation: Added a `Merge` node configured `combineByPosition` to keep the original Airtable `record_id` aligned with Flowise responses so writeback remains traceable.
- Confidence Extraction & Routing: `Normalizing Output` parses the model response and normalizes `confidence` to a 0–100 integer. An IF node routes items with `confidence >= 80` to `status = analyzed` (auto) and lower‑confidence items to `status = needs_review` for human triage.
- Airtable Changes: Added single‑select values `analyzed`, `needs_review`, and `error` to `Alerts.status`, and added `confidence` and `error_reason` fields for traceability. Remember to refresh n8n Airtable nodes after schema changes.
- Docs & Deliverables: Created `week-10/error-handling`, `week-10/confidence-routing`, and `week-10/dashboard-views` with implementation notes and one‑line dashboard view explanations.

Acceptance test notes:
- Verified TEST001/TEST002 were auto‑analyzed with confidence 90; TEST004 routed to `needs_review` with confidence 80; TEST003 was marked `error` with `error_reason` for missing fields. `Scoring_Results` rows were created with preserved `alert_id` linkage.

