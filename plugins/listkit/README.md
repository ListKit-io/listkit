# ListKit

Run your cold email on ListKit from a conversation with Claude.

## Use it

Connect your ListKit account from the plugin's Connectors tab, then tell Claude what you sell and who you sell to. Claude reads your website and account, builds lead lists that cover your whole market, writes your campaign with ListKit's AI Script Writer, launches it on all your inboxes, drafts replies for you to approve, and runs a weekly check so your campaigns never run out of leads. Claude asks for your yes before anything that spends credits, sends email or can't be undone.

Try: "Launch my first campaign", "Build me a list of 5,000 dental practice owners", "Is my campaign working?", "Check my inbox".

Requires a ListKit Bundle.

## Install

- **Claude (web, desktop, Cowork):** open **Customize**, then **Plugins**. Click **Add**, then **Add marketplace**. In **URL**, enter `ListKit-io/ListKit` and click **Sync**. Click **Add** next to ListKit, then connect your ListKit account from the plugin's **Connectors** tab. Plugins need a paid Claude plan.
- **Claude Code:** run `claude plugin install listkit --marketplace ListKit-io/ListKit` in your terminal (Claude Code 2.1.292 or later). Then run `/mcp`, choose `plugin:listkit:listkit` and **Authenticate**.
- **Any other AI tool:** add the MCP server `https://developer.listkit.io/mcp` and sign in with your ListKit account.

In Claude Code, you can also add the ListKit allow rule: on developer.listkit.io, open Claude Code, copy the setup prompt that adds the allow rule and paste it into Claude Code. The prompt offers to add the rule to `~/.claude/settings.json`. It asks you before it changes the file. A plugin cannot add permission rules itself, so the prompt does it. With the rule, Claude Code runs the ListKit tools that only read, and the free writes, without asking you each time. A free write does not spend credits, send email or delete. The rule also lets Claude Code send feedback or a help request to the ListKit team without asking. Tools that spend credits, send email or delete still ask each time. You can remove the entries at any time with `/permissions`.

## Data

The plugin sends your requests to your ListKit account through ListKit's MCP server (search filters, campaign content, inbox replies and account actions you approve). ListKit processes that data under its terms and privacy policy at listkit.io. The plugin itself stores nothing and runs no code on your machine: it is skills (instructions for Claude) plus one connection to ListKit's MCP server. Privacy policy: [listkit.io/privacy](https://listkit.io/privacy). Support: support@listkit.io
