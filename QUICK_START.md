# Quick Start

## 1. Get Grain API Key

**Grain** → **Settings** → **Integrations** → **API** → **Generate API Key**

Copy the key.

## 2. Add Grain Credential in n8n

**n8n Settings** → **Credentials** → **Add Credential** → **Grain** → Paste API key → Save

## 3. Import Workflow

In n8n: **⋮ menu** → **Import from File** → Upload `workflow.json`

## 4. Connect Credentials

- Click **"New Grain Recording"** node → Select your Grain credential
- Click **"Post to #general"** node → Select your Slack credential

## 5. Activate

Toggle workflow to **Active** (top right).

---

## That's It!

When Grain finishes a recording → n8n posts to Slack automatically.

## Message Posted

```
@channel Hi all — Here's the All Hands Recording, for those who weren't able to join!

https://grain.com/share/recording/...
```
