---
name: listkit-volume-engine
description: Keeps a ListKit account sending at full capacity week after week using the ListKit MCP, refilling lists, duplicating winners, launching new angles and catching stalls before they cost a month. Use for weekly check-ins, "keep my campaigns running", "scale this", "what should I launch next", when campaigns are running out of leads, or when sending is below capacity. Designed to run as a recurring weekly routine.
---

# Volume Engine

ListKit customers who keep sending stay and win. Customers who send 10,000+ emails churn far less. The basics compound: steady volume, a new test every few days, winners duplicated onto fresh lists. Don't let the machine starve.

Follow **listkit-operator** rules.

## Weekly routine (run in this order)

### 1. Capacity check (free)
- `get_usage_allowance`: sends used vs limit this period, contacts stored vs limit
- `list_mailboxes` (`limit: 100`, every page): READY inboxes x daily_limit = daily capacity; sum of `daily_sent_count` = today's actual
- `list_campaigns`: active campaigns and their totals
- For each active campaign, `get_campaign_analytics_day_wise` last 7 days

Inbox coverage: every active campaign should have every sending-ready inbox attached. If an inbox became READY (finished warmup or a new Bundle) and isn't on the active campaigns, propose attaching it to all of them and attach on the user's yes. A scheduled run only lists it for approval.

Flag immediately:
- Any day with 0 sends in the last 48h on a weekday → hand off to **listkit-sending-health**
- Sending under 70% of capacity → not enough leads loaded or too few campaigns
- Contacts stored near the limit (only Active campaigns count) → pause Active campaigns that have no leads left to contact (confirm). Never use STOPPED for this.

### 2. Lead runway
A 3-step sequence uses ~3 sends per lead. New leads needed per weekday ≈ daily capacity / 3 (about 330/day per Bundle, ~1,650/week). Keep at least 2 weeks loaded: READY inboxes x 25 x 10 weekdays / 3 (about 3,300 leads per Bundle). Sends are rarely the limit, leads are: most accounts use about a third of their capacity because they don't load enough leads.
- Estimate leads remaining per campaign (`list_leads` with campaign_id, or import stats).
- Refill from every saved route for the ICP (list-builder §3c), not just the first list. When a route stops returning net-new people, move to the next one; when all routes are used up, propose the next adjacent ICP before the runway hits zero.
- If total unsent leads < 2 weeks of runway, refill now. Don't wait for a campaign to finish: duplicate the winner (`duplicate_campaign`) and load a fresh list into the copy.

### 3. Score the campaigns
Reply rate from 1,000+ leads contacted, positives from 3,000+ (smaller samples lie).
- Pick winners on positives and booked calls, never on reply rate alone (3% reply with 2 positives loses to 1.5% reply with 10).
- Winner (≥ ~1 positive per 1,000): duplicate onto a new list in the same ICP, or the same angle to an adjacent ICP.
- Middle: new variant or angle (see **listkit-cold-email-copywriter**).
- Loser after 3,000+ leads with a good reply rate: pause it (PAUSED, never STOPPED), the offer/ICP didn't land. If reply rate is the problem, run **listkit-campaign-doctor**.
- Under 3,000 leads: keep it running and add a second angle alongside.

### 4. Launch this week's tests
Target rhythm (same for every stage, new or long-time customer):
- Always 2 to 4 campaigns of 2,000+ leads live, different ICPs or angles
- In a customer's first 21 days of sending, the goal is positives: full volume fast, every positive answered within minutes
- At least one new campaign or angle per week per Bundle. Agencies running many clients: one new campaign a day across the book.
- Brand new account with nothing live? Stop and run **listkit-launch-first-campaign** first.

For each launch: list (**listkit-list-builder**) → copy (**listkit-cold-email-copywriter**) → create, sequence, import, senders, weekday schedule → confirm start.

### 5. Hygiene
- Re-use a TAM slowly: recycle a list only after 2 to 4 months, remove anyone who replied, re-verify (`verify_email_list` for ≤5,000 or a file), new angle.
- Pull leads from several sources? Dedupe them so nobody gets hit twice.
- Scale with new domains/Bundles, not by pushing inboxes above 25/day.
- Keep everything Mon to Fri.

### 6. Report (short)
```
This week: 4,830 sent (87% of capacity) · 1,610 new leads · 3 positives
Runway: 9 days of leads loaded ⚠️
Waiting for your yes: duplicate "Dental - Ads" onto 2,000 fresh leads (~2,000 credits), pause "SaaS - SEO" (0 positives / 3,400 leads)
Next: launch "Orthodontists - Ads" Mon, refill by Wed
```

## Always-on: run it as a scheduled task
Bundles don't include ListKit Workflows. The always-on lead feed and weekly routine run as scheduled tasks in the user's AI tool (Claude scheduled tasks or Cowork tasks, ChatGPT scheduled tasks, or whatever their tool offers).

Set it up with the user once:
1. **Weekly run (Monday morning):** the full routine above.
2. **Mid-week check (Thursday):** steps 1 and 2 only (sending stalls, lead runway).
3. **Spending guardrails, agreed up front and written into the task:**
   - The task never spends credits or changes campaigns on its own. It prepares each refill as a proposal (saved filter, quantity, cost from `estimate_order`, target campaign); the order and import run only after the user's yes in a conversation and the server's confirmation.
   - A weekly credit budget for proposed refills (e.g. "up to 3,000 credits/week"). It caps what gets proposed; it is not a pre-approval.
   - Which saved filters it may refill from (`list_saved_filters`). It never invents a new ICP on its own.
   - New campaigns, new copy, pausing or deleting anything: the task drafts and reports, the user approves next time they're in the chat.
4. **Report** at the end of every run in the format above, with anything waiting for approval at the top.

Write the task prompt so it stands alone: account name, saved filter names, campaigns it may propose refills for, the weekly budget, and "load listkit-operator and listkit-volume-engine first. Read and propose only; never spend credits or change campaigns without my yes".
