# Instawork Agent Skills

A collection of agent skills for AI coding assistants (Cursor, Claude Code, etc.) provided by [Instawork](https://www.instawork.com) — the leading platform for flexible staffing.

## Skills

### find-workers

Guides finding workers and booking shifts on the Instawork platform for business partners via the Instawork Partner MCP server.

Use this skill to:

- Book temporary workers for shifts
- Find pricing for positions and locations
- Search available locations and positions
- Rebook from previous shifts or templates

> [!NOTE]
> Only USA and Canada are supported. Long-term staffing and permanent positions are not supported.

## Getting Started

### Prerequisites

- An [Instawork](https://www.instawork.com) business partner account
- An AI assistant that supports MCP (e.g. [Cursor](https://cursor.com), [Claude Desktop](https://claude.ai/download), or [Claude Code](https://claude.ai/code))

### MCP Server Setup

Add the Instawork Partner MCP server to your AI assistant configuration:

**Cursor / Claude Desktop** (`mcp.json` or `claude_desktop_config.json`):

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

**Claude Code:**

```bash
claude mcp add instawork-partner -- npx mcp-remote https://finch.instawork.com/mcp/partner/
```

Authentication is handled via OAuth. You will be prompted to log in with your Instawork partner credentials on first use.

## Usage

Once set up, you can ask your AI assistant natural language questions like:

- _"Book 3 warehouse workers for Friday at our Chicago location"_
- _"What's the pricing for a line cook shift next week?"_
- _"Rebook the same team from our last event"_

The skill will guide you through the booking workflow, confirm details, and submit the booking on your behalf.
