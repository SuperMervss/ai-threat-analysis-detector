# Checkpoint 2 Readiness Assessment

## Status: **READY** ✅

**Why?** The core Checkpoint 2 requirement is now met: one alert record flows from Airtable through n8n + Flowise scoring and writes back to Airtable automatically, then updates status to `scored`.

---

## What's Working ✅
- **Airtable schema live:** `Alerts` and `Scoring_Results` tables are configured with linked record handoff
- **n8n trigger working:** New `Alerts` records are detected automatically
- **Flowise call integrated:** Node 3 executes and returns parsable scoring payload
- **Normalization logic working:** Output is transformed to Airtable-compatible fields
- **Create record working:** Scoring results are written to `Scoring_Results`
- **Update record working:** Original `Alerts.status` is updated to `scored`
- **End-to-end proof complete:** Full run succeeded with record linkage visible in Airtable

---

## Critical Gaps (Must Fix Before Checkpoint 2) 🔴

| Gap | Action Item | Owner | Hours |
|-----|------------|-------|-------|
| **Scoring quality defaults too low** | Tighten Flowise prompt/rules so obvious phishing does not default to `informational` with `0` values | You | 0.5 |
| **Repeatability evidence** | Run at least 2 more records (edge case + bad input) and capture outputs for demo and report | You | 0.5 |
| **Artifact export pending** | Export final n8n workflow JSON and commit to repo under component-3-scoring | You | 0.25 |

---

## Schema Issues Found 🔧

**Current state:** Core schema and handoff are functioning. Remaining improvements:

1. **Linked record shape was resolved:** `Scoring_Results.alert_id` must be passed as an array containing the Airtable record ID (not `TEST001` text)
2. **Entity parsing was resolved:** `extracted_entities` must be converted to a string before Airtable write when model returns an array
3. **Status transition is now verified:** `Alerts.status` update to `scored` succeeded in live execution
4. **Optional cleanup:** Standardize link field naming to avoid duplicated helper field labels in Airtable views

---

## Recommended Fix Order (Next 2 Hours)

1. **Run two additional test alerts**
- One edge case and one malformed case
- Confirm no workflow break and status still transitions to `scored`

2. **Improve Flowise scoring prompt**
- Add explicit phishing/routine rules and prevent all-zero default output unless input is empty

3. **Export final workflow artifacts**
- Save n8n export JSON to repo and capture final screenshot set for demo evidence

---

## Test Data Gaps 📊

**Current status:** At least one successful end-to-end test (`TEST001`) has been completed.

Run these additional records for stronger demo evidence:

| Test Case | alert_id | threat_description | source | Expected Outcome |
|-----------|----------|-------------------|--------|-----------------|
| **Normal: High-risk phishing** | TEST001 | "Phishing email with spoofed Amazon domain detected targeting finance@acmecorp.com. Sender IP traced to Moscow." | email_gateway | severity_level = HIGH, relevance_score = 85+ |
| **Edge: Ambiguous threat** | TEST002 | "Routine firewall rule update completed on fw-01 during scheduled maintenance window." | firewall_log | severity_level = LOW, relevance_score = 20-40 |
| **Bad data: Missing fields** | TEST003 | "" (empty) | "" (empty) | Component 3 should gracefully fail and log error to Airtable or n8n |

---

## Acceptable Error Rate

For Checkpoint 2, **your threshold should be:**
- **Accuracy goal:** 70%+ on severity classification (low/medium/high/critical)
- **Error handling:** Any failed record should log error to Airtable or n8n console, not crash the workflow
- **False positives:** Acceptable < 25% for phishing (you're at ~20% from Week 5)

---

## 7-Hour Reality Check

**Completed reality check:**
1. ✅ Airtable schema (Alerts + Scoring_Results tables)
2. ✅ n8n trigger and handoff workflow
3. ✅ Flowise scoring call integrated and normalized
4. ✅ One end-to-end test record flowing through

**Demo scenario for Checkpoint 2:**
> "I create an alert in Airtable. n8n automatically triggers my Groq-powered scoring engine. Component 3 analyzes the threat, extracts entities using NER, retrieves context via RAG, assigns a relevance score and severity level, and writes the result back to Airtable—all automatically, no manual steps."

**That's exactly what Checkpoint 2 asks for.**

---

## Lab-Based Build Plan

This version matches the labs already in your repo and keeps the demo small enough to finish quickly.

### What the labs already give you
- **Week 4:** Evidence that you tested multiple models for threat relevance and scored sample alerts
- **Week 5:** Evaluation work showing how you compared generic and fine-tuned behavior
- **Week 7:** Flowise RAG setup that can serve as the scoring engine for Component 3

### Step-by-step build order
1. **Prepare Airtable as the input source.** Add one alert record with `status = pending_scoring`.
2. **Use n8n as the trigger layer.** Trigger on the Airtable `Alerts` table when `alert_id` appears.
3. **Pass the alert into Flowise.** Send `threat_description` as `input_text` so Flowise can score it.
4. **Make Flowise return structured output.** Have the chain return `relevance_score`, `severity_level`, `extracted_entities`, `threat_category`, `confidence`, and `recommended_actions`.
5. **Write the result back to Airtable.** Create the matching row in `Scoring_Results` and update the alert status to `scored`.

### Minimal demo path
- **Input:** one manual Airtable record
- **Processing:** one n8n trigger plus one Flowise scoring call
- **Output:** one row in `Scoring_Results`
- **Proof:** screenshot before scoring and screenshot after scoring

### If time is short
- Do not build full Components 1, 2, and 4 as separate systems.
- Simulate them by manually entering the alert in Airtable and letting n8n + Flowise do the scoring handoff.
- Keep the Flowise prompt simple so the JSON output stays stable.

### Exact field format to use
- `alert_id`: text
- `threat_description`: text
- `source`: text
- `timestamp`: date or text
- `input_text`: copy of `threat_description` for Flowise compatibility

### Exact scoring output to store
- `score_id`: generated text ID
- `alert_id`: linked Airtable record
- `relevance_score`: number
- `severity_level`: critical, high, medium, low, or informational
- `extracted_entities`: text or JSON string
- `threat_category`: text
- `confidence`: percent or number from 0 to 100
- `recommended_actions`: text
- `scored_at`: date/time

---

## Files to Create/Modify

1. **`component-3-scoring/n8n-workflow.json`** — Save your n8n workflow export here (add to GitHub)
2. **`component-3-scoring/api-integration.md`** — Document the Component 3 API endpoint (what n8n calls)
3. **`copilot-instructions.md`** — Update the "Current State" section with: "✅ Airtable schema live | ✅ n8n workflow operational | ✅ End-to-end demo record: TEST001"

---

## After Checkpoint 2: Next Steps

Once you pass:
1. **Fix the ~20% phishing false positive rate** (add more training data or tune threshold)
2. **Improve NER accuracy** on cybersecurity terms ("SSH" → "SSH" not "SS")
3. **Add Components 1, 2, 4** as real workflows (instead of manual injection)
4. **Implement real Slack alerting** for Component 4

---

## Interview Responses (Raw Data)

### Q1: Project Name & Problem
**Answer:** Yes (confirmed as Threat Intelligence Feed Dashboard)

### Q2: Your Role
**Answer:** I am doing the Relevance Scoring (Component 3)

### Q3: Airtable Schema Accuracy
**Answer:** Yes, these are correct. Requested more suggestions on what to add.

### Q4: What's Working per Component
**Answer:** Component 3 works on test alerts. This is the only component I am working on.

### Q5: What's NOT Working per Component
**Answer:** I haven't tested the Airtable connection yet.

### Q6: Component 1→2 Handoff
**Answer:** Not Applicable since I am not assigned to this.

### Q7: Component 2→3 Handoff
**Answer:** I will do simulations for this part since I am only doing component 3. I will simulate component 2 output and use that as an input for component 3.

### Q8: Field Mismatches
**Answer:** Yes (but specifics unclear—see schema section above).

### Q9: Test Data Records
**Answer:** I don't have any test records right now.

### Q10: Biggest Worry
**Answer:** I am worried about the inputs/outputs mismatch and also the classification.

### Follow-up: Field Mismatches Detail
**Answer:** Since I don't do component 2, I am assuming that its output is simply input_text.

### Follow-up: Component 2→3 Simulation Plan
**Answer:** I'll create 5 mock JSON records with threat_description, confidence_score, and threat_category, then feed them manually into Component 3 to test.

### Follow-up: Airtable Connection Blocker
**Answer:** I just haven't created the n8n workflow yet. I have the API keys.

### Follow-up: Classification Acceptable Error Rate
**Answer:** That I'm not sure of. Seeking help to answer this.

### Follow-up: Other Teams' Status
**Answer:** No, they are not. I am the only one left in my group and therefore this would be a standalone project.

### Follow-up: Solo Strategy for Checkpoint 2
**Answer:** I will inject test data. I believe that I do not have the time to build components 1, 2, 4.

### Follow-up: Component 2 Output Spec
**Answer:** input_text is fine.

### Follow-up: Airtable Current State
**Answer:** I need help setting up the airtable and requesting assistance on what to put there.

### Follow-up: Timeline
**Answer:** I am late actually and the demo is in 2 days. But I only have like 7 hours to do it.

### Follow-up: n8n Experience
**Answer:** I am fairly confident in fixing this by myself.

---

**Updated:** May 9, 2026  
**Project:** Threat Intelligence Feed Dashboard  
**Component:** 3 (Relevance Scorer)  
**Owner:** Mervin Laforteza
