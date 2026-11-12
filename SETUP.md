# Local setup (Will's machine)

How this fork is installed and wired into Claude Code, OpenWork, OpenCode Desktop and Unsloth Studio.

## Build

```powershell
cd ~/Developer/projects/firefox-devtools-mcp
npm ci
npm run build    # writes dist/index.js
```

## Update from Mozilla

```powershell
git checkout main
git pull upstream main
git push origin main
npm ci
npm run build
```

Restart Claude Code sessions, OpenWork, OpenCode Desktop and Unsloth Studio afterwards so they pick up the new build.

## Firefox profiles

Each app gets its own throwaway profile, never the everyday one:

- Claude Code: `~/.firefox-devtools-mcp/claude-code`
- OpenWork and OpenCode Desktop: `~/.firefox-devtools-mcp/openwork` (they share one config, so one profile)
- Unsloth Studio: `~/.firefox-devtools-mcp/unsloth`

Two apps can't share one profile at the same time, because Firefox locks it. So OpenWork and OpenCode Desktop can't drive Firefox at the same time.

## Claude Code

User scope, in `~/.claude.json`:

```json
"mcpServers": {
  "firefox-devtools": {
    "type": "stdio",
    "command": "node",
    "args": [
      "C:/Users/Will/Developer/projects/firefox-devtools-mcp/dist/index.js",
      "--profile-path", "C:/Users/Will/.firefox-devtools-mcp/claude-code",
      "--viewport", "1280x720"
    ],
    "env": {}
  }
}
```

## OpenWork and OpenCode Desktop

Global OpenCode config, in `~/.config/opencode/opencode.jsonc`, so every OpenWork workspace and OpenCode Desktop get it. A workspace-folder `opencode.jsonc` is not reliable: OpenWork's engine kept loading the old `Documents/OpenWork` folder after the workspace moved.

```jsonc
"mcp": {
  "firefox-devtools": {
    "type": "local",
    "command": [
      "node",
      "C:/Users/Will/Developer/projects/firefox-devtools-mcp/dist/index.js",
      "--profile-path", "C:/Users/Will/.firefox-devtools-mcp/openwork",
      "--viewport", "1280x720"
    ],
    "enabled": true
  }
}
```

Check that OpenWork's engine sees it:

```powershell
& "$env:LOCALAPPDATA\Programs\@openworkdesktop\resources\sidecars\opencode-x86_64-pc-windows-msvc.exe" mcp list
```

Same check for OpenCode Desktop (replace the version folder with the one that exists):

```powershell
& "$env:APPDATA\ai.opencode.desktop\cli\2.2.5-x64\opencode-cli.exe" mcp list
```

Both should print `firefox-devtools  connected`.

## Unsloth Studio

Unsloth Studio keeps MCP servers in its own database (`~/.unsloth/studio/studio.db`), not in a config file. Add the server in the app: open a chat, open the MCP servers dialog, choose the import option and paste:

```json
{
  "mcpServers": {
    "firefox-devtools": {
      "command": "node",
      "args": [
        "C:/Users/Will/Developer/projects/firefox-devtools-mcp/dist/index.js",
        "--profile-path", "C:/Users/Will/.firefox-devtools-mcp/unsloth",
        "--viewport", "1280x720"
      ]
    }
  }
}
```

Then enable it in the chat's MCP menu. It needs a model that supports tool calling.

- Local-command servers are allowed automatically while Studio listens on 127.0.0.1 only.
- They are switched off by Remote Access, or when tools are disabled. The dialog says "Local commands are disabled" in that case.
- If Studio can't find `node`, set `"command"` to `C:/Program Files/nodejs/node.exe`.

## Smoke test without an app

```powershell
npx -y @modelcontextprotocol/inspector --cli node dist/index.js --headless --start-url https://example.com --method tools/call --tool-name get_page_text
```

It should print the example.com text.
