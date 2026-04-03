# MCP Server Setup Guide for VS Code + GitHub Copilot

This guide documents how to connect VS Code (GitHub Copilot) to remote MCP servers — specifically Azure DevOps and Figma — so you can query work items, sprints, and designs directly from Copilot Chat.

---

## What Is MCP?

MCP (Model Context Protocol) is an open standard that lets AI assistants connect to external tools and services. VS Code supports remote MCP servers via HTTPS — no local processes or extensions required beyond GitHub Copilot.

---

## Setup Steps

### 1. Prerequisites

- **VS Code** installed
- **GitHub Copilot** extension installed and signed in
- **Microsoft/Azure DevOps account** with access to the target ADO organization
- **Figma account** (if using the Figma MCP server)

### 2. Create the MCP Config File

VS Code reads MCP server configuration from a single file at:

```
~/Library/Application Support/Code/User/mcp.json
```

> On Windows, this path would be: `%APPDATA%\Code\User\mcp.json`

Create this file if it doesn't exist, or add to it if it does.

### 3. Config File Contents

Paste the following into `mcp.json`, replacing `YOUR-ADO-ORG` with your Azure DevOps organization name:

```json
{
  "inputs": [],
  "servers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "type": "http"
    },
    "ado-remote-mcp": {
      "url": "https://mcp.dev.azure.com/YOUR-ADO-ORG",
      "type": "http"
    }
  }
}
```

**For Ace Hardware's ADO org**, the config is:

```json
{
  "inputs": [],
  "servers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "type": "http"
    },
    "ado-remote-mcp": {
      "url": "https://mcp.dev.azure.com/ACE-PROD-DevOps",
      "type": "http"
    }
  }
}
```

### 4. Authentication

- **Azure DevOps:** Authentication is handled automatically via your existing Microsoft account sign-in in VS Code. No tokens or keys needed — VS Code passes your credentials when it connects to `mcp.dev.azure.com`.
- **Figma:** You may need to sign in to Figma separately via the VS Code Figma extension or browser prompt the first time.

### 5. Reload VS Code

After saving `mcp.json`, either:
- Restart VS Code, or
- Open the Command Palette (`Cmd+Shift+P`) and run **"Developer: Reload Window"**

### 6. Verify the Connection

Open Copilot Chat (`Cmd+Shift+I`) and try a prompt like:

> *"List my assigned work items in Azure DevOps"*

If connected successfully, Copilot will use the MCP tools to query ADO directly.

---

## Notes

- The `mcp.json` file is **global** (applies to all VS Code projects), not workspace-specific. You do not need to re-configure it per project folder.
- To scope MCP servers to a specific workspace only, you can instead place a `mcp.json` file inside the workspace's `.vscode/` folder.
- The ADO MCP server URL format is always: `https://mcp.dev.azure.com/{your-org-name}`
- This setup works in VS Code Copilot only. The Claude desktop app does not yet fully support remote HTTP MCP servers (as of March 2026).

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Copilot doesn't use MCP tools | Reload VS Code window after editing `mcp.json` |
| Authentication errors with ADO | Sign out and back in to your Microsoft account in VS Code (`Accounts` icon in sidebar) |
| Figma not connecting | Ensure you're signed into Figma in the browser and accept any OAuth prompts |
| Config not found | Double-check the file is at `~/Library/Application Support/Code/User/mcp.json` (not inside a project folder) |

---

## Quick Reference

| Server | URL | Auth Method |
|---|---|---|
| Azure DevOps (Ace) | `https://mcp.dev.azure.com/ACE-PROD-DevOps` | Microsoft account (auto) |
| Figma | `https://mcp.figma.com/mcp` | Figma OAuth |
