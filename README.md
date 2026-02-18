# All Hands Recording Auto-Poster

Automatically share Grain meeting recordings to Slack when a recording is ready.

## How It Works

Grain has a **native Slack integration** that automatically posts recordings to a Slack channel when they're ready. No n8n or third-party tools needed for the basic flow.

```
Meeting ends → Grain processes recording → Grain posts to Slack channel
```

## Setup Guide

### Step 1: Connect Grain to Slack

1. Open **Grain** (grain.com) and log in
2. Go to **Settings** → **Integrations** → **Slack**
3. Click the green **"Connect Slack"** button
4. Authorize Grain to access your Slack workspace
5. Select the correct workspace if you have multiple

> This only needs to be done once for your entire workspace.

### Step 2: Set Up the Automation

1. In Grain, go to **Automations** → **Slack**
2. Click **"Get Started"**
3. You'll see options to configure:

**Choose which meetings to share:**
- Filter by **Owner** (e.g., only your meetings)
- Filter by **People/Participants** (e.g., only meetings with the whole team)
- Filter by **Tags** (e.g., tag your All Hands as "All Hands")

**Choose the destination channel:**
- For testing: Select `#proj-top-goal-emmily` (your private test channel)
- For production: You'll switch this to `#general` later

4. Click **"Add Automation"**

### Step 3: Make Sure Meetings Are Shared with Workspace

**Important:** Grain's Slack automation only works for meetings that are **shared with the workspace**.

1. Go to Grain **Settings** → **Sharing**
2. Ensure your recordings are set to share with your workspace
3. If not, enable workspace sharing for your All Hands recordings

### Step 4: Test It

To verify the automation works:

1. Wait for the next Monday All Hands meeting (11:15 AM ET)
2. After the meeting ends and Grain finishes processing (~30-60 min), check `#proj-top-goal-emmily`
3. You should see Grain's automated post with the recording link and summary

**Alternative test method:**
- Re-share an existing recording in Grain to trigger the automation
- Or record a quick test meeting and let Grain process it

### Step 5: Switch to #general (After Testing)

Once you've confirmed it works:

1. Go back to Grain → **Automations** → **Slack**
2. Edit your automation
3. Change the channel from `#proj-top-goal-emmily` to `#general`
4. Save

That's it. Fully automated, every Monday after All Hands.

---

## What Grain Posts

Grain's automated Slack message includes:
- AI-generated meeting summary
- Key points and action items
- Link to the full recording
- Link to the transcript

> Note: This uses Grain's built-in message format. You cannot customize
> the exact wording (e.g., no custom "Hi @channel" message). If you need
> a custom message format, see the "Custom Message Format" section below.

---

## Custom Message Format (Optional, Advanced)

If you want a custom message like:

```
@channel Hi all — Here's the All Hands Recording, for those who weren't able to join!
[link]
```

You'll need to add an n8n workflow on top. See `workflow-custom-format.json` for a
workflow that:

1. Grain posts to `#proj-top-goal-emmily` (automatically)
2. n8n watches that channel and reposts to `#general` with your custom message

This is the "Approach A" fallback if Grain's native format doesn't work for you.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Automation not firing | Ensure the recording is shared with workspace |
| Channel not in dropdown | Grain's Slack app must be invited to the channel |
| Only some meetings post | Check your filters (owner, participants, tags) |
| Recording takes too long | Grain processing time varies; usually 15-60 min |

---

## Files

| File | Purpose |
|------|---------|
| `README.md` | This guide (Grain native Slack setup) |
| `workflow-custom-format.json` | Optional n8n workflow for custom message formatting |

---

## References

- [Set up Slack integration with Grain](https://support.grain.com/en/articles/9248458-set-up-slack-integration-with-grain)
- [Slack Automations for Meeting Summaries](https://support.grain.com/en/articles/8055236-how-to-setup-slack-automations-to-receive-automated-meeting-summaries)
- [Grain + Slack Integration Page](https://grain.com/integrations/slack)
