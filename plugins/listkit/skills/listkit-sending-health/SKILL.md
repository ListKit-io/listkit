---
name: listkit-sending-health
description: Checks and explains ListKit sending infrastructure through the ListKit MCP, domains, inboxes, warmup, ramp-up, daily limits and provisioning, and troubleshoots "why isn't my campaign sending". Use when the user asks about inboxes, domains, warmup, when they can start sending, why sends are low or zero, disconnected inboxes, pre-warmed vs branded, or adding sending capacity.
---

# Sending Health

Most "it's not sending" is normal pacing (warmup, ramp-up, schedule, follow-up delays). Some is a real problem (disconnected inboxes, campaign paused, no leads). Tell the two apart fast, and give dates, not vibes.

Follow **listkit-operator** rules. Billing actions are never done for the user.

## 1. Health snapshot (free, parallel)
- `get_account_health`: one read for the inboxes (total, active, disconnected), the inboxes still ramping with the days left in each ramp, the daily capacity and today's sends, the setup status of the newest domain order, and the active campaigns that sent nothing yesterday or today. It checks only the first 5 active campaigns: `campaigns_checked` or `campaigns_first_page_only` in the answer says that it did not check all of them. Then check the others with `get_campaigns_report` (yesterday and today). It reads every mailbox page up to 500 inboxes, and says `inboxes_first_pages_only: true` past that.
- `get_domains(action=prewarmed_claims)` if anything is setting up. For the newest domain order, use `domains.latest_order` from `get_account_overview` (below). Read `get_domains(action=orders)` only when `latest_order.status` is `failed`, or when the overview has `errors.domain_orders`. `no_active_domain_order: true` means no domain order is in progress, not that domains are ready.
- `get_account_overview` for the ready inbox count (`inboxes.ready`: warmed up and connected, the server's own rule) and the campaign status (Draft, Active, Paused, Stopped, Completed; the first 25 campaigns, `campaigns_first_page_only: true` past that). The health check counts only the active ones.
- Per-inbox rows (`status`, `warmup_status` NOT_STARTED / WARMING / READY, `daily_limit`, `daily_sent_count`) only when a step below needs them: `list_mailboxes` (`limit: 100`, follow `next_cursor` to the end, then check the count against the health check's inbox total). Renewing-soon domains: `get_domains(action=stats)`.

Show it as:
```
Domains: 20 active · Inboxes: 38 ready, 2 disconnected ⚠️
Today: 612 / 950 sent · Ramp: 6 inboxes still ramping (full by Tue Oct 20)
Campaigns: 2 active, 1 paused
```
Ready inboxes = `get_account_overview` `inboxes.ready`. Do not compute it from the health check: its ramping list also has disconnected inboxes, so the groups overlap. If `inboxes_first_pages_only` is true, read the rest with `list_mailboxes` (`limit: 100`, follow `next_cursor`). Do not tell the customer about the tool limit.

## 2. The timeline facts
- Branded: usually set up within 12 hours (tell customers up to 72h), then 14 days warmup. Not ready after 24h with no progress = stuck: hand off with the domain names, don't keep waiting.
- Pre-warmed (.info): no warmup, usually ready in minutes. Stock can run out.
- Every inbox: ramps from 1 to 25 emails/day over 7 sending days. Daily limit is locked during ramp.
- Full capacity: 25/day x inboxes (about 1,000/day per Bundle).
- "0/1" on an inbox during warmup or day 1 of ramp is normal.
- Changing a sender name restarts that inbox's 14-day warmup. Changing the redirect URL doesn't affect warmup.

Always convert to dates for the user.

## 3. "Why isn't it sending?" checklist (in order)
1. Weekend or outside the campaign's schedule window? Normal.
2. Inboxes still warming or ramping? Normal. Give the date. "Stuck at 1/day" in the first sending days is the ramp. On multi-Bundle accounts, count only sending-ready inboxes.
3. Campaign status not Active? If it paused after a failed payment, check `get_billing(action=summary)` `dunning` first. If it isn't `none`, resuming won't hold: the payment needs updating in Billing, then hand off. Once dunning is `none`, resume it (confirm first). A STOPPED campaign can't resume: duplicate it.
4. Not all inboxes attached? Every campaign should have every sending-ready inbox on the account (read every page of `list_mailboxes`; a 2-Bundle account with only 40 of 80 attached is a common miss). Attach any missing READY inboxes (confirm first). This also covers inboxes that finished warmup or ramp after the campaign was built.
5. No leads left, or the import added 0? Check `get_leads(action=import_status)` / `get_leads(action=leads)`. Refill.
6. Follow-ups waiting on their delay? Normal pacing.
7. Several campaigns running on the same inboxes? Normal and expected: every campaign uses every inbox, and each inbox's daily limit is shared across them.
8. Stuck at a low limit after ramp ended? Try `update_mailboxes(action=daily_limit)` once per inbox (confirm), then re-read the limit. If it didn't change, hand off. Don't call `update_mailboxes(action=restart_ramp)` for a stuck ramp.
9. Inboxes disconnected, stuck in setup, or "provisioning failed"? Real problem, and only a ListKit person can fix it (same for redirect and sender name changes). Don't delete or restart things. Use the handoff in **listkit-account-help** with the inbox emails and what they see.

## 4. Settings changes (confirm each)
- `update_mailboxes(action=daily_limit)`: only after ramp. During ramp the call succeeds but the inbox keeps its ramp limit, so never tell the user it changed: give the ramp end date instead. Don't go above 25/inbox. For the best deliverability some senders run 15 to 20.
- `update_mailboxes(action=warmup)`: keep warmup on for ListKit inboxes. Don't turn it off to "go faster".
- `update_mailboxes(action=signature)`: fine anytime, under 350 characters.
- `update_mailboxes(action=restart_ramp)`: throws away ramp progress and drops the inbox to 1/day. Only for a mailbox that needs a deliverability fresh start, and say that clearly.
- `delete_mailbox`: stops all sending from it, no pause. Avoid. ListKit-provisioned inboxes can't be deleted via API.

## 5. Adding capacity
- More volume = more Bundles (new domains), not more sends per inbox.
- Pre-warmed: `get_domains(action=prewarmed_availability)` → `get_domains(action=prewarmed_pick)` → personas → confirm → `claim_prewarmed_domains`.
- Branded: `get_domains(action=search_new)` → `draft_domain_order` (each domain needs exactly 2 senders with different usernames, Google provider) → confirm → `confirm_domain_order` → poll `get_domains(action=provisioning)`. It's usually live within 12 hours; tell the user up to 72h, then 14 days warmup. No progress after 24h = stuck: hand off.
- These use the Bundle's domain allowance. If they need another Bundle, tell them to add it in Billing or talk to the ListKit team. Don't start a checkout.

## 6. Their own inboxes
They can connect existing Google/Outlook (Microsoft 365) or SMTP inboxes in the app (Inboxes page). Never ask for a mailbox password in chat. Send them to https://app.listkit.io/email-accounts.

## 7. What not to say
Don't mention warm-up scores, don't promise inbox placement, don't promise a fix date for a bug. Reply rate is the real deliverability metric: 1%+ means mail is landing.
