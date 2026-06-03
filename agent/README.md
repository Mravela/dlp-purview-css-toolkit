# MDA DLP Migration CSS Assistant — Agent Deployment

## Files in this folder

| File | Purpose |
|---|---|
| `appmanifest.json` | Teams app manifest — packages the agent for Teams sideload |
| `declarativeAgent.json` | Agent definition — name, instructions, knowledge source, conversation starters |
| `instructions.md` | System prompt — agent persona, guardrails, knowledge scope |
| `color.png` | 192×192 app icon (replace with your own) |
| `outline.png` | 32×32 app icon (replace with your own) |

The grounding knowledge source is `knowledge.md` in the root of this repo.

---

## How to deploy (personal sideload — no admin needed)

### Step 1 — Add icons
Replace `color.png` (192×192) and `outline.png` (32×32) with your own icons, or use any PNG placeholders.

### Step 2 — Create the app package zip
Select these files and zip them together (the zip must contain files at the root, not inside a subfolder):
- `appmanifest.json`
- `declarativeAgent.json`
- `instructions.md`
- `color.png`
- `outline.png`

Name it: `dlp-css-agent.zip`

### Step 3 — Sideload in Teams
1. Open Microsoft Teams
2. Click **Apps** in the left rail
3. Click **Manage your apps** (bottom left)
4. Click **Upload an app**
5. Select `dlp-css-agent.zip`
6. Click **Add**

### Step 4 — Use the agent
1. Open **Copilot** in Teams (left rail)
2. Click the agent picker (top of chat)
3. Select **CSS Migration Agent**
4. Start asking questions — try the conversation starters

---

## How to update the agent

1. Edit `knowledge.md` (root of repo) or `instructions.md` (this folder)
2. Re-zip the package files
3. In Teams → Apps → Manage your apps → find CSS Migration Agent → Update

---

## Org-wide deployment (requires IT admin)

Once validated, ask your Teams admin to publish the app via:
Teams Admin Center → Manage apps → Upload → Publish to org
