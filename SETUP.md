# Local setup (Will's machine)

How this fork is installed and wired into Claude Code and OpenWork.

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

Restart Claude Code sessions and OpenWork afterwards so they pick up the new build.

## Firefox profiles

Each app gets its own throwaway profile, never the everyday one:

- Claude Code: `~/.firefox-devtools-mcp/claude-code`
- OpenWork: `~/.firefox-devtools-mcp/openwork`

Two apps can't share one profile at the same time, because Firefox locks it.

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

## OpenWork

Global OpenCode config, in `~/.config/opencode/opencode.jsonc`, so every OpenWork workspace gets it. A workspace-folder `opencode.jsonc` is not reliable: OpenWork's engine kept loading the old `Documents/OpenWork` folder after the workspace moved.

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

## Smoke test without an app

```powershell
npx -y @modelcontextprotocol/inspector --cli node dist/index.js --headless --start-url https://example.com --method tools/call --tool-name get_page_text
```

It should print the example.com text.
