# All Hands Recording Auto-Poster

Every Monday after your All Hands meeting, n8n automatically fetches the recording from Grain and posts it to Slack with your custom message.

```
Monday 12:30pm ET → n8n calls Grain API → Gets latest "All Hands" recording → Shares to team → Posts to Slack
```

Fully automated. No manual steps after setup.

---

## Setup

### Step 1: Create Grain API Credential in n8n

1. Go to **n8n** → **Settings** → **Credentials**
2. Click **Add Credential**
3. Search for **Header Auth**
4. Configure it:
   - **Name**: `Grain API Token`
   - **Header Name**: `Authorization`
   - **Header Value**: `Bearer grain_pat_Ob1NJO3H_YOUR_FULL_TOKEN_HERE`
5. Save

### Step 2: Import the Workflow

1. Download `workflow.json` from this repo
2. In n8n, create a new workflow
3. Click **⋮ menu** → **Import from File** → upload `workflow.json`

### Step 3: Configure the Nodes

**"Get Recordings from Grain" node:**
1. Click on it
2. Under Credential, select your **Grain API Token**

**"Share to Team" node:**
1. Click on it
2. Under Credential, select your **Grain API Token** (same one)

**"Post to Slack" node:**
1. Click on it
2. Select your **Slack credential**
3. Change `REPLACE_WITH_CHANNEL_ID` to your test channel ID (e.g., `C0XXXXXXX`)

### Step 4: Test It

1. Click **Test Workflow** to run it manually
2. Check if it finds your recordings and posts to the test channel
3. If it works, change the channel to `#general`

### Step 5: Activate

Toggle the workflow to **Active**. It will now run automatically every Monday at 12:30pm ET.

---

## How It Works

| Node | What it does |
|------|-------------|
| Monday 12:30pm ET | Triggers every Monday at 12:30pm Eastern |
| Get Recordings from Grain | Calls Grain API, filters for "All Hands" in title |
| Get Latest Recording | Finds the most recent recording from the last 24 hours |
| Has Recent Recording? | Only continues if there's a recording from today |
| Share to Team | Shares the recording with the Archive team so the link works for everyone |
| Post to Slack | Posts your custom message with the recording URL |

---

## Custom Message

Default message:
```
@channel Hi all — Here's the All Hands Recording, for those who weren't able to join!

https://grain.com/share/recording/...
```

To change it, edit the **"Post to Slack"** node's text field.

---

## Configuration Options

### Change the schedule
Edit the **"Monday 12:30pm ET"** node to adjust:
- Time (if your meeting ends earlier/later)
- Day (if All Hands is on a different day)

### Change the search filter
Edit the **"Get Recordings from Grain"** node's JSON body:
```json
{
  "filter": {
    "title_search": "All Hands"
  }
}
```
Change `"All Hands"` to match your meeting title.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| 401 Unauthorized | Check your Grain API token is correct and includes `Bearer ` prefix |
| No recordings found | Check if the title filter matches your meeting name |
| Wrong recording | Adjust the title_search filter to be more specific |
| Doesn't run on Monday | Make sure workflow is Active and timezone is set correctly |
| Channel not found | Use channel ID (starts with C), not channel name |

---

## API Reference

**Grain List Recordings:**
```
POST https://api.grain.com/_/public-api/v2/recordings
Headers:
  Authorization: Bearer <your_token>
  Public-Api-Version: 2025-10-31
  Content-Type: application/json
Body:
  {"filter": {"title_search": "All Hands"}}
```

Response includes `recordings` array with `url` field for each recording.

**Grain Share Recording to Team:**
```
PUT https://api.grain.com/_/public-api/v2/recordings/:recording_id/teams/:team_id
Headers:
  Authorization: Bearer <your_token>
  Public-Api-Version: 2025-10-31
```

Shares the recording with the specified team. Returns `{"success": true}`.
