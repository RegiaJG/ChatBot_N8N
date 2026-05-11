# Nola Customer Support Chatbot — n8n Workflow

An AI-powered customer support chatbot for **Nola**, a restaurant management platform. Built with [n8n](https://n8n.io), Google Sheets as the knowledge base, and Google Gemini as the language model.

---

## Overview

This workflow automates first-level customer support for Nola by answering common questions from restaurant owners and staff. It reads a structured Q&A knowledge base from Google Sheets and uses an AI agent to find and deliver the most relevant answer in a conversational interface.

---

## Architecture

```
User Message
     │
     ▼
┌─────────────────────┐
│   Chat Trigger      │  ← Public chatbot interface
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│  Google Sheets      │  ← Fetches all Q&A rows (executed once per session)
│  Knowledge Base     │
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│  Aggregate Node     │  ← Consolidates rows into a single JSON array
└─────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│             AI Agent (Nola Bot)             │
│                                             │
│  ┌──────────────────┐  ┌─────────────────┐  │
│  │  Google Gemini   │  │ Memory Buffer   │  │
│  │  Chat Model      │  │ (last 10 msgs)  │  │
│  └──────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────┘
     │
     ▼
 Response to User
```

---

## How It Works

1. A user sends a message through the public chat interface embedded in the Nola platform.
2. The workflow fetches all Q&A pairs from the Google Sheets knowledge base (runs once per session).
3. The Aggregate node consolidates the rows into a single structured context.
4. The AI Agent receives the user's question along with the full knowledge base and conversation history.
5. Google Gemini processes the input and returns a relevant, friendly response in Brazilian Portuguese.
6. If the question is not covered by the knowledge base, the bot instructs the user to contact `suporte@nola.com.br`.

---

## Workflow Nodes

| Node | Type | Role |
|---|---|---|
| When chat message received | Chat Trigger | Entry point — receives user messages |
| Buscar Base de Conhecimento | Google Sheets | Reads Q&A data from the spreadsheet |
| Consolidar Base | Aggregate | Merges all rows into one JSON payload |
| Agente CS Nola | AI Agent (LangChain) | Processes the question and generates a response |
| Google Gemini Chat Model | LLM | Powers the AI Agent |
| Memoria da Conversa | Memory Buffer Window | Maintains context for the last 10 messages |

---

## Knowledge Base Structure

The file `planilha_nola.csv` contains **100 Q&A pairs** across **15 support categories**.

| Column | Description |
|---|---|
| `pergunta` | The question (as a user would ask it) |
| `resposta` | The answer provided by the bot |
| `categoria` | Topic category for organization |

### Categories covered

| Category | Topic |
|---|---|
| Acesso | Login, password recovery, multi-device access |
| Suporte | Technical issues, error messages, support contact |
| Pedidos | Orders — creation, editing, cancellation, splitting |
| Cardápio | Menu management — products, categories, combos, photos |
| Financeiro | Revenue, expenses, cash closing, commissions |
| Relatórios | Sales, cancellations, payment method reports |
| Configurações | Users, permissions, establishment settings |
| Integrações | iFood, Rappi, card terminal, PDV integrations |
| Fiscal | NF-e, NFC-e, SAT, fiscal configuration |
| Estoque | Inventory, stock alerts, entry of goods |
| Clientes | Customer registration, loyalty discounts, history |
| Mesas | Table management, comandas, open table view |
| Impressora | Printer setup, kitchen printer, auto-print |
| App | Mobile app usage, offline mode |
| Assinatura | Plan management, upgrades, billing |

---

## Prerequisites

- [n8n](https://n8n.io) instance (self-hosted or cloud)
- Google account with access to Google Sheets API
- Google Gemini API key (via Google AI Studio or Vertex AI)

---

## Setup Instructions

### 1. Import the Knowledge Base into Google Sheets

1. Go to [Google Sheets](https://sheets.google.com) and create a new spreadsheet.
2. Click **File → Import → Upload** and select `planilha_nola.csv`.
3. Choose **Comma** as the separator and click **Import data**.
4. Note the **Spreadsheet ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/<SPREADSHEET_ID>/edit
   ```

### 2. Import the Workflow into n8n

1. Open your n8n instance.
2. Click **+ New Workflow → Import from file**.
3. Select `Chatbot CS Nola.json`.

### 3. Configure Credentials

After importing, two credentials need to be set up:

**Google Sheets (OAuth2)**
- Go to **Settings → Credentials → New Credential → Google Sheets OAuth2 API**.
- Authorize with the Google account that owns the spreadsheet.

**Google Gemini**
- Go to **Settings → Credentials → New Credential → Google Gemini(PaLM) API**.
- Enter your API key from [Google AI Studio](https://aistudio.google.com).

### 4. Update the Spreadsheet ID

1. Open the workflow in n8n.
2. Click the **Buscar Base de Conhecimento** node.
3. Replace the `documentId` value with your own Spreadsheet ID obtained in step 1.

### 5. Activate the Workflow

1. Toggle the workflow to **Active**.
2. Copy the **Chat URL** from the Chat Trigger node to embed or share the chatbot.

---

## Bot Behavior

- **Language:** Brazilian Portuguese
- **Tone:** Friendly and helpful
- **Scope:** Strictly limited to the knowledge base — the bot will not speculate or invent answers
- **Fallback:** Questions outside the knowledge base are redirected to `suporte@nola.com.br`
- **Memory:** Keeps the last 10 messages for contextual conversation

---

## Important Notes

> **Credentials:** All credential IDs in the JSON are tied to the original n8n instance and will not work after import. Reconfigure them as described in the setup steps above.

> **Spreadsheet ID:** The `documentId` in the `Buscar Base de Conhecimento` node references the original spreadsheet. Update it to your own.

> **Webhook ID:** The webhook ID in the Chat Trigger node will be regenerated automatically by n8n upon import.

---

## Files

```
.
├── Chatbot CS Nola.json   # n8n workflow definition
├── planilha_nola.csv      # Knowledge base — 100 Q&A pairs
└── README.md              # This file
```

---

## License

This project is intended for internal use within the Nola platform. Adapt freely for your own support workflows.
