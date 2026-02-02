# SupportX AI Assist (Multi-Agent IT Support System)

SupportX AI Assist is a multi-agent, LLM-powered IT support assistant that triages tickets, searches a vectorized knowledge base, and escalates unresolved issues. The app offers a Streamlit chat-style UI where users describe their issue, receive a suggested resolution, and provide feedback. If the user indicates the answer was not helpful, the system generates a ticket ID and notifies IT support via an email escalation workflow.【F:app.py†L1-L118】【F:group_chat.py†L1-L68】【F:tools/send_email.py†L1-L50】

## Story overview

Imagine an employee whose Outlook crashes every time they open it. They launch SupportX AI Assist and describe the issue. The **ClassifierAgent** labels the ticket type, then the **KnowledgeBaseAgent** performs a vector search across historical fixes and returns the most relevant resolution. The assistant summarizes the solution and the employee tries it. If the fix works, they confirm success. If not, SupportX automatically issues a ticket ID and escalates the case to IT support by sending a notification email so a human can step in. The result is a seamless path from self-service to human escalation, without losing context or urgency.【F:app.py†L1-L118】【F:group_chat.py†L1-L68】【F:agents/classifier_agent.py†L1-L17】【F:agents/knowledge_base_agent.py†L1-L38】【F:tools/send_email.py†L1-L50】

## Key features

- **Multi-agent orchestration** with AutoGen GroupChat and a manager agent to coordinate the flow.【F:group_chat.py†L1-L68】
- **Ticket classification** into standard IT categories (network, hardware, software bug, access request, password reset, other).【F:utility/prompt.py†L1-L27】
- **Knowledge base retrieval** via Azure AI Search + OpenAI embeddings to surface similar, proven solutions.【F:tools/knowledge_base_tool.py†L1-L82】
- **User feedback loop** with escalation flow and ticket ID generation.【F:app.py†L7-L118】
- **Email notification** when escalation is required.【F:tools/send_email.py†L1-L50】
- **Streamlit UI** with custom styling for a friendly, support-centric experience.【F:app.py†L9-L29】【F:style.css†L1-L98】

## Architecture at a glance

1. **User submits issue** in the Streamlit UI.
2. **ClassifierAgent** categorizes the ticket.
3. **KnowledgeBaseAgent** calls `search_similar_solution` to retrieve the most relevant fix from Azure AI Search.
4. **Response returned** to the user.
5. **Feedback captured**; unresolved issues are escalated and emailed to IT support.

The orchestration happens through an AutoGen GroupChat with a manager and multiple agents, while the knowledge base lookup uses vector embeddings stored in Azure AI Search.【F:group_chat.py†L1-L68】【F:agents/knowledge_base_agent.py†L1-L38】【F:tools/knowledge_base_tool.py†L1-L82】

## Repository layout

```
.
├── app.py                    # Streamlit UI + feedback loop
├── group_chat.py             # Multi-agent orchestration
├── agents/                   # Agent definitions
├── tools/                    # Knowledge base + email tools
├── utility/                  # LLM config + prompts
├── data/                     # Knowledge base JSON
├── create_and_upload_index.py # Azure AI Search indexing script
├── style.css                 # UI styling
└── requirements.txt          # Python dependencies
```

## Getting started

### 1) Install dependencies

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2) Configure environment variables

Create a `.env` file (or export these variables in your shell) for Azure OpenAI and Azure AI Search. The services in `tools/knowledge_base_tool.py` and `utility/llm_config.py` rely on them.【F:tools/knowledge_base_tool.py†L1-L27】【F:utility/llm_config.py†L1-L15】

```bash
# Azure OpenAI
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_ENDPOINT=...
AZURE_OPENAI_DEPLOYMENT=...
AZURE_DEPLOYMENT_NAME=...
AZURE_API_VERSION=...

# Azure AI Search
AZURE_SEARCH_ENDPOINT=...
AZURE_SEARCH_KEY=...
```

> **Note:** The email escalation helper currently has SMTP credentials hard-coded in `tools/send_email.py`. Replace those with your own secure credentials or refactor to load them from environment variables before production use.【F:tools/send_email.py†L1-L50】

### 3) Build the knowledge base index (one-time setup)

```bash
python create_and_upload_index.py
```

This script creates an Azure AI Search index named `it-ticket-solutions-index`, generates embeddings for each issue in `data/knowledge_base.json`, and uploads the documents.【F:create_and_upload_index.py†L1-L126】

### 4) Run the Streamlit app

```bash
streamlit run app.py
```

Open the app in your browser and describe your issue to get a resolution suggestion.

## How escalation works

If the user clicks **“No, not helpful”**, the app:

1. Generates a ticket ID.
2. Shows the ticket to the user.
3. Sends an escalation email via the notification agent and the email helper.

This behavior is implemented in the Streamlit UI and the email tool wrapper.【F:app.py†L73-L118】【F:group_chat.py†L12-L23】【F:tools/send_email.py†L1-L50】

## Local development tips

- Start by testing the group chat flow with `python group_chat.py` to validate agent orchestration before launching the UI.【F:group_chat.py†L56-L68】
- Keep your `.env` file out of version control; it includes secrets required by Azure services.

## Roadmap ideas

- Move email credentials to environment variables and use a secrets manager.
- Add telemetry and audit logging for escalations.
- Add more knowledge-base content and expand categories.
- Add a dedicated admin console for IT staff.


