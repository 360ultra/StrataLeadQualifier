# Lead Qualification Agent — n8n Workflow

Telegram bot that qualifies B2B leads against an ICP using **Groq API** (free, fast, llama-3.3-70b).

Webhook-based — responds instantly. No polling needed.

## Deliverable: `n8n-workflow.json`

Import this file into any n8n instance.

---

## Quick Start

### 1. Get your free keys

| Key | Where to get |
|-----|-------------|
| **Telegram Bot Token** | Open [@BotFather](https://t.me/BotFather) → `/newbot` → name it → copy token |
| **Groq API Key** | https://console.groq.com → API Keys → **Create API Key** (free, no credit card) |

### 2. Import the workflow

Import `n8n-workflow.json` into n8n. You'll see 5 nodes.

### 3. Create these credentials

#### Groq API Key
- Type: **HTTP Query Auth**
- Name: `Groq API Key`
- Auth Type: **Query Auth**
- Key: `Authorization`
- Value: `Bearer gsk_YOUR_GROQ_KEY`

#### Telegram Bot (trigger + reply nodes)
- Type: **Telegram Bot API**
- Name: `Telegram Bot`
- Access Token: *your Telegram bot token*

#### Google Sheets (optional)
- Type: **Google Sheets OAuth2 API**
- Auth Method: **Service Account**
- Upload your service account JSON file

### 4. Configure Google Sheets Log node

Set `documentId` to your spreadsheet ID (from the Google Sheets URL).

**Done.** Activate the workflow. Send a message to your Telegram bot.

---

## What It Does

```
Telegram message → Groq (llama-3.3-70b) → Parse JSON → Telegram reply + Google Sheets log
```

### ICP Criteria
1. Company type: services or consulting (NOT products, e-commerce, manufacturing)
2. Minimum 5 employees
3. Location: Spain or Latin America
4. Interest: automation, AI, or digital transformation

### Bot Response
```
✅ CALIFICADO

La empresa cumple con el ICP ya que...
```

OR

```
❌ NO CALIFICADO

La empresa no cumple con el ICP porque...
```

### Google Sheets Log
| A (Timestamp) | B (Original Text) | C (Decision) | D (Reasoning) |
|--------------|-------------------|--------------|----------------|

Timestamp format: `DD/MM/YYYY HH:mm:ss`

---

## Workflow Nodes

1. **Telegram Trigger** — receives incoming messages
2. **Call Groq** — calls Groq API with system prompt + user message
3. **Parse Response** — extracts qualified/reasoning from JSON response
4. **Telegram Reply** — sends formatted reply to user
5. **Google Sheets Log** — appends row to spreadsheet

---

## Setup Details

### Groq Model
- `llama-3.3-70b-versatile`
- Temperature: 0.3
- Max tokens: 500
- System prompt in Spanish, response in Spanish JSON

### Telegram
- Parse mode: plain text (emoji included)
- Webhook mode (not polling)

### Google Sheets
- Service Account credential required
- Sheet name: `StrataBoldLeadQualifier`
- Appends: timestamp, original text, decision, reasoning

---

## Error Handling

- JSON parsing wrapped in try/catch with fallback
- n8n auto-retries on Groq/API failure
- Bot always replies even if parsing fails