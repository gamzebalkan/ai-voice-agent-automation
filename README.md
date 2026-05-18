# AI Voice Agent Automation & Orchestration

## 🤖 ElevenLabs → n8n → Zendesk AI Support System

An AI Voice Agent automation system designed for automated inbound call handling and post-call workflow orchestration in travel assistance operations.

The platform combines a real-time voice intake assistant with a rule-based LLM-powered post-call automation layer to reduce manual support workload, automate ticket operations, and streamline customer service workflows inside Zendesk and connected backend systems.

The architecture is intentionally split into two specialized layers to maximize operational safety, maintainability, and scalability.

---

# 🏗️ System Architecture

The infrastructure runs entirely on a self-hosted Railway environment and integrates ElevenLabs, Twilio, n8n, Zendesk, and OpenAI-powered agents.

## 1. In-Call Layer (Real-Time Intake Assistant)

### Tech Stack
- ElevenLabs
- Twilio
- n8n
- Zendesk
- Railway
- OpenAI (ChatGPT 5.5)

### Responsibilities

The In-Call Agent operates during the live phone conversation and is intentionally restricted to safe, read-only operations.

Its primary responsibilities include:
- Customer identity verification
- Ticket lookup and retrieval
- Capturing user requests
- Reading ticket timelines and metadata
- Collecting structured intake information

### Safety Design

To maintain system integrity and avoid unintended live modifications:
- The In-Call Agent operates in a strict **read-only / no-write mode**
- It cannot modify Zendesk tickets
- It cannot perform backend write operations
- It does not provide sensitive operational guidance

### MCP Connectivity

The system securely communicates with n8n workflows through an MCP-based architecture using Bearer Token authentication.

---

## 2. Post-Call Automation Layer (Planner Agent)

### Tech Stack
- n8n
- Zendesk
- Railway
- OpenAI (ChatGPT 5.5)

### Responsibilities

After the phone call ends, the Post-Call Planner Agent orchestrates complex operational workflows across Zendesk and connected backend systems.

The workflow:
1. Receives ElevenLabs `post_call_transcription` webhook events
2. Immediately returns `200 OK` to avoid webhook blocking
3. Retrieves full conversation data from ElevenLabs API
4. Normalizes transcripts into a lightweight structured format
5. Sends runtime data to the LLM Planner Agent
6. Executes rule-based operational workflows

---

# 🧾 Transcript Normalization

Before reaching the LLM layer, raw transcripts are transformed into a clean structured format to reduce context noise and optimize token usage.

Example:

```text
AGENT (12s): Hello, how may I assist you today?
USER (15s): I need help with my travel booking.
```

This preprocessing step improves:
- Agent reasoning quality
- Runtime efficiency
- Context clarity
- Downstream analytics consistency

---

# 🧠 Agent Execution Logic

The Planner Agent evaluates runtime data and routes execution through one of three operational paths.

## 🟢 Path A — Verified Active Sales Ticket Found

Most common operational flow.

Possible actions:
- Create a dedicated call ticket
- Merge tickets into active sales tickets
- Add internal call summaries
- Reopen tickets when necessary
- Update ticket priority
- Escalate billing-related requests
- Send approved follow-up emails
- Synchronize status with backend systems
- Solve tickets when no further action is required

Additional logic:
- Customer frustration detection
- Urgency detection based on travel dates
- Customer profile enrichment
- Phone number synchronization

---

## 🟡 Path B — No Related Ticket or Customer Found

Fallback flow for unidentified callers.

Actions:
- Create isolated Zendesk call ticket
- Attach transcript summary
- Store runtime execution data
- Route for manual triage

---

## 🔵 Path C — Lead Ticket Found

Dedicated flow for unpaid or lead-based customer records.

Actions are intentionally restricted to:
- Lead-safe updates
- Controlled field modifications
- Non-destructive operational handling

---

# 🛠️ Available Agent Tools

The Planner Agent operates through constrained tool execution rules.

Available tools include:
- `zendesk_search`
- `zendesk_get_user`
- `create_call_ticket`
- `merge_tickets`
- `leave_call_summary`
- `send_followup_email`
- `update_application_status`
- `escalate_to_billing`
- `change_priority`
- `reopen_ticket`
- `solve_ticket`

Tool execution paths are strictly controlled to prevent unsafe or invalid actions.

---

# ⚡ Key Features

- Non-blocking webhook ingestion
- LLM-powered operational orchestration
- MCP-based real-time architecture
- Structured transcript normalization
- Strict ticket validation logic
- Read-only live-call safety model
- Automated Zendesk workflow handling
- Analytics-ready structured JSON outputs
- Modular n8n workflow architecture
- Fallback Detective Agent logic

---

# 🕵️ Detective Agent Fallback

If transcript data is unavailable, a dedicated fallback sub-workflow automatically activates.

The Detective Agent attempts to:
- Identify customers via Caller ID
- Locate related Zendesk tickets
- Recover operational context
- Maintain workflow continuity

This ensures operational resilience even under incomplete webhook payload conditions.

---

# 📂 Repository Structure

workflows/
├── Postcall Planner Agent/
└── Inbound Voice Agent/

prompts/
├── Postcall Planner Agent Prompt
└── Inbound Voice Agent Prompt

screenshots/
├── Case 1/
├── Case 2/
└── Additional Examples/

README.md

---

# 🚀 Deployment & Setup

## 1. Environment & Infrastructure
The infrastructure runs entirely self-hosted on **Railway**. Ensure your running instance has public endpoints exposed for your n8n webhooks to interface with Twilio and ElevenLabs.

---

## 2. Secrets & Credentials Management
All secrets and Bearer tokens must be stored securely using n8n Credentials or server environment variables.
* ElevenLabs API Key
* Zendesk API Credentials
* OpenAI API Key
* Internal UCC System authentication keys

Never commit:
- API keys
- `.env` files
- Bearer tokens
- Webhook secrets
- Customer PII
- Real transcripts
- Internal URLs
- Sensitive backend configuration

---

## 3. Import Workflows into n8n

1. Import `[David] Railway - ElevenLabs Calls End [Entry].json`
2. Import all required sub-workflows
3. Reconnect workflow references if necessary
4. Configure credentials inside n8n
5. Enable workflows

---

## 4. Configure ElevenLabs Webhook

Webhook endpoint:

```text
POST https://<your-n8n-instance>/webhook/elce/entry
```

Event:

```text
post_call_transcription
```

---

# 🔒 Security & Compliance

- Strict Bearer Token verification for MCP communication
- No hardcoded secrets inside workflows
- No customer PII stored in repository files
- Public repository examples must remain redacted
- Internal operational links are never exposed externally

The system is designed with operational safety and controlled execution as core architectural principles.

---

# 📌 Notes

- Workflows are designed for manual n8n import
- No CI/CD synchronization is required
- Runtime execution occurs entirely inside n8n
- The repository is intended for architecture demonstration and portfolio purposes

---

# 📄 License

Proprietary

This repository is shared for portfolio and demonstration purposes only.

Unauthorized use, reproduction, or distribution is prohibited.
