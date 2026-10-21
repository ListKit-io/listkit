# ListKit agent kit

ListKit for AI assistants: the ListKit MCP connector and the cold email skills.

- **Any MCP client** (Claude, ChatGPT and others that support remote MCP): connect `https://developer.listkit.io/mcp`.
- **Claude**: install the `listkit` plugin from this repo, which bundles the connector and the skills.

```
/plugin marketplace add ListKit-io/listkit-agent-kit
/plugin install listkit@listkit
```

Updates ship when `main` changes.

## Layout

```
.claude-plugin/marketplace.json   Claude plugin marketplace: lists the plugins below.
plugins/listkit/                  The listkit plugin: .mcp.json (the connector) and skills/<name>/SKILL.md.
```

The skills are plain Markdown, so other assistants can use them as instructions too.

## Changing a skill

Skill text is instructions an agent follows. Every change goes through a PR and a review for prompt injection
before it merges, and it must match the tools the live MCP server has. Spending credits, sending email, deleting
leads and adding exclusions always wait for the user's yes.
