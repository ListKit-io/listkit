---
name: listkit-results-report
description: Produces a clear weekly or monthly cold email results report from ListKit data via the ListKit MCP, for the user themselves or for an agency's client. Use when the user asks for a report, weekly update, client update, recap, how campaigns did this week or month, or numbers to share with a client, partner or boss.
---

# Results Report

Agencies keep clients by communicating progress before the client has to ask. This report does that in 2 minutes: real numbers, measured the right way, with what's next.

## 1. Scope
Ask only if unclear: period (default last 7 days), which campaigns (default all active; for agencies, the client's campaigns by name match), audience (owner or client).

## 2. Pull (free)
- `list_campaigns`
- `get_campaign_analytics` for the period, per campaign
- `get_campaign_analytics_day_wise` for trend
- Interested threads: `list_inbox_threads` filtered to the Interested category, for names/companies of positive replies (no email addresses in a client-facing report unless asked)
- `list_mailboxes` for capacity context

## 3. Metrics (always per leads contacted)
- Leads contacted, emails sent
- Reply rate, positive replies, positives per 1,000 leads (benchmark ~1)
- Booked calls: count leads in the Meeting Booked category if the account uses it, otherwise ask the user
- Bounce rate (mention only if high or asked)
- ListKit doesn't track opens or clicks; judge campaigns on replies and positive replies. If asked about opens, say ListKit doesn't track them because tracking pixels hurt deliverability.

## 4. Client-facing format (three-part update)

```
[Client] outbound update · Oct 8 to Oct 14

What we worked on
- Launched "Dental - Implant ads" to 2,100 new practice owners
- Tested 4 subject/intro variants; variant C is leading

Results
- 2,640 leads contacted · 1.9% reply rate
- 4 interested replies (Bright Smiles Ohio, ...) · 2 calls booked
- On pace with the benchmark of ~1 interested reply per 1,000 leads

What's next
- Duplicate the winning variant onto orthodontists (Mon)
- Follow up the 2 open interested replies today

What we need from you
- Availability for 2 calls next week
```

Rules: lead with progress, even in a slow week (activity, tests launched, learnings). Acknowledge a problem before the client finds it, with the fix already in motion. No guarantees about next week.

## 5. Owner-facing format
Shorter, more blunt: totals, positives per 1,000 vs benchmark, best and worst campaign, capacity used, the ONE thing to fix (hand off to **listkit-campaign-doctor** if a benchmark is missed), the ONE thing to launch next.

## 6. Delivery
Offer it as a doc or email draft in their tools if connected (Docs, Gmail draft, Slack), never sent without approval. Offer to make it a recurring weekly scheduled task.
