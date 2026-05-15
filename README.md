# AI Voice Agent Automation & Orchestration

## 🤖 ElevenLabs → n8n → Zendesk

An enterprise-grade, end-to-end AI Voice Agent system designed for automated inbound call handling in travel assistance. The system architecture is cleanly split into two specialized layers: a real-time **In-Call Agent** for secure data intake and a **Post-Call Automation Layer** powered by an advanced LLM planner to execute safe, rule-based operational workflows within Zendesk and backend systems.

The primary business goal of this live production product is to automate inbound call workflows, eliminating manual data entry and drastically reducing support team overhead.

---

## 🏗️ Architecture Overview

The system runs entirely on a self-hosted **Railway** setup and divides responsibilities between two specialized layers:

### 1. In-Call Layer (Intake Assistant)
* **Tech Stack:** ElevenLabs, Twilio, Model Context Protocol (MCP)
* **Role:** Acts as a real-time intake assistant focused exclusively on basic identity verification, ticket reading, and request capturing. 
* **Safety Design:** To ensure strict system integrity, the In-Call agent operates in a **read-only/no-write execution state**. It does not provide complex guidance, modify data, or execute write operations on live CRM tickets during the live conversation.
* **Tool Connectivity:** Securely connects to n8n workflows through an **MCP architecture** utilizing Bearer Token authentication. 
* *Available In-Call Tools:* Customer identity verification, active ticket retrieval, and ticket event timeline scanning.

### 2. Post-Call Automation Layer (Planner Agent)
* **Tech Stack:** n8n, ChatGPT 5.5
* **Role:** Orchestrates complex, rule-based operations across various support networks right after a call ends.
* **Pre-Processing:** Before data hits the agent, n8n filters out irrelevant webhook metadata and normalizes the raw transcript into a lean, low-noise structured format (`AGENT (Xs): ... / USER (Xs): ...`) to minimize LLM context load.
* **Fallback Logic (Detective Agent):** If a transcript is unavailable, a specialized *Detective Agent* sub-workflow triggers automatically to locate customers and related tickets using data points like the inbound Caller ID.

---

## 🛠️ Post-Call Agent Execution Paths

Powered by ChatGPT 5.5, the Post-Call Planner evaluates the runtime data and routes execution through three distinct operational paths:

### 🟢 Path A: Active Sales Ticket Found (Most Common)
1. **Ticket Creation & Merge:** Creates a fresh call ticket containing the conversation transcript, call summary, ElevenLabs ID, and operational tags, then merges it directly into the open Sales Ticket.
2. **Profile Enrichment:** Fetches profile data via `zendesk_get_user`. If the customer called from an unrecorded number or requested an update, it dynamically updates their customer profile.
3. **Escalation & Actioning:** * **Billing:** Escalates financial requests directly to the billing team.
   * **Backend Systems:** Synchronizes and updates application states on the internal UCC backend system.
   * **Email Automation:** Dispatches context-aware follow-up emails containing links requested by the user.
4. **Queue Prioritization:** Inspects sentiment and details. If the traveler is highly frustrated or their travel date is dangerously close, it auto-escalates the Zendesk ticket priority.
5. **Lifecycle Management:** Reopens closed tickets if further processing is required, or updates the status to **Solved** if no further human agent intervention is necessary.

### 🟡 Path B: No User or Ticket Information Found
* Creates a completely isolated call ticket in Zendesk.
* Attaches the normalized transcript and an AI-generated call summary for manual triage.

### 🔵 Path C: Lead Ticket Found (Potential Customer)
* Routes details down a dedicated pipeline for unpaid / potential customer accounts.
* Restricts updates to safe, lead-specific fields only.

---

## 🔧 System Features & Capabilities

- ⚡ **Non-Blocking Ingestion:** Instantly responds to ElevenLabs `post_call_transcription` events with a `200 OK` to prevent webhook timeouts.
- 🧩 **MCP-Based Architecture:** Uses highly reliable, decoupled Model Context Protocol communication for real-time tool calls.
- 🎛️ **Granular Tool Ecosystem:** Empowered with individual node tools including:
  * `zendesk_search` & `zendesk_get_user`
  * `create_call_ticket` & `merge_tickets`
  * `leave_call_summary` & `send_followup_email`
  * `update_application_status` (UCC Backend)
  * `escalate_to_billing` & `change_priority`
  * `reopen_ticket` & `solve_ticket`

---

## 🚀 Deployment & Setup

### 1. Environment & Infrastructure
The infrastructure runs entirely self-hosted on **Railway**. Ensure your running instance has public endpoints exposed for your n8n webhooks to interface with Twilio and ElevenLabs.

### 2. Secrets & Credentials Management
All secrets and Bearer tokens must be stored natively within your n8n credentials vault or server environment configuration. 
* ElevenLabs API Key
* Zendesk API Credentials (Token/OAuth)
* OpenAI API Key (ChatGPT 5.5 access)
* Internal UCC System authentication keys

### 3. Workflow Import
1. Navigate to your n8n instance dashboard.
2. Import the workflows located inside the `/workflows` directory (`main-workflow.json` and its corresponding sub-workflows).
3. Set your ElevenLabs webhook processing entry point to:
   `POST https://<your-railway-n8n-instance>/webhook/elce/entry`

---

## 🔒 Security & Data Privacy

* **Strict Token Authorization:** All real-time MCP tool invocations use robust Bearer Token verification.
* **No Hardcoded Secrets:** Absolutely no API keys, environment parameters, or configuration files containing sensitive keys should ever be pushed to this repository.
* **PII Compliance:** Customer Personal Identifiable Information (emails, phone records, live tracking IDs) and raw conversation scripts must remain within secure database lines and are never to be used in public tests or markdown readmes.

---

## 📄 License
Proprietary
