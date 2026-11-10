---
name: listkit-check-install
description: Checks that the ListKit plugin for Claude works after the install, the connector, the ListKit sign-in and the skills. Use when the user says "check my ListKit install", "is ListKit connected", "test the ListKit connection", "did the ListKit plugin install", "is ListKit working", or asks if the ListKit setup or sign-in worked.
---

# Check the ListKit install

The user installed the ListKit plugin and wants to know if it works. Answer with one ListKit call and one message.

This check replaces the usual session start. Do not run the **listkit-operator** session start for it, and do not greet the user as `get_me`'s description says.

## 1. Call `get_me` one time

- Call the ListKit tool `get_me` one time, with no arguments. If your client loads tools on demand, search for it first.
- Make no other call. No `get_account_overview`, `get_account_snapshot`, `list_campaigns`, `authenticate`, terminal command or file read. The check spends nothing and changes nothing.

## 2. Find the result

| What you see | Result |
|---|---|
| `get_me` answers with an `email` | Install works (§3) |
| ListKit has only an `authenticate` tool, or `get_me` answers that sign-in is needed (401, unauthorized) | Not signed in (§4) |
| There is no ListKit tool | Connector not connected (§4) |
| `get_me` fails in a different way | ListKit did not answer. Tell the user to try again in a minute. If it fails again, they write to support@listkit.io. |

This skill runs, so the plugin is loaded. "ListKit skills loaded" is always checked.

Pick the client from what your system prompt says you are. Tools do not tell you the client: Cowork and Claude chat with code execution also have a terminal and file tools.

| Your system prompt says | Client |
|---|---|
| You are Claude Code | Claude Code |
| Cowork, claude.ai, the Claude app or Claude Desktop | Claude |
| Neither | Both: give the text for each client, under the labels **In Claude (chat, Desktop or Cowork):** and **In Claude Code:** |

## 3. Install works

Send this, with the `email` from `get_me` as the account (`get_me` has no other name):

```
Your ListKit install works. Signed in as <email>.

- [x] Connector connected
- [x] Signed in to ListKit
- [x] ListKit skills loaded
```

Then give the permission advice for the client (for Both, give the two texts under their labels):

- **Claude:** "So Claude does not ask before each ListKit read: open the ListKit connector. Under **Tool permissions**, set **Read-only tools** to **Always allow**. Keep **Write/delete tools** on **Needs approval**, so Claude still asks before orders, sends and deletes."
- **Claude Code:** "Claude Code asks before each ListKit tool until you add the ListKit allow rule. The Claude Code section of https://developer.listkit.io has a prompt that adds it. Paste that prompt here, and I add the rule after your yes."

You cannot read the permission settings. Never say that you checked them, or that they are set or not set. Give the advice as a step "if you did not do it yet".

If `first_connection` is true, add one line with `walkthrough_video_url` and `strategy_call_url`. End with one question: what they want to work on first.

## 4. A step failed

Show the same list. Mark the failed step `- [ ]`. A step after it that you could not check gets "(not checked)". Then give the fix for the client (for Both, give the two fixes under their labels) and stop.

| Failed step | Claude | Claude Code |
|---|---|---|
| Connector connected | Open **Plugins**, then **ListKit**. On its **Connectors** tab, connect ListKit. Start a new chat. | Run `/mcp`. If `plugin:listkit:listkit` is not in the list or shows failed, restart Claude Code and run `/mcp` again. |
| Signed in to ListKit | Open the ListKit connector and click **Connect**. Sign in with your ListKit account. Start a new chat. | Run `/mcp`, choose `plugin:listkit:listkit` and select **Authenticate**. Sign in with your ListKit account. |

End with: "Then paste `Check my ListKit install` again."
