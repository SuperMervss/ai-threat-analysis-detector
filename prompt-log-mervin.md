# Prompt Log - Mervin Laforteza
**Project:** Threat Intelligence Feed Dashboard
**Team:** Original group project, currently executed as a standalone delivery
**My Component:** Specialist (Component 3: Relevance Scoring)
**AI Tools Used:** GitHub Copilot, n8n, Airtable, Flowise, Groq

## How to Use This Log
Add an entry for each significant AI interaction:
- Copilot Chat conversations where you asked it to generate, explain, or debug something
- Moments where Copilot was wrong and you had to fix it (these are the most valuable entries)
- Cases where you refined a prompt to get a better result

Do not log every autocomplete of a bracket or variable name.

## Your First Entry
## [2026-05-09] - Build end-to-end scoring workflow for Checkpoint 2
**Context:** I was working on Component 3 and needed one alert record to flow from Airtable through n8n and Flowise, then write to Scoring_Results and update Alerts status. I had README and audit docs open while configuring Create and Update Airtable nodes in n8n.

**Prompt:**
> help me make the flowise. i got the node 3 to work

**Result:** Copilot gave step-by-step setup for Flowise and n8n mappings, then helped debug Airtable errors. Key fixes were: use array format for linked field alert_id, parse/normalize extracted_entities to a string, and map Update node id to the original Alerts record ID from Airtable Trigger.

**Evaluation:** It worked after iterative debugging. The guidance was accurate for the root issues, especially linked record arrays and expression handling in n8n. The first attempts still failed because of field type mismatch and expression formatting, but each error was resolved with targeted corrections.

**What I changed:** I changed the Create Record mapping so alert_id is passed as an array of Airtable record ID (not TEST001 text), converted extracted_entities from array to string, and fixed the Update Record id expression to use Airtable Trigger id. I also limited the Update node to only status=scored to avoid overwriting fields.

**What I learned:** In Airtable + n8n integrations, data shape matters more than field names alone. Linked fields require arrays, and model outputs need normalization before writing to Airtable. Next time I will validate field types first and add a normalization node earlier to reduce rework.
