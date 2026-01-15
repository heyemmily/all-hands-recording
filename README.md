# All Hands Recording Auto-Poster

Automatically reposts Grain meeting recordings from a private Slack channel to #general.

## Overview

This n8n workflow monitors `#proj-top-goal-emmily` for new Grain recording posts and automatically reposts them to `#general` with a custom message format.

## Setup Guide

### Prerequisites

- n8n cloud instance (archive-team.app.n8n.cloud)
- Slack app with bot token ("n8n AH message")
- Slack credentials configured in n8n ("Slack account 11")

### Required Slack App Permissions

Your Slack app needs these **Bot Token Scopes**:

| Scope | Purpose |
|-------|---------|
| `channels:history` | Read messages in public channels |
| `groups:history` | Read messages in private channels (required for #proj-top-goal-emmily) |
| `chat:write` | Post messages to channels |
| `channels:read` | List public channels |
| `groups:read` | List private channels |

**Important:** After adding scopes, reinstall the app to your workspace.

### Recommended Approach: Schedule Trigger (Polling)

Since the Slack Trigger webhook verification can be problematic with n8n cloud, we use a **polling approach** that checks for new messages every few minutes.

## Workflow Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Schedule Trigger│────▶│ Slack: Get      │────▶│ Filter: Find    │
│ (Every 5 min)   │     │ Channel History │     │ Grain Messages  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Slack: Post to  │◀────│ Format Message  │◀────│ Check if New    │
│ #general        │     │ with @channel   │     │ (Not Processed) │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Step-by-Step Node Configuration

### Node 1: Schedule Trigger

- **Node Type:** Schedule Trigger
- **Settings:**
  - Trigger Interval: Every 5 minutes (adjust as needed)
  - Mode: "Every X Minutes"

### Node 2: Get Channel Messages (Slack)

- **Node Type:** Slack
- **Operation:** Get Many (under Message)
- **Credentials:** Slack account 11
- **Settings:**
  - Resource: Message
  - Operation: Get Many
  - Channel: `#proj-top-goal-emmily` (select from dropdown or use channel ID)
  - Return All: No
  - Limit: 10
  - Additional Fields:
    - Oldest: `{{ $now.minus(10, 'minutes').toSeconds() }}` (look back 10 mins)

### Node 3: Filter for Grain Messages

- **Node Type:** IF
- **Conditions:**
  - Check if message text contains "grain.com" OR "grain.co"
  - Mode: Any (OR)

**Condition 1:**
```
{{ $json.text }}
contains
grain.com
```

**Condition 2:**
```
{{ $json.text }}
contains
grain.co
```

### Node 4: Extract Grain Link (Code Node)

- **Node Type:** Code
- **Language:** JavaScript
- **Code:**

```javascript
// Extract Grain link from message
const items = $input.all();
const results = [];

for (const item of items) {
  const text = item.json.text || '';
  const ts = item.json.ts || '';

  // Extract URL - Grain links can be in different formats
  // Format 1: <https://grain.com/...> (Slack unfurled)
  // Format 2: Plain URL https://grain.com/...

  let grainLink = '';

  // Try to find Slack-formatted URL first
  const slackUrlMatch = text.match(/<(https?:\/\/[^|>]*grain\.[^|>]*)(?:\|[^>]*)?>/) ;
  if (slackUrlMatch) {
    grainLink = slackUrlMatch[1];
  } else {
    // Try plain URL
    const plainUrlMatch = text.match(/(https?:\/\/[^\s]*grain\.[^\s]*)/);
    if (plainUrlMatch) {
      grainLink = plainUrlMatch[1];
    }
  }

  // Also check attachments for links
  if (!grainLink && item.json.attachments) {
    for (const attachment of item.json.attachments) {
      if (attachment.original_url && attachment.original_url.includes('grain')) {
        grainLink = attachment.original_url;
        break;
      }
      if (attachment.title_link && attachment.title_link.includes('grain')) {
        grainLink = attachment.title_link;
        break;
      }
    }
  }

  if (grainLink) {
    results.push({
      json: {
        grainLink: grainLink,
        originalTs: ts,
        messageId: ts.replace('.', '_') // For deduplication tracking
      }
    });
  }
}

return results;
```

### Node 5: Check if Already Processed (Optional but Recommended)

To avoid duplicate posts, use a **Static Data** approach or check a Google Sheet/database.

**Simple Approach - Code Node with Static Data:**

```javascript
// Check if we've already processed this message
const items = $input.all();
const results = [];

// Get workflow static data (persists between executions)
const staticData = $getWorkflowStaticData('global');
const processedMessages = staticData.processedMessages || [];

for (const item of items) {
  const messageId = item.json.messageId;

  if (!processedMessages.includes(messageId)) {
    // Mark as processed
    processedMessages.push(messageId);
    results.push(item);
  }
}

// Keep only last 100 message IDs to prevent unbounded growth
staticData.processedMessages = processedMessages.slice(-100);

return results;
```

### Node 6: Post to #general (Slack)

- **Node Type:** Slack
- **Operation:** Send Message (under Message)
- **Credentials:** Slack account 11
- **Settings:**
  - Resource: Message
  - Operation: Send
  - Channel: `#general` (or test channel first)
  - Message Text:

```
<!channel> Hi all — Here's the All Hands Recording, for those who weren't able to join!

{{ $json.grainLink }}
```

**Important Notes:**
- Use `<!channel>` (not `@channel`) for the mention to work
- Enable "Send as User" if you want it to appear from the bot name

## Complete Workflow JSON

Import the `workflow.json` file in this repository to get the complete workflow.

## Testing

1. **Start with a test channel** instead of #general
2. **Manually trigger** the workflow first to verify it works
3. **Check the execution log** in n8n for any errors
4. **Verify the Grain link extraction** works with your actual Grain message format

## Troubleshooting

### "channel_not_found" Error
- Ensure your Slack app is invited to both channels
- For private channels, the bot must be a member

### No Messages Retrieved
- Verify the channel ID is correct
- Check that `groups:history` scope is added for private channels
- Ensure the "Oldest" timestamp filter isn't excluding messages

### Duplicate Posts
- Implement the deduplication node (Node 5)
- Or use a shorter polling interval with a matching lookback window

### @channel Mention Not Working
- Must use `<!channel>` syntax, not `@channel`
- Ensure `chat:write` scope is present

## Alternative: Event-Based Trigger

If you later want real-time triggering, you can:

1. Set up a **Slack Event Subscription** pointing to a generic n8n Webhook
2. Handle the `url_verification` challenge manually
3. Listen for `message.channels` events

See `docs/event-based-setup.md` for details (if implemented).
