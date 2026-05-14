# ai-voice-agent-automation

## 🤖 ElevenLabs → n8n → Zendesk Agent

An end-to-end automation system that processes ElevenLabs post-call transcription webhooks, retrieves full conversation data, normalizes transcripts, and runs an LLM-powered **Post-Call Planner Agent** to execute safe, rule-based actions in Zendesk.

---

## What it does

- Receives ElevenLabs `post_call_transcription` webhook events
- Immediately responds with `200 OK` (non-blocking execution)
- Fetches full conversation data from ElevenLabs API
- Normalizes raw transcript into structured format:
AGENT (Xs): ...
USER (Xs): ...
- Sends structured data to an LLM Post-Call Agent
- The agent can:
- Search and verify Zendesk tickets
- Create call tickets
- Add internal summaries
- Merge or reopen tickets
- Update priority
- Escalate billing cases
- Send follow-up emails (if allowed)

- Outputs a single structured JSON object for logging and downstream analytics

---

## Architecture Overview

1. **Webhook Trigger (ElevenLabs)**
 - Receives `post_call_transcription` event

2. **Immediate Response**
 - Returns 200 OK instantly (avoids blocking webhook)

3. **Fetch Call Data**
 - Retrieves conversation using ElevenLabs API:
   ```
   GET /v1/convai/conversations/{conversation_id}
   ```

4. **Transcript Normalization**
 - Converts raw transcript into readable structured format

5. **Decision Layer**
 - Checks if transcript exists:
   - Yes → run Post-Call LLM Agent
   - No → optional fallback agent

6. **LLM Agent Tool Execution**
 Available tools:
 - zendesk_search
 - zendesk_get_user
 - create_call_ticket
 - merge_tickets
 - leave_call_summary
 - send_followup_email
 - update_application_status
 - escalate_to_billing
 - change_priority
 - reopen_ticket
 - solve_ticket

7. **Execution Output**
 - Returns structured JSON object
 - Used for logging and analytics

---

## Key Features

- ⚡ **Fast webhook response** (non-blocking design)
- 🧠 **LLM-based decision engine**
- 🔐 **Strict tool execution rules**
- 🧾 **Structured transcript normalization**
- 🧯 **Safety-first ticket validation logic**
- 📊 **Analytics-ready JSON output**

---

## Quickstart

### 1. Requirements

- n8n (self-hosted or n8n cloud)
- ElevenLabs API key
- Zendesk API credentials
- OpenAI API key
- Internal UCC system credentials

All secrets must be stored in:
- n8n Credentials
- Environment variables

---

### 2. Import Workflows into n8n

- Import `workflows/main-workflow.json`
- Import all tool workflows (if used as sub-workflows)
- Ensure all workflow references are correctly linked after import

---

### 3. Configure Webhook (ElevenLabs)

Set webhook URL in ElevenLabs:
POST `https://<your-n8n-host>/webhook/elce/entry`


Event:

post_call_transcription


---

## Agent Execution Logic

The agent follows strict execution paths:

### Path A: Verified Sales Ticket Found
- Validate ticket ownership
- Create new call ticket
- Update or merge ticket
- Add call summary
- Possible escalation, priority or contact info updates

### Path B: No Related Ticket Found
- Create new call ticket
- Attach transcript summary

### Path C: Lead Ticket Found
- Handle unpaid / lead-specific logic
- Controlled updates only

---

## Security Rules

- No API keys or secrets are stored in the repository
- All credentials are managed via n8n Credentials
- Do NOT commit:
  - `.env` files
  - API keys
  - webhook secrets
  - customer PII (emails, phone numbers, IDs)
  - raw conversation transcripts

- Use redacted examples only

---

## Notes

- This system is designed for **manual n8n workflow import**
- No automatic sync or CI/CD is required
- Workflows are executed inside n8n only

---

## License
Proprietary
