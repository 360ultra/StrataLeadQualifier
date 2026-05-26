# Lead Qualification Agent — n8n on Render (100% Free)

Telegram bot that qualifies B2B leads using **Google Gemini** (free) running on **n8n** (self-hosted) deployed on **Render free tier**.

**Cost: $0 — no credit card needed for Render or Gemini.**

## Deliverable: `n8n-workflow.json`

Import this file into any n8n instance.

---

## Deploy on Render (no local PC, no ngrok)

### Step 1 — Get your free keys (2 min)

| Key | How to get |
|-----|-----------|
| **Telegram Bot Token** | Open [@BotFather](https://t.me/BotFather) → `/newbot` → name it → copy the token |
| **Gemini API Key** | Go to https://aistudio.google.com/apikey → **Create API Key** (no credit card) → copy it |

### Step 2 — Push this code to GitHub

Create a GitHub repo with these files:

```
LeadQualifier/
├── Dockerfile              # FROM n8nio/n8n  (1 line)
├── n8n-workflow.json       # The workflow
├── README.md
├── .env.example
└── service_account.json.example
```

```bash
cd LeadQualifier
git init
git add .
git commit -m "n8n lead qualifier"
# Create repo at https://github.com/new, then:
git remote add origin https://github.com/YOUR_USER/lead-qualifier.git
git push -u origin main
```

### Step 3 — Deploy on Render

1. Go to https://dashboard.render.com
2. Click **New +** → **Web Service**
3. Connect GitHub → select your repo
4. Fill the form:

| Field | Value |
|-------|-------|
| **Name** | `lead-qualifier` |
| **Runtime** | **Docker** (Render detects the Dockerfile) |
| **Plan** | **Free** ($0/month) |
| **HTTP Port** | `5678` |

5. Click **Advanced** → **Add Environment Variable**:

| Key | Value |
|-----|-------|
| `GEMINI_API_KEY` | Paste your Gemini key |
| `N8N_HOST` | `lead-qualifier.onrender.com` |
| `N8N_PROTOCOL` | `https` |
| `N8N_PORT` | `5678` |
| `N8N_METRICS` | `false` |

> **Note:** Replace `lead-qualifier` with your actual Render service name if different.

6. Click **Create Web Service**

Wait ~3 minutes for the build and deploy.

When done, your n8n instance is live at: `https://lead-qualifier.onrender.com`

### Step 4 — Configure n8n

1. Open `https://lead-qualifier.onrender.com`
2. Create an owner account (email + password) — this is local to your n8n instance, no external service
3. Go to **Settings** → **Credentials** → **Add Credential**
4. Select **Telegram Bot API** → paste your bot token → name it `Telegram Bot` → **Save**

### Step 5 — Import the workflow

1. In n8n, go to **Workflows** → **Import from File**
2. Select `n8n-workflow.json` from the repository
3. The workflow appears on the canvas
4. In the **Telegram Trigger** node → credentials → select `Telegram Bot`
5. In the **Telegram Reply** node → credentials → select `Telegram Bot`
6. Click **Active** toggle at the top → **Save**

### Step 6 — Test

Open Telegram, find your bot, send:

```
Consulting firm, 15 employees, Madrid, they want to automate their sales process.
```

You should get a reply in 1-3 seconds: **Qualified** + reasoning.

Try a rejected lead:

```
Coffee shop in Mexico City, 3 employees, needs a new POS system
```

→ **Not Qualified** — explains why (not services/consulting, under 5 employees, no AI interest)

---

## Google Sheets Logging (optional)

Without this step, the bot still works — it just skips the logging.

1. https://console.cloud.google.com → **New Project**
2. Enable **Google Sheets API** + **Google Drive API**
3. **Credentials** → **Create Service Account** → download the JSON key
4. Create a Google Sheet → **Share** with the service account email (Editor role)
5. Copy the spreadsheet ID from the URL: `https://docs.google.com/spreadsheets/d/THIS_IS_THE_ID/edit`
6. In n8n → **Credentials** → **Google Sheets OAuth2 API** → **Service Account** → upload the JSON
7. In n8n → open the workflow → **Google Sheets Log** node → set `documentId` to your spreadsheet ID → set the credential
8. **Save** and re-activate

---

## Workflow Architecture

```
Telegram Message
       │
       ▼
Telegram Trigger (webhook)
       │
       ▼
Gemini API (HTTP Request)
  ├─ system_instruction = ICP criteria + anti-injection prompt
  ├─ contents = user's lead text
  └─ generationConfig = JSON mode, temp 0.2
       │
       ▼
Parse Response (Code node)
  ├─ Extracts JSON from Gemini
  └─ Returns: chatId, replyText, decision, reason, originalText
       │
       ┌─────────────────────┐
       ▼                     ▼
Telegram Reply        Google Sheets Log
(send result)         (append: date, text, decision, reason)
```

## Risk Mitigation

1. **Error handling** — Code node wraps Gemini response parsing in try/catch. n8n has built-in execution retries. If any node fails, the bot replies with a fallback message instead of crashing silently.

2. **Prompt injection** — The system instruction (`system_instruction.parts[0].text`) explicitly commands Gemini to ignore any user-embedded override instructions. JSON output mode (`responseMimeType: application/json`) constrains the response to structured data, reducing injection surface area.

3. **API costs** — Gemini's free tier gives 60 requests/minute and 1 million token context window at no cost. No credit card is required to use the API. At reasonable lead volumes, the cost is exactly $0 with no surprise-bill risk.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't reply | Check workflow is **Active** (green toggle) |
| "Bad Request: URL is invalid" | Verify `N8N_HOST` and `N8N_PROTOCOL` are set correctly in Render env vars |
| Gemini returns errors | Confirm `GEMINI_API_KEY` is set in Render env vars, then **Deploy latest commit** to restart |
| "Workflow not found after restart" | Render free tier uses ephemeral storage — re-import `n8n-workflow.json` after a restart (takes 15 seconds) |
