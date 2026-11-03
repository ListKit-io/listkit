---
name: listkit-launch-first-campaign
description: Takes a ListKit user from "I just signed up" or "I have nothing running" to a scheduled, live first cold email campaign in one conversation, using the ListKit MCP. Use when the user is new, asks how to get started, has zero live campaigns, says they're overwhelmed, or onboarding status shows no campaign launched. Also use for any user with paid sending capacity sitting idle.
---

# Launch My First Campaign

The goal of this conversation is binary: **a campaign scheduled or live before we finish.** Not a plan. Not "watch the videos". A campaign.

Why it matters: customers with zero live campaigns churn at ~60%. But launching alone doesn't protect them: most customers who leave had already sent something. What protects them is results in their first 21 days of sending: customers with 6+ positive replies (or 2+ per 1,000 sends) in that window churn about half as often. So get them live today, get them to full volume fast, and make sure every positive gets answered.

Where users actually stall: the biggest group of stuck accounts has inboxes READY and no campaign at all. Next: campaigns that went quiet (nothing sent in 4+ weekdays), then accounts still warming.

Follow **listkit-operator** rules for every spend.

## Step 1. Read the account (silent, no questions yet)

Run in parallel: `get_onboarding_status`, `list_mailboxes` (limit 100, follow `next_cursor`), `get_domain_stats`, `get_domain_provisioning_status`, `list_campaigns`, `list_orders`, `get_account_snapshot`.

Classify where they are:

| State | What you see | Path |
|---|---|---|
| A. No infrastructure | 0 domains, 0 inboxes | Step 2 |
| B. Infra provisioning | provisioning step not `ready` | Build everything now, schedule the start for when inboxes are ready |
| C. Warming | inboxes `warmup_status` WARMING | Build everything now, schedule the start for warmup end |
| D. Ready, nothing live | inboxes READY, 0 active campaigns | Skip any welcome tour. Go straight to the Step 3 questions, build and launch now |
| E. Has a real draft or paused campaign | Draft/Paused with leads or a saved sequence | Find what's missing (leads? sequence? senders? signatures?) and finish it. Ignore empty drafts, and never resume a draft the user didn't name. |
| F. Launched, but quiet | Active campaign, nothing sent in 4+ weekdays | Run the "why isn't it sending" checklist in **listkit-sending-health** before anything else |

If they open with "I have 0 credits and no idea how to start", answer the credits part in one line (credits buy leads, they don't limit sending, here's your balance and reset date) and keep going with this flow.

`get_onboarding_status` `completed` only means they subscribed. Campaign state comes from `list_campaigns`.

Tell them in one sentence where they are and the date they can send. Example: "Your 40 inboxes finish warmup on Thu Oct 22, so we'll build your first campaign now and schedule it to start that morning."

## Step 2. Infrastructure (only if state A)

- `check_prewarmed_availability` with their bundle count, then present both options neutrally. Don't push either as the default:
  - Pre-warmed (.info domains): sending in minutes, then the 7-sending-day ramp. Gets them sending sooner, but hasn't retained better: pre-warmed customers who rush a list out and see no quick result leave faster.
  - Branded (domains close to their brand): usually set up within 12 hours (tell them up to 72h), then 14 days warmup, then the same ramp.
- If `can_fulfill` is false: don't apologize and don't promise inboxes from anywhere else. "Pre-warmed is out of stock right now. With branded domains you'll send your first email on [date] and hit full volume around [date]. Let's build the campaign now so it starts that morning."
- If they bought pre-warmed and ask why checkout said "14-day warmup begins": that line doesn't apply to pre-warmed domains. Give their real dates.
- Let them choose.
- Pre-warmed: `pick_prewarmed_domains` then ask for 1 to 2 sender personas (first + last name, real person who'll reply) and a redirect URL (their website). Confirm, then `claim_prewarmed_domains` with an idempotency key.
- Branded: `search_new_domains` on their brand word, pick names, `draft_domain_order` (each domain listed by exactly 2 senders with different usernames, Google provider), confirm, `confirm_domain_order`.
- Don't wait for infra to finish. Move on and build the campaign now.

## Step 3. The 5-question ICP interview

Use the business profile first (**listkit-business-profile**): research their website and connected tools, then ask only what's still missing. With a full profile, skip straight to Step 4. Otherwise ask these together in one message, short. They feed the profile, the list and the AI Script Writer:

1. What do you sell, and what result does a client get, as a number? ("10 to 15 qualified meetings a month")
2. Who buys it? Job title, type of company, and the words they use for their world.
3. Where? (countries, states). One campaign sends on one time zone: if the countries span more than ~5 hours (US + UK, US + Australia), build one campaign and one order per region.
4. Company size you close best? (suggest 11 to 50, up to 200; their answer sets the filter)
5b. How many leads do you want to reach? (if they don't know: ~7,000 new leads a month per Bundle). The list builder works backwards from this number.
5. Your best proof: one client result. Plus your business name, and a guarantee if you really offer one.
The CTA you pick yourself from the offer type (copywriter §2): video or call for services, sample for visual work, free estimate for trades.

If they give a website instead, read it with your web tools and propose the ICP yourself.

Say these plainly up front, not after they've paid for a month:
- **B2C offers** (consumers, retirees, homeowners): cold email to consumers rarely works.
- **Local, physical or real-estate businesses as the SELLER** (contractors, salons, venues, brokers selling locally): the highest-churn group. Cold email works best for B2B services; for local buyers, phone and referrals usually beat it. If they go ahead, set expectations with the dated timeline.
- **Agencies selling to their own clients:** they get the most replies but churn when THEIR client churns. Offer **listkit-results-report** weekly from day one.
- **Tiny markets** (one city, one niche of 300 companies): the audience will run dry; widen geography or niche.

## Step 4. Build the list (hand the method to listkit-list-builder)

Target for campaign 1: **2,000 to 3,000 verified leads** (never under 1,000), one ICP, one country set. Not 67. Not 20,000.

Map the whole TAM while you're at it (list-builder §3c): every route, a count per route, a saved filter per route. Order the first test batch from the tightest route now; the rest is the runway for campaign 2 onward. Tell the user how many people they can reach in total, so they know the market won't run dry.

Quick recipe: `expand_job_titles` → industry or keywords (see list-builder §3) → `find_filter_values` for countries → `count_search_results` → `get_search_analytics` on `personal_country` to make sure nothing leaks outside their countries → show 10 sample rows → `estimate_order` → confirm → `create_order` with `fields: ["work_email"]`.

Warming? If the first send is within ~7 days, order now so leads are waiting. If it's further out (branded domains), build the list, save the filter and show the estimate now, then order about a week before the start date so the emails are freshly verified. Finish everything else now (sequence, senders, signatures, schedule) and offer a one-time scheduled task for ~7 days before the start that re-checks the estimate, prepares the order and import with their cost, and notifies the user. The order and import run only after the user's yes and the server's confirmation.

## Step 5. One plan, one yes

Before spending anything on copy or launch, show the whole plan in one block:
```
Campaign: Dental owners US - Implant ads - Oct 15
List: 2,000 verified leads (ordered, ~2,000 credits)
Copy: ListKit AI Script Writer + spintax (200 credits), brief:
  Offer: ... / Audience: ... / Ask: ...
Import: the 2,000 ordered leads into this campaign when the order completes
Sending: all 40 ready inboxes attached, signatures set where missing,
  Mon to Fri, 8am to 5pm ET, START on Thu Oct 22
Go?
```
If starting the campaign reserves credits (`get_campaign_credits_info` `credits_needed`, or a `commitment_warning`), you'll check that before START in Step 7. If it's more than the plan above, show it and get a yes before starting.
This yes covers exactly the plan above. Each spend call still goes through the server's confirmation. Anything not in the block, or costing more, needs a new yes.

## Step 6. Write the sequence (hand the method to listkit-cold-email-copywriter)

1. `create_campaign` named `[ICP] - [Offer] - [Mon DD]`. If it answers with an error, check `list_campaigns` before retrying: the campaign may have been created anyway, and a retry makes a duplicate.
2. Brief and run ListKit's AI Script Writer (`generate_campaign_sequence`) with the offer, audience and easy-yes CTA from the interview, then `spintax_campaign_sequence`.
3. Review it against the copywriter checklist, show it, apply their small edits once, move on. Perfect copy on day 1 is decorating a house that doesn't exist yet. Volume and testing find the winner.
4. `save_campaign_sequence`: 3 steps, delays 0 / 3 / 4 days, follow-ups with empty subject so they thread. Then `get_campaign_sequence` to confirm it saved complete (copywriter §5).

## Step 7. Assemble and launch

1. Add leads once the order completes: poll `get_order`, then `import_leads_from_orders` (`order_ids`, `campaign_ids`), then `get_lead_import_status` until done. Read `statistics` (`upload_count` added, `duplicate_count`, `block_count`, `bounce_count`). If added is under 80% of what you sent, say so in one line ("1,240 of 2,000 went in: 700 were already in another campaign"). Don't call the campaign ready until 1,000+ were added. If most of the drop is duplicates from the user's OTHER campaigns, ask before re-importing with `ignore_duplicate_leads_in_other_campaign: true`. Never set `ignore_global_block_list` or `ignore_community_bounce_list` to true unless the user asks.
2. Senders: attach EVERY sending-ready inbox on the account (`list_mailboxes` with `limit: 100`, following `next_cursor` to the end, all with `warmup_status` READY and not disconnected; check the total against `get_domain_stats`) via `update_campaign` or `set_campaign_senders` with the full list. That's the default for every campaign, so it can use the account's full capacity. Only use fewer if the user explicitly asks to split inboxes between campaigns or clients.
3. Schedule: `time_zone` = the prospects' IANA zone (e.g. `America/New_York`), `days_of_week` [1,2,3,4,5] (0 = Sunday), `start_hour` "08:00", `end_hour` "17:00", `schedule_start_time` as a full ISO 8601 datetime (e.g. `2026-10-22T08:00:00-04:00`), never a bare date. Start = now if inboxes are ready, else the warmup end date.
4. Signatures (blocking): every attached inbox needs a signature that matches its sender (`list_mailboxes` `signature`). If any are empty or wrong (a test signature, a different name), set one with `set_mailbox_signature` (confirm, under 350 characters, name + title + company). Don't launch with blank signatures.
5. `get_campaign_credits_info` to make sure nothing blocks it, then START the campaign yourself (`update_campaign` status START, or `set_campaign_status`). The user already said yes to the plan. Don't leave it in Draft for them to click.
6. Read it back with `list_campaigns` and confirm the status is Active (or scheduled for the ready date).

If inboxes are still warming, start it with the schedule set to the warmup end date and say exactly when it goes out.

## Step 8. Lock in expectations (say all of this, with dates)

- "You'll be sending your first emails on [date] and hit full volume around [date]."
- "Benchmark: about 1 positive reply per 1,000 leads contacted. At full volume that's roughly 7 to 10 interested replies a month per Bundle."
- "40 to 60% of those should turn into booked calls if you reply fast."
- "Replies show up 1 to 3 weeks after the first email. Positives are rare, so we judge a campaign after ~3,000 leads contacted and the account after ~10,000, not after day 3. You'll reach 3,000 around [date]."
- "This works on volume and testing: keep 2 to 4 campaigns of 2,000+ leads live, and launch a new test every week."
- "The goal for your first 3 weeks of sending is your first handful of interested replies, answered within minutes."

If `get_onboarding_status` shows the strategy call isn't done, give them the link before you finish and suggest a date. It's part of getting set up, not optional.

Then book the review: offer a calendar reminder for a results check ~21 days after first send, the window that decides whether customers stay (use their calendar connector if they have one). That review should be with this same workflow: run **listkit-campaign-doctor**.

## Step 8b. Set up the inbox routine
Before replies start coming in, set up the inbox routine (**listkit-reply-desk** §1), which drafts every reply and follow-up for one-tap approval: booking link, what the call is, pricing they'll share, who takes calls, time zone. Every positive then has a reply waiting for approval within the hour and gets followed up until it books, which is what decides the first 21 days.

## Step 9. Queue campaign 2 before you leave

Ask: "What's the next ICP or angle we test?" If they have an answer, run steps 4 to 7 again. Offer to set up the weekly routine from **listkit-volume-engine** as a scheduled task so the account never goes idle. Two campaigns live on day one beats one perfect one.

## Done means

- [ ] Infra ready or provisioning with a known date
- [ ] Campaign 1 created with 1,000+ leads actually added (or, for a far-off branded start, the order and import prepared for the user's yes ~7 days before), a 3-step sequence, every READY inbox attached with a signature, weekday schedule
- [ ] Started, or scheduled with a date the user heard
- [ ] Benchmark and dated timeline said out loud
- [ ] Inbox routine on
- [ ] Review date set
- [ ] Campaign 2 ICP named
