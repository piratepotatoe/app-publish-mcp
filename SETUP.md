# Setup Guide

This MCP server connects Claude Desktop to App Store Connect (iOS) and Google Play Console (Android), giving you 101 tools to manage your apps directly from Claude.

---

## Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- [Claude Desktop](https://claude.ai/download)
- Apple App Store Connect API key *(for iOS)*
- Google Play service account JSON *(for Android)*

---

## Step 1 — Clone & Build

```bash
git clone https://github.com/piratepotatoe/app-publish-mcp.git
cd app-publish-mcp
npm install
npm run build
```

Note the full path to the cloned folder — you'll need it in Step 3.  
Example: `/Users/yourname/app-publish-mcp`

---

## Step 2 — Prepare Credentials

### Apple (App Store Connect)

1. Go to [App Store Connect → Users and Access → Integrations → App Store Connect API](https://appstoreconnect.apple.com/access/integrations/api)
2. Create an API key with **Admin** role
3. Download the `.p8` file and note:
   - **Key ID** (e.g. `ABC123DEF4`)
   - **Issuer ID** (e.g. `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
   - Path to the downloaded `.p8` file

### Google Play

1. Go to [Google Play Console → Setup → API access](https://play.google.com/console/developers/api-access)
2. Link to a Google Cloud project
3. Create a **Service Account** with **Release Manager** role
4. Download the JSON key file
5. Grant the service account access in Play Console → Users and permissions

---

## Step 3 — Configure Claude Desktop

Open (or create) `~/.claude.json` and add the `mcpServers` section:

```json
{
  "mcpServers": {
    "app-publish-mcp": {
      "type": "stdio",
      "command": "node",
      "args": ["/FULL/PATH/TO/app-publish-mcp/dist/index.js"],
      "env": {
        "APPLE_KEY_ID": "YOUR_KEY_ID",
        "APPLE_ISSUER_ID": "YOUR_ISSUER_ID",
        "APPLE_P8_PATH": "/full/path/to/AuthKey_XXXXXXXX.p8",
        "GOOGLE_SERVICE_ACCOUNT_PATH": "/full/path/to/service-account.json"
      }
    }
  }
}
```

**Replace:**
- `/FULL/PATH/TO/app-publish-mcp` → actual path from Step 1
- `YOUR_KEY_ID` → Apple Key ID
- `YOUR_ISSUER_ID` → Apple Issuer ID
- `/full/path/to/AuthKey_XXXXXXXX.p8` → path to downloaded .p8 file
- `/full/path/to/service-account.json` → path to Google service account JSON

> If `~/.claude.json` already exists with other entries, add `"mcpServers"` alongside them without overwriting existing keys.

---

## Step 4 — Install the Skill

1. Download `app-store-manager.skill` from this repo's [releases](../../releases) or the `skills/` folder
2. Open Claude Desktop
3. Drag & drop the `.skill` file into the Claude Desktop window  
   *(or go to Settings → Skills → Install from file)*
4. Restart Claude Desktop

---

## Step 5 — Verify

Restart Claude Desktop, then ask:
> *"List all my iOS apps"*

Claude should use the `apple_list_apps` tool automatically.

---

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `APPLE_KEY_ID` | iOS only | App Store Connect API Key ID |
| `APPLE_ISSUER_ID` | iOS only | App Store Connect Issuer ID |
| `APPLE_P8_PATH` | iOS only | Path to the `.p8` private key file |
| `GOOGLE_SERVICE_ACCOUNT_PATH` | Android only | Path to Google service account JSON |

You can omit Apple or Google variables if you only need one platform.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `401 Unauthorized` on Apple tools | Check Key ID, Issuer ID, and .p8 file path |
| `403 Forbidden` on `apple_create_app` | App creation requires App Store Connect UI — API restriction |
| Google tools fail silently | Make sure to call `google_create_edit` first, and `google_commit_edit` after changes |
| MCP server not showing in Claude | Check `~/.claude.json` syntax, restart Claude Desktop |
| `node: command not found` | Use full path to node (find with `which node`) |
