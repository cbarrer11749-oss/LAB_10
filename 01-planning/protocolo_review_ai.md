# AI Review Protocol — GSC_AI Project
## Generator Supercenter | N8n + Prompts + JavaScript/Python
## Version 1.0

---

> This protocol is a fixed 5-point checklist to review before committing any AI-generated code, workflow, or prompt to the project. Its purpose is to catch common AI generation errors before they reach production.
> Copy this checklist for every review session. Check every point — do not skip.

---

## Checkpoint 1 — Library and Integration Hallucinations

**Key question:** Does every tool, node, or library referenced actually exist and work in this environment?

**What to review:**
- Every N8n node used in the workflow exists in the installed version of N8n
- Every API integration (Telegram, Google Sheets, Supabase, OpenAI) uses the correct and current endpoint
- Any JavaScript or Python library imported in N8n Code nodes is real, maintained, and compatible with the Node.js version running in N8n
- No invented function names or methods are used on native N8n objects

**How to catch it:**
- Cross-check every node name against the N8n node library
- Verify every npm package at npmjs.com before using it in a Code node
- Test API calls in isolation before wiring them into the workflow

**Red flags:**
- A node or method name you don't recognize
- An npm package with zero downloads or last updated years ago
- An OpenAI parameter that doesn't appear in the official API docs

---

## Checkpoint 2 — Business Logic Errors

**Key question:** Does the logic correctly reflect the real rules of the business — city codes, tank restrictions, routing conditions?

**What to review:**
- Setback values in the system prompt match the Google Sheets source of truth exactly
- Tank restriction rules are correctly implemented: Boca Raton max 250 gal, Hallandale Beach max 750 gal UG / 100 gal AG, South Miami LP tank 5 ft, Weston underground only
- Switch1 routing conditions correctly classify all message intents — no intent falls through to the wrong agent
- The pre-classification prompt returns exactly one of three values: SETBACKS, MEDIA, or GENERAL — and the workflow handles all three without ambiguity
- ALF rules and Coconut Grove exceptions are present and correctly triggered

**How to catch it:**
- Query each city with a known restriction and verify the agent response against the spreadsheet
- Test each Switch1 routing path with at least one real message per intent type
- Test edge cases: a message with no city name, a city outside Miami-Dade and Broward, a message that is both an image and a setback query

**Red flags:**
- A setback value that differs from the Google Sheets source
- A message type that produces no response or routes to the wrong agent
- A tank restriction that is missing from the agent response for a flagged city

---

## Checkpoint 3 — Security

**Key question:** Does the generated code or workflow expose data or grant access it should not?

**What to review:**
- No API keys, tokens, or credentials are hardcoded in any N8n Code node, system prompt, or exported workflow file
- All credentials are stored in N8n's credential manager, not in node parameters as plain text
- The Telegram bot does not process messages from unauthorized users — whitelist check is in place before any message reaches the AI Agent (status: to be defined, but must be verified once implemented)
- The Google Sheets connection does not grant write access unless explicitly required
- The system prompt does not expose internal company processes, pricing, or operational data beyond what is necessary for the agent to function
- No conversation history stored in PostgreSQL contains data that should not persist (e.g., customer personal information entered accidentally by a sales rep)

**How to catch it:**
- Export the N8n workflow as JSON and search for any string that looks like a key, token, or password
- Review each credential node and confirm it references the credential manager, not a hardcoded value
- Check the PostgreSQL memory table structure and confirm what data is being stored per session

**Red flags:**
- Any value starting with "sk-", "Bearer", or "AIza" appearing in plain text anywhere in the workflow
- A credential field filled with text instead of a credential reference
- The bot responding to a test message sent from an unknown Telegram account

---

## Checkpoint 4 — Context Loss and Constraint Compliance

**Key question:** Did the AI respect every constraint and requirement from the brief when generating this output?

**What to review:**
- The agent response format matches the defined structure: city, county, setbacks, survey type, LP tank distance, special conditions
- The agent uses the sales rep's name in every response via {{contact_name}}
- The agent does not invent setback data when a city is not in the database — it explicitly redirects to the permit department
- The distinction between "city outside service area" and "city not in database" is handled with different messages
- The system prompt does not contain instructions that contradict the business constraints defined in the brief
- Each AI Agent node handles only its designated scope — no agent responds to tasks outside its responsibility

**How to catch it:**
- Compare the generated system prompt or workflow output against the Definition of Done checklist in the brief
- Run a test query for a city not in the database and verify the response does not include invented setback numbers
- Run a test query for a city outside Miami-Dade and Broward and verify the response is different from the "not in database" response
- Check that no agent responds to a query type assigned to a different agent

**Red flags:**
- A response that includes setback numbers for a city not in the database
- A response that is missing the sales rep's name
- An agent responding to an image analysis query when it is configured only for setbacks

---

## Checkpoint 5 — Stack-Specific: N8n Workflow Integrity and Prompt Versioning

**Key question:** Is the workflow stable, exportable, and versioned — and does the system prompt reflect the current state of the database?

**What to review:**
- The N8n workflow executes without errors from the Telegram Trigger node all the way to the Send Message node for all three input types: text, audio, and image
- All nodes are connected — no orphaned nodes exist in the canvas
- The workflow has been exported as a JSON backup and saved outside of N8n before any change is deployed
- The system prompt version in the AI Agent node matches the latest version saved in the external document (Google Docs, Notion, or equivalent)
- Any city code update made in Google Sheets has been reflected in the system prompt fallback knowledge base simultaneously
- The PostgreSQL memory node has a defined window size — unbounded memory accumulation is not acceptable

**How to catch it:**
- Run a full end-to-end test after every workflow change before exporting
- Open the external system prompt document and compare it line by line against the prompt currently loaded in the AI Agent node
- Check the PostgreSQL memory configuration and confirm a session window limit is set
- Verify the workflow JSON export is dated and stored in the project repository

**Red flags:**
- A node with no output connection
- The system prompt in N8n differs from the version saved externally
- No workflow export exists prior to the most recent change
- PostgreSQL memory with no window limit defined

---

## Review Session Log

Use this block every time you run the protocol.

```
Date: _______________
Reviewer: _______________
Change reviewed: _______________

[ ] Checkpoint 1 — Library and Integration Hallucinations     PASS / FAIL
[ ] Checkpoint 2 — Business Logic Errors                      PASS / FAIL
[ ] Checkpoint 3 — Security                                   PASS / FAIL
[ ] Checkpoint 4 — Context Loss and Constraint Compliance     PASS / FAIL
[ ] Checkpoint 5 — Stack-Specific Workflow Integrity          PASS / FAIL

Issues found:
_______________

Action taken before commit:
_______________
```
