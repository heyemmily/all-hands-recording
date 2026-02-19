# All Hands Recording Auto-Poster

Grain sends a webhook to n8n when a recording is ready. n8n posts a custom message to Slack.

```
Grain (recording ready) → n8n Webhook → Slack custom message
```

## Setup

### Step 1: Import Workflow into n8n

1. Go to `archive-team.app.n8n.cloud`
2. Create new workflow
3. **⋮ menu** → **Import from File** → upload `workflow.json`
4. Click the **"Post to Slack"** node → select your Slack credential
5. Change the channel ID to your **test channel** (switch to #general later)
6. Click **Save** then toggle workflow to **Active**

### Step 2: Copy the n8n Webhook URL

After activating the workflow:

1. Click the **"Grain Webhook"** node
2. You'll see two URLs:
   - **Test URL** — use this while testing (only works when you click "Listen for Test Event")
   - **Production URL** — use this after testing (works when workflow is active)
3. Copy the **Production URL**. It looks like:
   ```
   https://archive-team.app.n8n.cloud/webhook/grain-recording
   ```

### Step 3: Configure Grain Webhook

1. Go to **Grain** → **Settings** → **Webhooks/API**
2. Add a new webhook
3. Paste the n8n webhook URL from Step 2
4. Set the event to trigger on: **Recording Added** (or equivalent)
5. Save

### Step 4: Test It

**Option A: Use n8n's test mode**

1. In n8n, click the **"Grain Webhook"** node
2. Click **"Listen for Test Event"**
3. Send a test POST request (from browser, Postman, or terminal):
   ```bash
   curl -X POST https://archive-team.app.n8n.cloud/webhook-test/grain-recording \
     -H "Content-Type: application/json" \
     -d '{"recording": {"url": "https://grain.com/share/test-recording-123", "title": "All Hands Test"}}'
   ```
4. Check your test Slack channel — you should see the custom message

**Option B: Trigger from Grain**

1. Make the workflow active (not in test mode)
2. In Grain, trigger a recording event (re-process a recording or wait for the next meeting)
3. Check your test channel

### Step 5: Switch to #general

Once testing works:

1. Edit the **"Post to Slack"** node
2. Change the channel ID to your `#general` channel ID
3. Save

---

## Custom Message

The message posted to Slack:

```
@channel Hi all — Here's the All Hands Recording, for those who weren't able to join!

https://grain.com/share/recording/...
```

To change the message, edit the **"Post to Slack"** node text field.

Use `<!channel>` for @channel mention (not `@channel`).

---

## Workflow Nodes

| Node | Purpose |
|------|---------|
| Grain Webhook | Receives POST from Grain when recording is ready |
| Respond OK | Sends 200 response back to Grain |
| Extract Recording URL | Parses the Grain payload to find the recording link |
| Has Recording URL? | Only continues if a valid URL was found |
| Post to Slack | Posts your custom message to the channel |

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Webhook not receiving | Make sure workflow is **Active** (green toggle) |
| Wrong URL in Slack post | Check n8n execution log → click "Extract Recording URL" to see raw payload from Grain. Adjust the Code node if needed. |
| `channel_not_found` | Use channel ID, not name. Bot must be in the channel. |
| `<!channel>` not working | Ensure `chat:write` scope on your Slack app |

---

## Important: First-Time Grain Payload

The first time Grain sends a webhook, check the **n8n execution log** to see the exact payload structure. You may need to adjust the "Extract Recording URL" code node to match Grain's actual field names. The current code handles multiple common formats as a fallback.

---

## Files

| File | Purpose |
|------|---------|
| `workflow.json` | Import into n8n |
| `README.md` | This guide |
