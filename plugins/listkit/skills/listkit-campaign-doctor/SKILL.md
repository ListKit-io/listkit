---
name: listkit-campaign-doctor
description: Diagnoses ListKit campaign performance with the ListKit MCP and prescribes the fix. Use when the user asks if their campaigns are working, why they get no replies, no positive replies, high bounce, no meetings, or "is cold email even working", and for any scheduled results review. Pulls real analytics, measures everything per leads contacted, finds the one bottleneck, then fixes it with the user's approval.
---

# Campaign Doctor

Most "no results" isn't the platform. It's not enough tests, not enough volume, not enough time, or slow replies. Find the bottleneck with data, name ONE disease, prescribe, then do the fix.

Follow **listkit-operator** rules. Read-only analysis needs no confirmation.

## 1. Pull the data (parallel, free)
- `list_campaigns` (status + totals)
- For each active or recent campaign: `get_campaign_analytics` (since launch) and `get_campaign_analytics_day_wise` (last 21 days)
- `get_campaign_sequence` for the top campaigns
- `list_mailboxes` (status, warmup, daily_sent_count vs daily_limit)
- `get_usage_allowance`
- `list_inbox_threads` on the Interested folder (folder id from `list_inbox_folders`)

## 2. Do the math (per LEADS, not emails)
For each campaign and the account total:
- Leads contacted (`total_contacted`), emails sent, days live
- Reply rate = replies / leads contacted (analytics now return a per-lead reply rate: use that field, never the per-email one)
- Positive rate = interested / leads contacted, and **positives per 1,000 leads**
- Bounce rate, and whether it's concentrated in one campaign
- Sends per day vs capacity (daily limit x READY inboxes)
- Variants per step, spintax yes/no, number of live campaigns

`interested` or `unique_leads_contacted` = null on a day means not fetched, not zero.

## 3. Sample size gate
Positives are rare (about 1 per 1,000 leads), so small samples lie. At 1,000 leads an average campaign has about a 1 in 3 chance of zero positives.
- Under ~1,000 leads contacted, or under 10 sending days: "Too early to judge. Here's what I'm watching." Check only sending, volume and setup.
- 1,000+ leads: judge reply rate (deliverability).
- 3,000+ leads in one campaign: judge its positives. Never pause or kill a campaign for zero positives before this.
- 10,000+ leads across the account: judge the account.
- "I reached out to 1,000 people and got nothing" is the most common too-early complaint. Give the dates they'll hit 3,000 and 10,000, and launch a second angle now instead of killing the first.

## 4. Diagnosis tree (stop at the first failure)

| # | Check | Symptom | Disease | Prescription |
|---|---|---|---|---|
| 1 | Sending at all? | 0 sends 48h+, or far under capacity | Infra or setup | Hand off to **listkit-sending-health**. Check campaigns paused, no leads left, inboxes disconnected or stuck. |
| 2 | Enough volume? | Sending under ~50% of capacity, 1 small campaign | Starved machine | Hand off to **listkit-volume-engine**. More lists, more campaigns. |
| 3 | Reply rate | Under 1% after 1,000+ leads | Deliverability | Check variants/spintax (1 variant, no spintax = sender bounce). Rebuild with 4/2/2 variants + spintax via the copywriter, lower per-inbox daily limit if it was raised above 25, duplicate campaign with re-verified leads. If bounce is high on their own CSV, verify it. |
| 4 | Positive rate | Replies 1%+ but under ~0.5 positives per 1,000 leads (after 3,000+ leads) | Offer or targeting | Read the replies: mostly "not interested" or unsubscribes = offer; "wrong person" = targeting. New angle, easier yes (free deliverable), tighter niche list. New campaign, not just a tweak. |
| 5 | Booked rate | Positives exist but under 40% booked | Follow-up | Hand off to **listkit-reply-desk**. Speed to lead (under 5 min), two specific times, follow up every 1 to 2 days, multichannel. |
| 6 | All healthy | Benchmarks met | Scale | Duplicate the winner, new lists, new ICPs. See **listkit-volume-engine**. |

Bounce: if replies are healthy, don't chase bounce. If bounce is high AND replies are low, it's #3.

### 4b. High bounce on a first batch ("triple verified but bouncing")
Usually sender-side, not the list. Check these four first, in order:
1. One variant, or no spintax: the same text from new inboxes gets blocked. Fix with 4/2/2 variants + spintax (copywriter).
2. Inboxes still ramping, or daily limits raised above 25.
3. Same exact copy across several campaigns on the same inboxes.
4. Their own uploaded CSV that wasn't verified: verify it.
Separate sender-side bounce (copy, variants, ramp) from recipient bounce (bad addresses). For bad ListKit rows, offer to send ListKit feedback (`send_feedback`): tell the user what it contains (filters and share of bad rows) and send it only if they agree, never with company names or contact data. Don't argue about verification.
Don't blame the list or the user, and don't comment on data quality. If bounce stays high after the fixes, offer a lower per-inbox limit (15 to 20) for a week, then send the account to ListKit support with the campaign names. Never delete or restart inboxes to fix bounce.

Opens: ListKit doesn't track opens or clicks (tracking pixels hurt deliverability). Judge on replies and positive replies.

## 5. Output format

```
Verdict: [one line, e.g. "Delivery is fine, the offer isn't landing."]

Last 21 days: 4,210 leads contacted · 1.6% reply · 2 positive (0.5 per 1,000, benchmark ~1) · 3.1% bounce

Why: [2 to 3 bullets with the evidence, quoting 1 to 2 real replies if useful]

Fix (I can do these now):
1. ...
2. ...
3. ...
```

Then ask: "Want me to do 1 and 2?" and execute with the normal confirmations.

"Do I stop the campaign or what do you suggest?" is this skill's entry point. Answer with a direct recommendation and numbers, never a menu.

## 6. Common fixes you can execute
- Rewrite with 4 variants + spintax: **listkit-cold-email-copywriter**, then `save_campaign_sequence` (read first, it replaces everything).
- New angle as a new campaign: `duplicate_campaign`, new sequence, fresh list.
- Fresh list for a winner: **listkit-list-builder**, then import.
- Kill a loser (3,000+ leads, confirmed): pause via `update_campaign` / `set_campaign_status` PAUSED (confirm). PAUSED, not STOPPED: STOPPED can't be undone.
- Reply backlog: **listkit-reply-desk**.

## 7. Tone
Honest and specific. If the data says the offer is the problem, say it. If it's too early, say that and give the date to check again. Never promise the fix will produce X meetings. Benchmark: about 1 positive reply per 1,000 leads contacted, 40 to 60% booked.

If the user is frustrated or talking about cancelling, switch to **listkit-account-help**.
