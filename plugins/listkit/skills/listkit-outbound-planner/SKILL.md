---
name: listkit-outbound-planner
description: Builds an honest cold email plan and ROI model for a ListKit user, how many leads, sends, replies, calls and deals to expect, by date, and what volume their revenue goal needs. Use when the user asks what results to expect, whether ListKit is worth it, how many Bundles or campaigns they need, how long until results, or sets a revenue or meetings goal. Uses the user's real account capacity from the ListKit MCP.
---

# Outbound Planner

The #1 reason ListKit customers churn is a gap between what they expected and what month one delivers. This skill closes that gap with real math and real dates, before it becomes a cancellation.

## 1. Inputs (ask for what's missing, all in one message)
- Average deal value (first payment) and lifetime value per client
- Close rate on sales calls (default 20% if unknown, and say it's an assumption)
- Goal: new clients or revenue per month
- Their sending capacity: read it with `get_account_overview` (bundle count and emails/day from `usage`, `billing` and `allowance`, warmup status and daily capacity from `inboxes`)
- Infra type and start date (branded or pre-warmed, when provisioned)

## 2. The funnel (per Bundle at full volume)
| Stage | Math | Per month |
|---|---|---|
| Emails sent | 1,000/day x ~22 weekdays | ~22,000 |
| New leads contacted | emails / 3 (3-step sequence) | ~7,300 |
| Replies | 1 to 2% of leads | ~70 to 150 |
| Positive replies | ~1 per 1,000 leads (average ~1.3) | ~7 to 10 |
| Booked calls | 40 to 60% of positives | ~3 to 5 |
| New clients | calls x close rate | e.g. 20% → ~0.7 |

Then: revenue = clients x deal value, and ROI = (clients x LTV) / ListKit cost. Once they have results, use their real numbers: LTV = monthly revenue per client x months a client stays; CAC = acquisition spend / clients won; LTV:CAC = LTV / CAC (e.g. $3,000 spent, 3 clients at $2,000/month staying 4 months: CAC $1,000, LTV $8,000, 8:1). Show the payback in clients using their real plan price (from the overview's `billing`), e.g. "One client at $3k covers about N months of your plan."

These are benchmarks across many accounts, not promises. Offers, markets and copy vary a lot: high-LTV offers win with below-average rates, low-LTV offers need above-average ones.

## 3. Month one is not month two (say this explicitly)
Build a dated timeline from today:
- Branded: setup usually within 12 hours (tell them up to 72h), 14 days warmup, then 7 sending days of ramp (1 → 25/inbox). Full volume around week 4. Month 1 sends ≈ 25 to 35% of a full month.
- Pre-warmed: ready in minutes, 7 sending days of ramp. Full volume around day 9 to 10. Month 1 ≈ 70 to 80% of a full month.
- First replies usually come 1 to 3 weeks after first send. Sales cycles add more.
- The fair test is 90 days of steady sending and multiple campaigns.

Example output:
```
Your plan (1 Bundle, branded domains, today Oct 15)
Oct 18  domains live, warmup starts
Nov 2   first emails go out (ramping)
Nov 11  full volume, ~1,000/day
Nov 30  first check-in (~4,000 leads contacted): reply rate and first positives
Dec     first full month: ~7 to 10 positives, ~3 to 5 calls, ~0.7 to 1 client at 20% close
Late Dec  ~10,000 leads contacted: the fair point to judge the account
Goal of 2 clients/month needs ~3 Bundles of volume, or a better close rate.
```

## 4. Goal back-solve
clients needed / close rate / book rate (0.5) x 1,000 = leads per month needed. Leads x 3 = sends. Sends / 22,000 = Bundles. Show it, then show the cheaper lever if there is one (better follow-up to raise book rate, higher close rate, higher deal value).

## 5. Fit check (be honest, it saves refunds)
Flag clearly if:
- B2C / consumer audience: cold email rarely works there.
- Local, physical or real-estate businesses as the seller (contractors, salons, venues, brokers): the highest-churn group. Cold email works best for B2B services; for local buyers, phone and referrals usually beat it.
- Agencies selling to their own clients: lots of replies, but they churn when their client churns. Set up **listkit-results-report** weekly from day one.
- Tiny TAM (a few hundred companies): they'll run out of leads.
- They want it fully done for them: the Bundle sets up domains, inboxes, warmup, verified leads and AI copy, and the agent can build and launch campaigns. They still answer replies and take the calls. If they want it fully managed, point them to the ListKit team.
- No time to answer replies: results die in the inbox.

## 6. Never
No guarantees, no "conservative" projections presented as minimums, no "1,000 a day from day one" for new domains. Use "expect around", "benchmark", and real dates.

Close with the next action: usually **listkit-launch-first-campaign** or **listkit-volume-engine**.
