---
name: autonexus:setup
description: One-time setup wizard — configures Obsidian MCP server for AutoNexus. Gets API key, detects platform, writes .mcp.json, verifies connection.
argument-hint: "[--global] [--api-key <key>]"
---

EXECUTE IMMEDIATELY.

## Argument Parsing

- `--global` or `Global:` — configure in `~/.claude/.mcp.json` (default: project-level `.mcp.json`)
- `--api-key <key>` or `API-Key:` — skip the API key prompt

## Execution

### Step 1: Check if Obsidian MCP is Already Configured

Read the target `.mcp.json` file:
- If `--global`: read `~/.claude/.mcp.json`
- Else: read `.mcp.json` in the current working directory
- If the file doesn't exist, that's fine — we'll create it

If the file exists AND contains `"obsidian-mcp-server"`:
  - Inform user: "Obsidian MCP is already configured in your .mcp.json."
  - Use `AskUserQuestion`:
    | # | Header | Question | Options |
    |---|--------|----------|---------|
    | 1 | `Action` | "Obsidian MCP is already configured. What would you like to do?" | "Reconfigure (new API key)", "Test connection", "Remove Obsidian MCP", "Cancel" |
  - If "Reconfigure": continue to Step 2
  - If "Test connection": skip to Step 5
  - If "Remove": remove the `obsidian-mcp-server` entry from `.mcp.json`, save, inform user, STOP
  - If "Cancel": STOP

### Step 2: Collect Configuration

If `--api-key` was provided, use it. Otherwise ask:

```
AskUserQuestion:
  questions:
    - header: "API Key"
      question: "Paste your Obsidian Local REST API key. You can find it in Obsidian → Settings → Community Plugins → Local REST API → Copy API Key."
      options:
        - label: "I need help finding it"
          description: "I'll walk you through enabling the Local REST API plugin and getting the key."
        - label: "I don't have Obsidian yet"
          description: "I'll explain how to install Obsidian and the required plugin."
    - header: "Scope"
      question: "Where should I save the Obsidian MCP config?"
      options:
        - label: "Global — all projects (Recommended)"
          description: "Save to ~/.claude/.mcp.json. Obsidian will be available in every Claude Code session."
        - label: "This project only"
          description: "Save to .mcp.json in the current directory. Only this project gets Obsidian."
```

If user selected "I need help finding it":
  Print step-by-step instructions:
  ```
  1. Open Obsidian
  2. Go to Settings (gear icon, bottom-left)
  3. Go to Community Plugins (left sidebar)
  4. If "Restricted mode" is ON → click "Turn off restricted mode"
  5. Click "Browse" → search "Local REST API" → Install → Enable
  6. Go back to Community Plugins list → click the gear icon next to "Local REST API"
  7. Copy the API Key shown at the top
  8. Paste it here
  ```
  Then re-ask for the API key using `AskUserQuestion`.

If user selected "I don't have Obsidian yet":
  Print:
  ```
  1. Download Obsidian from https://obsidian.md
  2. Install and create a vault (or open an existing one)
  3. Then follow the steps above to enable Local REST API
  4. Run /autonexus:setup again when ready
  ```
  STOP.

### Step 3: Detect Platform

Detect the operating system to set the correct MCP command format:

```bash
# Check platform
uname -s 2>/dev/null || echo "Windows"
```

| Platform | command | args |
|----------|---------|------|
| Windows (win32, MINGW, MSYS) | `"cmd"` | `["/c", "npx", "-y", "obsidian-mcp-server"]` |
| macOS / Linux | `"npx"` | `["-y", "obsidian-mcp-server"]` |

### Step 4: Write .mcp.json

Read the existing `.mcp.json` (or start with `{"mcpServers": {}}`).

Add the `obsidian-mcp-server` entry:

```json
{
  "obsidian-mcp-server": {
    "command": "{platform_command}",
    "args": {platform_args},
    "env": {
      "OBSIDIAN_API_KEY": "{user_provided_key}",
      "OBSIDIAN_BASE_URL": "https://127.0.0.1:27124",
      "OBSIDIAN_VERIFY_SSL": "false",
      "OBSIDIAN_ENABLE_CACHE": "true"
    }
  }
}
```

Merge into existing `mcpServers` object — preserve all other MCP servers. Write the file.

**CRITICAL:** Do NOT overwrite or remove other MCP servers in the file. Read → merge → write.

### Step 5: Verify Connection (Optional)

After writing the config:

```
Obsidian MCP has been configured!

Configuration saved to: {path to .mcp.json}
API Key: {first 8 chars}...{last 4 chars}
Base URL: https://127.0.0.1:27124
Platform: {Windows/macOS/Linux}

⚠️  You MUST restart Claude Code for the new MCP server to load.

After restarting, run /autonexus:setup --test to verify the connection,
or just run any /autonexus command — it will check automatically at Phase 0.
```

If `--test` flag was passed OR user selected "Test connection" in Step 1:
  Try calling `obsidian_list_notes({ dirPath: "/", recursionDepth: 0 })`.
  - If succeeds: "Obsidian connection verified! Vault contains {N} items."
  - If fails: "Connection failed. Check that Obsidian is open and Local REST API is enabled."

### Step 6: Offer First Run

```
AskUserQuestion:
  header: "Next"
  question: "Setup complete! What would you like to do next? (Remember to restart Claude Code first)"
  options:
    - label: "Nothing — I'll restart later"
      description: "You can run any /autonexus command after restarting."
    - label: "Show me available commands"
      description: "List all 9 AutoNexus commands with descriptions."
```

If "Show me available commands":
  Print the command table from SKILL.md.

## Error Handling

| Error | Recovery |
|-------|----------|
| `.mcp.json` is malformed JSON | Show the error, ask user to fix it manually, or offer to create a fresh one |
| API key looks invalid (too short, wrong format) | Warn but proceed — the connection test will catch real issues |
| Write permission denied on `.mcp.json` | Suggest running with appropriate permissions or using `--global` |
| Obsidian MCP already configured with different key | Ask if they want to update (Step 1 handles this) |
