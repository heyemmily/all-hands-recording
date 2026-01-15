# Quick Start Guide

## 1. Add Required Slack Scopes

In your Slack app settings (api.slack.com), add these **Bot Token Scopes**:

- `channels:history`
- `groups:history` (required for private channels!)
- `chat:write`
- `channels:read`
- `groups:read`

Then **reinstall the app** to your workspace.

## 2. Invite Bot to Channels

In Slack:
```
/invite @n8n AH message
```
Do this in BOTH:
- `#proj-top-goal-emmily` (source channel)
- `#general` (destination channel)

## 3. Import Workflow

1. Go to your n8n instance: `archive-team.app.n8n.cloud`
2. Create new workflow
3. Click the three dots menu (⋮) → Import from File
4. Upload `workflow.json` from this repo

## 4. Configure Credentials

After importing:
1. Click on "Get Channel Messages" node
2. Select your "Slack account 11" credential
3. Repeat for "Post to #general" node

## 5. Update Channel Names

If your channel names differ:
1. Edit "Get Channel Messages" → change channel to your source
2. Edit "Post to #general" → change channel to your destination

## 6. Test First!

1. Change destination to a test channel (not #general)
2. Click "Test Workflow" to run manually
3. Check execution results for any errors
4. Once working, change destination back to #general

## 7. Activate

Toggle the workflow to **Active** in the top right.

---

## Troubleshooting

| Error | Fix |
|-------|-----|
| `channel_not_found` | Bot not invited to channel, or wrong channel name |
| `missing_scope` | Add required scope in Slack app, reinstall app |
| No messages found | Check if "oldest" filter is too restrictive |
| Duplicate posts | The "Skip Already Posted" node should handle this |

## Custom Message Format

Edit the "Post to #general" node to change the message. Current format:

```
<!channel> Hi all — Here's the All Hands Recording, for those who weren't able to join!

{{ $json.grainLink }}
```

Note: Use `<!channel>` not `@channel` for the mention to work.
