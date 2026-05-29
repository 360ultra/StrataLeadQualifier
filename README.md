# Lead Qualification Agent — n8n Workflow

Telegram bot that qualifies B2B leads against an ICP using **Google Gemini** (free).
No webhook, no HTTPS needed — uses polling (bot responds within 10 seconds).

## Deliverable: `n8n-workflow.json`

Import this file into any n8n instance.

---

## Quick Start (3 steps)

### 1. Get your free keys

| Key | Where to get |
|-----|-------------|
| **Telegram Bot Token** | Open [@BotFather](https://t.me/BotFather) → `/newbot` → name it → copy token |
| **Gemini API Key** | https://aistudio.google.com/apikey → **Create API Key** (no credit card) |

### 2. Import the workflow

Import `n8n-workflow.json` into n8n. You'll see 6 nodes with red badges.

### 3. Create these 3 credentials

Click each node's credential dropdown → **Create New** → fill:

#### Credential A: `Telegram Bot Token`
| Field | Value |
|-------|-------|
| Type | **Query Auth** |
| Name | `Telegram Bot Token` |
| Parameter Name | `token` |
| Parameter Value | *paste your Telegram bot token* |

#### Credential B: `Gemini API Key`
| Field | Value |
|-------|-------|
| Type | **Query Auth** |
| Name | `Gemini API Key` |
| Parameter Name | `key` |
| Parameter Value | *paste your Gemini API key* |

#### Credential C: `Telegram Bot`
| Field | Value |
|-------|-------|
| Type | **Telegram Bot API** |
| Name | `Telegram Bot` |
| Access Token | *paste your Telegram bot token* |

**Done.** Click **Active** → **Save**. Send a message to your bot on Telegram — it replies within 10 seconds.

---

## How It Works

```
Every 10s → Poll Telegram → New message? → Call Gemini → Parse → Reply + Log
```

**ICP criteria evaluated by Gemini:**
1. Company type: services or consulting
2. Minimum 5 employees
3. Location: Spain or Latin America
4. Interest: automation, AI, or digital transformation

---

## Google Sheets Logging (optional)

Without this, the bot still works — it just logs to n8n execution history.

1. https://console.cloud.google.com → **New Project**
2. Enable **Google Sheets API** + **Google Drive API**
3. **Credentials** → **Create Service Account** → download JSON
4. Create a Google Sheet → **Share** with the service account email (Editor)
5. Copy the **Spreadsheet ID** from the URL
6. In n8n → **Credentials** → **Google Sheets OAuth2 API** → **Service Account** → upload JSON
7. Open workflow → **Google Sheets Log** node → set `documentId` to your sheet ID → select credential

---

## Risk Mitigation

1. **Error handling** — Code node wraps Gemini parsing in try/catch. n8n auto-retries on failure. Bot always replies with a fallback message instead of crashing.

2. **Prompt injection** — System instruction forces Gemini to ignore user-embedded override attempts. `responseMimeType: application/json` constrains output to structured data, reducing injection surface.

3. **API costs** — Gemini free tier: 60 requests/min, 1M tokens, $0. No billing config needed. No surprise charges at any volume.

