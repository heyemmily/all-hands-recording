# All Hands Recording Auto-Poster

Automatically posts Grain meeting recordings to Slack with a custom message when a recording is ready.

## How It Works

```
Grain (recording ready) → n8n → #general with custom message
```

Just 2 nodes. No polling. No intermediate channels.

## Setup Guide

### Step 1: Get Your Grain API Key

1. Go to **Grain** → **Settings** → **Integrations** → **API**
2. Click **Generate API Key**
3. Copy the key (you'll need it for n8n)

### Step 2: Add Grain Credentials in n8n

1. Go to your n8n instance: `archive-team.app.n8n.cloud`
2. Click **Settings** (gear icon) → **Credentials**
3. Click **Add Credential** → Search for **Grain**
4. Paste your API key
5. Save it

### Step 3: Import the Workflow

1. Create a new workflow in n8n
2. Click **⋮ menu** (top right) → **Import from File**
3. Upload `workflow.json` from this repo

### Step 4: Configure the Nodes

**Grain Trigger Node:**
1. Click on "New Grain Recording" node
2. Select your Grain credential
3. (Optional) Add filters if you only want specific meetings

**Slack Node:**
1. Click on "Post to #general" node
2. Select your Slack credential
3. Change channel if needed (use channel ID for private channels)

### Step 5: Activate

Toggle the workflow to **Active** (top right).

Done! When Grain finishes processing a recording, it will automatically post to your Slack channel.

---

## Custom Message Format

The default message is:

```
@channel Hi all — Here's the All Hands Recording, for those who weren't able to join!

[Grain recording link]
```

To customize, edit the "Post to #general" node and change the **Message Text** field.

### Available Variables from Grain

You can use these in your message:
- `{{ $json.recording.url }}` - Link to the recording
- `{{ $json.recording.title }}` - Meeting title
- `{{ $json.recording.date }}` - Meeting date
- `{{ $json.recording.duration }}` - Recording length

---

## Filtering (Optional)

If you only want to post certain recordings (like All Hands meetings), edit the Grain Trigger node and add filters by:
- Meeting title contains "All Hands"
- Specific participants
- Tags

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Workflow not triggering | Make sure workflow is **Active** (green toggle) |
| No Grain credential option | Add Grain credential in Settings → Credentials first |
| `channel_not_found` | Bot not in channel, or use channel ID instead of name |
| Wrong recording URL | Check Grain's webhook payload structure - may need to adjust `$json.recording.url` path |

## Testing

To test without waiting for a real meeting:
1. Make the workflow active
2. In Grain, re-process an old recording or upload a test video
3. Check n8n execution log to see if it triggered

---

## Files

| File | Purpose |
|------|---------|
| `workflow.json` | Import this into n8n |
| `README.md` | This guide |
| `QUICK_START.md` | Condensed setup steps |
