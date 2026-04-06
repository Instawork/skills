# Instawork Partner MCP — Connection Setup

## Cursor / Claude Desktop (`mcp.json` or `claude_desktop_config.json`)

```json
{
  "mcpServers": {
    "instawork-partner": {
      "command": "npx",
      "args": ["mcp-remote", "https://finch.instawork.com/mcp/partner/"]
    }
  }
}
```

## Claude Code

```bash
claude mcp add instawork-partner -- npx mcp-remote https://finch.instawork.com/mcp/partner/
```

Authentication is handled via OAuth. You will be prompted to log in with your Instawork partner credentials and provide consent on first use.
