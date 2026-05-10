# Prompt Log - Mervin Laforteza
**Project:** Threat Intelligence Feed Dashboard
**Team:** Originally a group project, currently executed as a standalone delivery
**My Component:** Specialist (Component 3: Relevance Scoring)
**AI Tools Used:** GitHub Copilot, n8n, Airtable, Flowise, Groq

## How to Use This Log
Add an entry for each significant AI interaction:
- Copilot Chat conversations where you asked it to generate, explain, or debug something
- Moments where Copilot was wrong and you had to fix it (these are the most valuable entries)
- Cases where you refined a prompt to get a better result

Do not log every autocomplete of a bracket or variable name.

## Prompt Log Entries
## [2026-05-09] - Build end-to-end scoring workflow for Checkpoint 2
**Context:** I was working on Component 3, with the README, checkpoint audit, and Airtable schema open, and needed one alert record to flow from Airtable through n8n and Flowise, then write to `Scoring_Results` and update `Alerts.status`.

**Prompt:**
> help me make the flowise. i got the node 3 to work

**Result:** Copilot gave step-by-step setup for Flowise and n8n mappings, then helped debug Airtable errors. Key fixes were: use array format for linked field alert_id, parse/normalize extracted_entities to a string, and map Update node id to the original Alerts record ID from Airtable Trigger.

**Evaluation:** It worked after iterative debugging. The guidance was accurate for the root issues, especially linked record arrays and expression handling in n8n. The first attempts still failed because of field type mismatch and expression formatting, but each error was resolved with targeted corrections.

**What I changed:** I changed the Create Record mapping so alert_id is passed as an array of Airtable record ID (not TEST001 text), converted extracted_entities from array to string, and fixed the Update Record id expression to use Airtable Trigger id. I also limited the Update node to only status=scored to avoid overwriting fields.

**What I learned:** In Airtable + n8n integrations, data shape matters more than field names alone. Linked fields require arrays, and model outputs need normalization before writing to Airtable. Next time I will validate field types first and add a normalization node earlier to reduce rework.

## [2026-05-09] - Refine scoring prompt and finalize checkpoint documentation
**Context:** After the workflow started working end-to-end, I noticed the LLM was too generic at first and the workflow docs still described the older Week 8 state. I needed the scoring prompt to behave consistently on phishing, routine maintenance, and empty input, and I needed the project docs to reflect the actual Checkpoint 2 results and the manual Airtable test-sample setup.

**Prompt:**
> give me a better prompt 

**Result:** Copilot helped me tighten the Flowise prompt so it uses only `threat_description`, then later I confirmed the workflow output on three test alerts. It also helped identify that the HTTP node was losing record IDs, so I added a Merge + normalization step to preserve Airtable linkage and clean up array fields before writing to Airtable.

**Evaluation:** This was useful because it moved the scoring from inconsistent generic outputs to stable, input-dependent JSON. The resulting workflow now writes the correct fields to Airtable, and the docs were updated to match the live behavior.

**What I changed:** I updated the prompt to force structured scoring output, added Merge-based record ID preservation, normalized `extracted_entities` and `recommended_actions` for Airtable, and updated `README.md`, `docs/checkpoint2-audit.md`, `docs/checkpoint2-results.md`, and `copilot-instructions.md` to reflect the completed Checkpoint 2 scope.

**What I learned:** In n8n, each node can change the item shape, so preserving IDs across HTTP calls is critical. I also learned that documentation needs to match the actual demo scope: for my solo checkpoint, manual Airtable test samples are acceptable for ingestion simulation, as long as the scoring workflow is proven end-to-end.

## [2026-05-09] - Fix Flowise quota error
**Context:** While testing the HTTP Request node against Flowise Cloud, the workflow failed with a service error instead of returning scored JSON. I needed to understand whether the issue was in n8n, the request payload, or the Flowise account limit before I could trust the scoring pipeline.

**Prompt:**
> what is this

**Result:** Copilot explained that the error meant Flowise Cloud had hit its prediction quota/rate limit (`predictions limit exceeded`) and that the HTTP node itself was not the root cause. That clarified that the fix was operational rather than a mapping issue.

**Evaluation:** This was useful because it prevented me from debugging the wrong layer. The workflow failure was caused by Flowise capacity, not Airtable or JSON parsing.

**What I changed:** I treated the error as a Flowise quota issue, not a bad n8n mapping. I also documented the limitation in the README/audit notes so the demo report reflects the real constraint.

**What I learned:** When a chain fails in n8n, I need to distinguish between data-shape bugs and upstream service limits. A clean HTTP setup can still fail if the LLM service is rate-limited.

## [2026-05-09] - Preserve Airtable record IDs through scoring flow
**Context:** The workflow output looked correct in the HTTP Request node, but the downstream Airtable create step lost the original `record_id`. I needed to keep the Airtable ID alive across `Set -> HTTP Request -> Normalize Output` so linked records could be written correctly.

**Prompt:**
> help me do the merge

**Result:** Copilot recommended adding a Merge node after the HTTP Request and using combine-by-position so the original Set item and the Flowise response could be carried together. That let me keep `record_id` available for the create/update nodes.

**Evaluation:** This solved the item-shape problem that was causing `alert_id` and `score_id` to break. The fix was practical and matched the way n8n handles node outputs.

**What I changed:** I inserted a Merge node, preserved `record_id` in the normalized output, and mapped `alert_id` as an array containing the Airtable record ID. I also updated the create node so `score_id` is generated from the preserved record ID.

**What I learned:** In n8n, the apparent output of one node is not enough if a later node replaces the item context. When linking Airtable rows, preserving the original record ID is more important than the model output shape.

## [2026-05-09] - Fix Airtable create record mapping
**Context:** The create node started failing even when the normalized output looked right because the linked record field and long-text fields were mapped to the wrong value types. I needed to align the Airtable schema with the actual output from the normalization node.

**Prompt:**
> here are my outputs

**Result:** Copilot identified that `recommended_actions` was being passed as an array instead of a string and that `alert_id` needed a linked-record array built from the Airtable record ID. After the mapping changes, the rows were created successfully.

**Evaluation:** This confirmed the end-to-end scoring path. Once the data types matched the Airtable schema, the workflow stopped erroring and the score rows appeared correctly.

**What I changed:** I mapped `recommended_actions_text` and `extracted_entities_text` into Airtable long-text fields, used the preserved `record_id` for `score_id`, and passed `alert_id` as `[record_id]`.

**What I learned:** Airtable writeback is strict about field types. Even when the values look readable in JSON, the linked record and long-text fields need exact shapes to avoid service errors.

## [2026-05-09] - Update project documentation after checkpoint results
**Context:** After the scoring workflow and test records were working, I needed the project documentation to match the actual Checkpoint 2 state. I wanted the README, audit notes, and supporting docs to reflect the completed component scope instead of the older Week 8 planning state.

**Prompt:**
> update the readme file now

**Result:** Copilot helped me rewrite the project README so it reflected the working end-to-end scoring flow, the manual Airtable test-sample setup for my solo scope, and the verified TEST001 to TEST003 results. It also helped update the checkpoint audit and current-state docs so they matched the live workflow.

**Evaluation:** This was important because the documentation had to stay consistent with the demo behavior. The AI assistance was accurate once I described the checkpoint changes clearly.

**What I changed:** I updated the README, the checkpoint audit, the results file, and `copilot-instructions.md` to show what was working, what remained open, and which parts were simulated for the Component 3 scope.

**What I learned:** Good documentation is part of the deliverable. If the workflow is working but the README still describes an older state, the project looks unfinished even when the implementation is ready.
