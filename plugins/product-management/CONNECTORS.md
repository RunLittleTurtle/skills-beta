# Connectors (Claude Code adaptation)

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool fills that category in your setup. For example, `~~project tracker` might mean Linear, Asana, Jira, monday.com, ClickUp, or any other tracker.

The skills are **tool-agnostic** — they describe workflows in terms of categories (project tracker, design, product analytics, etc.) rather than specific products. In Claude Code, the resolution is:

1. **MCP server connected** for that category → use it
2. **Local files in CWD** that look like content for that category (e.g. `.md` files matching "PRD", "research notes", "roadmap") → use them
3. **Otherwise** → ask the user to paste or describe the content

The skills will not insist on MCP servers and will not refuse to work without them.

## Claude Code specifics

Compared to the Cowork source plugin:

- **No `.mcp.json` shipped with this plugin.** Configure MCP servers you actually use in your global Claude Code config (`~/.claude/settings.json` or `~/.claude/mcp.json`). The plugin will pick them up.
- **Local markdown is a first-class input.** PRDs in `~/Notes/PRDs/`, roadmap drafts in a project folder, interview transcripts pasted in chat — all valid sources.
- **Obsidian vault access is opt-in.** The plugin will not scan an Obsidian vault unless you explicitly point it at a file or folder (privacy rule).

## Connector categories used by this plugin

| Category | Placeholder | Typical servers (if you connect them) | Local fallback |
|----------|-------------|--------------------------------------|----------------|
| Calendar | `~~calendar` | Google Calendar, Microsoft 365 | Paste your schedule |
| Chat | `~~chat` | Slack, Microsoft Teams | Paste threads or describe |
| Competitive intelligence | `~~competitive intelligence` | Similarweb, Crayon, Klue | Web search + paste |
| Design | `~~design` | Figma | Paste screenshots, link to file |
| Email | `~~email` | Gmail, Microsoft 365 | Paste threads |
| Knowledge base | `~~knowledge base` | Notion, Confluence, Coda, Guru | Local markdown files (Obsidian, project folder) |
| Meeting transcription | `~~meeting transcription` | Fireflies, Gong, Dovetail, Otter.ai | Paste transcripts |
| Product analytics | `~~product analytics` | Amplitude, Pendo, Mixpanel, Heap, FullStory | Paste numbers, screenshots of dashboards |
| Project tracker | `~~project tracker` | Linear, Asana, monday.com, ClickUp, Atlassian (Jira/Confluence), Shortcut | Markdown task lists, paste tickets |
| User feedback | `~~user feedback` | Intercom, Productboard, Canny, UserVoice | Paste tickets, screenshots |

## Adding an MCP server

In Claude Code, edit `~/.claude/mcp.json` (or `~/.claude/settings.json` `mcpServers` section). Example for Linear:

```json
{
  "mcpServers": {
    "linear": {
      "type": "http",
      "url": "https://mcp.linear.app/mcp"
    }
  }
}
```

Then `/reload-plugins` or restart Claude Code. The skills will detect the server automatically the next time they look for `~~project tracker`.
