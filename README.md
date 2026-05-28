# AI Voice Agent Automation & Orchestration
### 🤖 ElevenLabs → n8n → Zendesk AI Support System

An enterprise-grade automation system designed for automated inbound call handling and post-call workflow orchestration in travel assistance operations. 

By separating a real-time voice intake assistant from an LLM-powered post-call automation layer, the platform maximizes operational safety, reduces manual workload, and streamlines customer service inside Zendesk.

---

## 🏗️ System Architecture

The entire infrastructure is self-hosted on **Railway**, integrating ElevenLabs, Twilio, n8n, Zendesk, and OpenAI (ChatGPT 5.5).

### 1. In-Call Layer (Real-Time Intake Assistant)
* **Tech Stack:** ElevenLabs, Twilio, n8n, Zendesk, Railway, OpenAI
* **Responsibilities:** Operates during live conversations under a strict **read-only / no-write mode** to maintain system integrity.
    * Verifies customer identity and performs ticket lookups.
    * Captures user requests and collects structured intake info.
    * Reads ticket timelines and metadata without making modifications.
* **Connectivity:** Communicates securely with n8n workflows via an MCP-based architecture using Bearer Token authentication.

### 2. Post-Call Automation Layer (Planner Agent)
* **Tech Stack:** n8n, Zendesk, Railway, OpenAI
* **Responsibilities:** Orchestrates complex post-call workflows upon receiving the ElevenLabs `post_call_transcription` webhook.
* **Execution Pipeline:**
    1.  **Ingest:** Receives webhook and immediately returns `200 OK` (non-blocking).
    2.  **Fetch & Normalize:** Retrieves raw data from ElevenLabs API and converts transcripts into a lightweight, structured JSON format to minimize noise and optimize token usage.
    3.  **Process:** Sends runtime data to the LLM Planner Agent.
    4.  **Execute:** Triggers rule-based operational workflows across connected systems.

---

## ⚡ Key Features

* **Dual-Layer Safety Model:** Read-only live-call interactions paired with a secure post-call write layer.
* **Non-Blocking Webhook Ingestion:** Immediate handshake response preventing webhook timeouts.
* **Transcript Normalization:** Pre-processing pipeline that improves agent reasoning and downstream analytics consistency.
* **🕵️ Fallback Detective Agent:** Dedicated sub-workflow that automatically activates to recover operational context and locate related tickets if transcript data is missing.
* **Enterprise Integrations:** Native MCP-based real-time architecture, strict ticket validation, and modular n8n design.

---

## 🧠 Agent Execution Logic

The Planner Agent evaluates runtime data and routes execution through one of three restricted operational paths:

| Execution Path | Trigger Condition | Core Actions |
| :--- | :--- | :--- |
| **🟢 Path A** | Verified Active Sales Ticket Found | • Merge/create tickets & update priority<br>• Add internal call summaries & reopen tickets<br>• Detect customer frustration/urgency<br>• Send approved follow-up emails & sync backend |
| **🟡 Path B** | No Related Ticket or Customer Found | • Create isolated Zendesk call ticket<br>• Attach transcript summary & store execution data<br>• Route to manual triage queue |
| **🔵 Path C** | Lead Ticket Found (Unpaid/Lead Records) | • Restricted to lead-safe updates<br>• Controlled field modifications<br>• Non-destructive operational handling |

---

## 🛠️ Available Agent Tools

Tool execution paths are strictly constrained to prevent unsafe or unauthorized actions:

* **Search & Retrieval:** `zendesk_search`, `zendesk_get_user`
* **Ticket Management:** `create_call_ticket`, `merge_tickets`, `reopen_ticket`, `solve_ticket`, `change_priority`
* **Communications & Logs:** `leave_call_summary`, `send_followup_email`
* **Backend Operations:** `update_application_status`, `escalate_to_billing`

---

## 📂 Repository Structure

```text
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
```
---

## 🚀 Deployment & Setup

### 1. Environment & Infrastructure
The infrastructure runs entirely self-hosted on **Railway**. Ensure your running instance has public endpoints exposed for your n8n webhooks to interface with Twilio and ElevenLabs.



### 2. Secrets & Credentials Management
Configure necessary integration keys (`ElevenLabs`, `Zendesk`, `OpenAI`, `UCC System`) securely via n8n Credentials or server environment variables.

> ⚠️ **Critical Security:** Never commit API keys, `.env` files, webhook secrets, customer PII, or internal system URLs to the repository.


### 3. Import Workflows into n8n

1. Import `[David] Railway - ElevenLabs Calls End [Entry].json`
2. Import all required sub-workflows
3. Reconnect workflow references if necessary
4. Configure credentials inside n8n
5. Enable workflows



### 4. Configure ElevenLabs Webhook

Webhook endpoint:

```text
POST https://<your-n8n-instance>/webhook/elce/entry
```

---

## 🔒 Security & Compliance

The platform is engineered around zero-trust and operational safety boundaries.

* **Secrets & Credentials:** All integration keys (`ElevenLabs`, `Zendesk`, `OpenAI`, `UCC System`) must be stored securely via n8n Credentials or server environment variables. 
* **Data Privacy & PII:** No customer PII, real conversation transcripts, or internal system URLs are stored or exposed within the repository. Public examples remain strictly redacted.
* **Architectural Boundaries:** Strict Bearer Token verification is enforced for all MCP communications, ensuring zero hardcoded secrets within active workflows.

> ⚠️ **CRITICAL SECURITY NOTE:** Never commit `.env` files, API keys, bearer tokens, or webhook secrets to the version control system.

---

## 📌 Notes

- Workflows are designed for manual n8n import
- No CI/CD synchronization is required
- Runtime execution occurs entirely inside n8n
- The repository is intended for architecture demonstration and portfolio purposes

---

## 📄 License

This project is proprietary and confidential. All rights intellectual and material property belong exclusively to **Entry LLC**. 

Authorized personnel may access this repository for evaluation and maintenance purposes subject to company NDAs. Any unauthorized copying, distribution, alteration, or usage of this software via any medium is strictly prohibited.

For complete legal terms and compliance metrics, please refer to the main [LICENSE](LICENSE) file.
