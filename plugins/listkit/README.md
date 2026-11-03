# ListKit

Run your cold email on ListKit from a conversation with Claude.

## Use it

Connect your ListKit account from the plugin's Connectors tab, then tell Claude what you sell and who you sell to. Claude reads your website and account, builds lead lists that cover your whole market, writes your campaign with ListKit's AI Script Writer, launches it on all your inboxes, drafts replies for you to approve, and runs a weekly check so your campaigns never run out of leads. Claude asks for your yes before anything that spends credits, sends email or can't be undone.

Try: "Launch my first campaign", "Build me a list of 5,000 dental practice owners", "Is my campaign working?", "Check my inbox".

Requires a ListKit Bundle.

## Install

- **Claude (web, desktop, Cowork):** add ListKit from the Claude directory, then connect your ListKit account when asked.
- **Claude Code:** `/plugin install listkit --marketplace ListKit-io/listkit-agent-kit`
- **Any other AI tool:** add the MCP server `https://developer.listkit.io/mcp` and sign in with your ListKit account.

## Data

The plugin sends your requests to your ListKit account through ListKit's MCP server (search filters, campaign content, inbox replies and account actions you approve). ListKit processes that data under its terms and privacy policy at listkit.io. The plugin itself stores nothing and runs no code on your machine: it is skills (instructions for Claude) plus one connection to ListKit's MCP server. Privacy policy: [listkit.io/privacy](https://listkit.io/privacy). Support: support@listkit.io
