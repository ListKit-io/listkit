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

Call `get_account_overview`. One read gives the onboarding status, the inboxes, the domains (`domains.stats`, the provisioning step `domains.provisioning.step` and the newest domain order `domains.latest_order`), the campaigns, the orders, and the credits and plan. The overview already has the domain state, so don't read `get_domains(action=orders)` or `get_domains(action=domains)` in this skill, except for a failed order (state H) or a failed overview read of the orders (`errors.domain_orders`). Its `inboxes` part has the counts (`ready`, `warming`, `disconnected`, `without_signature`, `signature_name_mismatch`), `ready_ids` and the latest ramp end. If it says `inboxes_first_pages_only: true`, the counts cover only the first 500 inboxes.

Classify where they are:

| State | What you see | Path |
|---|---|---|
| A. No infrastructure | 0 domains, 0 inboxes, `domains.latest_order` null or `cancelled`, and no `errors` key `inboxes`, `domain_stats`, `domain_provisioning` or `domain_orders` | Step 2 |
| B. Infra provisioning | provisioning step not `ready`, or `domains.latest_order.status` `pending` or `active` with 0 inboxes | Build everything now, schedule the start for when inboxes are ready |
| C. Warming | inboxes still warming (`inboxes.warming` above 0) | Build everything now, schedule the start for warmup end |
| D. Ready, nothing live | inboxes READY (`inboxes.ready` above 0), 0 active campaigns | Skip any welcome tour. Go straight to the Step 3 questions, build and launch now |
| E. Has a real draft or paused campaign | Draft/Paused with leads or a saved sequence | Find what's missing (leads? sequence? senders? signatures?) and finish it. Ignore empty drafts, and never resume a draft the user didn't name. |
| F. Launched, but quiet | Active campaign, nothing sent in 4+ weekdays | Run the "why isn't it sending" checklist in **listkit-sending-health** before anything else |
| G. Unconfirmed domain order | `domains.latest_order.status` `draft` | They drafted a branded order and didn't confirm it. Offer to finish that order: tell them its `domain_count` and the day they drafted it (`created_at`), confirm, then `confirm_domain_order` with its `id`. Don't start a new order |
| H. Failed domain order | `domains.latest_order.status` `failed` | Read `get_domains(action=orders)` one time for that order's `failure_kind` and `can_retry`, and tell them what happened (below) |

Which state wins:
- F comes first: its checklist runs before anything else.
- Then the inboxes, in this order: D, C, B. Ready inboxes win: if some are READY and some still warming, launch on the ready ones now.
- G and H apply only when none of B, C and D matches, so inboxes from an older order always come first.
- A applies only when nothing above matches.
- E can come with any of these. Finish the campaign on that state's path. With G or H, deal with the domain order first, then finish the campaign.

For state H, `failure_kind` means:
- `temporary_retrying`: a vendor fault that ListKit retries automatically. With `can_retry` false, ListKit is still retrying it: treat it as state B. With `can_retry` true, the retries stopped and the retry rule below applies.
- `pool_exhausted`: the pre-warmed pool couldn't cover it. Go to Step 2 and show both options there.
- `needs_user_fix`: the request was rejected. It can be fixed and sent again, but no field says what to fix. Don't confirm it again unchanged. Offer `request_human_help` with a summary.
- `needs_support`: ListKit has to resolve it. Offer `request_human_help` with a summary.

If `can_retry` is true and the kind is not `needs_user_fix`, offer to confirm it again: `confirm_domain_order` with its `id` and a new `idempotency_key`. If `can_retry` is false, don't try it again.

The overview lists each part it couldn't read under `errors`:
- `errors.domain_orders`: `latest_order` is null because the read failed, not because there is no order. Read `get_domains(action=orders)` one time and classify with its newest order. If that read fails too, tell them the domain order state can't be read right now. Don't start a new domain order. Go on with what doesn't need it (the Step 3 questions), or offer `request_human_help`.
- `errors.inboxes`, `errors.domain_stats` or `errors.domain_provisioning`: it isn't state A, so don't go to Step 2. Tell them which part couldn't be read, and go on with what doesn't need it.

If they open with "I have 0 credits and no idea how to start", answer the credits part in one line (credits buy leads, they don't limit sending, here's your balance and reset date) and keep going with this flow.

The overview's `onboarding` `completed` only means they subscribed. Campaign state comes from its `campaigns` rows.

Tell them in one sentence where they are and the date they can send. Example: "Your 40 inboxes finish warmup on Thu Oct 22, so we'll build your first campaign now and schedule it to start that morning."

## Step 2. Infrastructure (only if state A, or state H with `pool_exhausted`)

- `get_domains(action=prewarmed_availability)` with their bundle count, then present both options neutrally. Don't push either as the default:
  - Pre-warmed (.info domains): sending in minutes, then the 7-sending-day ramp. Gets them sending sooner, but hasn't retained better: pre-warmed customers who rush a list out and see no quick result leave faster.
  - Branded (domains close to their brand): usually set up within 12 hours (tell them up to 72h), then 14 days warmup, then the same ramp.
- If `can_fulfill` is false: don't apologize and don't promise inboxes from anywhere else. "Pre-warmed is out of stock right now. With branded domains you'll send your first email on [date] and hit full volume around [date]. Let's build the campaign now so it starts that morning."
- If they bought pre-warmed and ask why checkout said "14-day warmup begins": that line doesn't apply to pre-warmed domains. Give their real dates.
- Let them choose.
- Pre-warmed: `get_domains(action=prewarmed_pick)` then ask for 1 to 2 sender personas (first + last name, real person who'll reply) and a redirect URL (their website). Confirm, then `claim_prewarmed_domains` with an idempotency key.
- Branded: `get_domains(action=search_new)` on their brand word, pick names, `draft_domain_order` (each domain listed by exactly 2 senders with different usernames, Google provider), confirm, `confirm_domain_order`.
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

Quick recipe: `expand_job_titles` → industry or keywords (see list-builder §3) → `find_filter_values` for countries → `preview_search` → `get_search_analytics` on `personal_country` to make sure nothing leaks outside their countries → show 10 sample rows → `estimate_order` → confirm → `find_and_order_leads` with `fields: ["work_email"]`.

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
If starting the campaign reserves credits, the `launch_campaign` confirmation in Step 7 says up to how many (and any commitment warning), and you'll check that before START. The leads go in before the launch, so that figure counts them. If it's more than the plan above, show it and get a yes before starting.
This yes covers exactly the plan above. Each spend call still goes through the server's confirmation. Anything not in the block, or costing more, needs a new yes.

## Step 6. Write the sequence (hand the method to listkit-cold-email-copywriter)

1. `create_campaign` named `[ICP] - [Offer] - [Mon DD]`. If it answers with an error, check `list_campaigns` before retrying: the campaign may have been created anyway, and a retry makes a duplicate.
2. Brief and run ListKit's AI Script Writer (`generate_campaign_sequence`) with the offer, audience and easy-yes CTA from the interview, then `spintax_campaign_sequence`.
3. Review it against the copywriter checklist, show it, apply their small edits once, move on. Perfect copy on day 1 is decorating a house that doesn't exist yet. Volume and testing find the winner.
4. `save_campaign_sequence`: 3 steps, delays 0 / 3 / 4 days, follow-ups with empty subject so they thread. Then `get_campaign_sequence` to confirm it saved complete (copywriter §5).

## Step 7. Assemble and launch

Add the leads first, then one `launch_campaign` call attaches the senders, saves the schedule and STARTs. The user approves it once, and ListKit's confirmation names every step and its cost.

1. Add leads once the order completes: poll `get_order`, then `add_order_contacts_to_campaign` (`order_ids`, `campaign_ids`), then `get_leads(action=import_status)` until done (the `file_id` is in the import answer's `imports`). Read `statistics` (`upload_count` added, `duplicate_count`, `block_count`, `bounce_count`). If added is under 80% of what you sent, say so in one line ("1,240 of 2,000 went in: 760 were duplicates or on the block list"). Don't call the campaign ready until 1,000+ were added, and don't launch it before that: fill it first. People who are already leads in the user's OTHER campaigns go in by default (`ignore_duplicate_leads_in_other_campaign: false`). `true` LEAVES OUT every contact that is already a lead in another campaign: set it only if the user asks to keep those people out, and ask before you re-import with it. Never set `ignore_global_block_list` or `ignore_community_bounce_list` to true unless the user asks.
2. Senders: `senders: "all_ready"` attaches EVERY sending-ready inbox on the account (warmed up and connected, from at most 5 pages of 100 inboxes, so up to 500 inboxes). That's the default for every campaign, so it can use the account's full capacity. Only use fewer if the user explicitly asks to split inboxes between campaigns or clients: then pass those inboxes' ids from `list_mailboxes`. If the inboxes are still warming (state C), `"all_ready"` finds none and refuses. Then pass the ids of the warming inboxes from `list_mailboxes` that are connected and whose signature names their sender, and use `get_account_health` for the date they are ready. If the account has no inboxes yet (state B), finish everything else and launch once the inboxes exist (offer a one-time scheduled task for that date).
3. Schedule: `time_zone` = the prospects' IANA zone (e.g. `America/New_York`), `days_of_week` [1,2,3,4,5] (0 = Sunday), `start_hour` "08:00", `end_hour` "17:00", `schedule_start_time` as a full ISO 8601 datetime (e.g. `2026-10-22T08:00:00-04:00`), never a bare date. Start = now if inboxes are ready, else the warmup end date.
4. Signatures (blocking): every attached inbox needs a signature that matches its sender. Step 1's `inboxes.without_signature` and `inboxes.signature_name_mismatch` count the empty and mismatched ones. To see which inboxes they are, and to catch a wrong one (a test signature, a different name), read `signature` with `list_mailboxes` (`limit: 100`, follow `next_cursor`). If any are empty or wrong, set one with `update_mailboxes(action=signature)` (confirm, under 350 characters, name + title + company). Don't launch with blank signatures.
5. Launch: `launch_campaign` with `id`, `senders: "all_ready"`, `schedule` and a new `idempotency_key` (e.g. `launch-2026-10-22-dental-us`), without `order_ids` (the leads are in from item 1). If the Step 5 plan set a credit limit for START, pass it as `max_credits`: the tool does not START above it. Start the campaign yourself: the user already said yes to the plan. Don't leave it in Draft for them to click.
   - Before it asks, the tool checks the senders. If a sender has a blank signature, it refuses and names those inboxes. Tell the user what the refusal says (which inboxes have no signature), set a signature on each as in item 4, then call `launch_campaign` again.
   - Its confirmation says how many inboxes `"all_ready"` takes (check that against `inboxes.ready` from Step 1) and which it leaves out because their signature names someone other than the sender. Tell the user which were left out, and fix their signatures if the user wants them on the campaign.
   - The confirmation also says up to how many credits START reserves, with any commitment warning. If it's more than the Step 5 plan, show it and get a yes before starting.
   - If it stops at a step, it says which steps ran and what to do next. Tell the user in plain words, and get their yes before you run it again or change anything: a new launch with a new `idempotency_key` can spend credits.
6. Read it back with `list_campaigns` and confirm the status is Active (or scheduled for the ready date).

If inboxes are still warming, start it with the schedule set to the warmup end date and say exactly when it goes out.

## Step 8. Lock in expectations (say all of this, with dates)

- "You'll be sending your first emails on [date] and hit full volume around [date]."
- "Benchmark: about 1 positive reply per 1,000 leads contacted. At full volume that's roughly 7 to 10 interested replies a month per Bundle."
- "40 to 60% of those should turn into booked calls if you reply fast."
- "Replies show up 1 to 3 weeks after the first email. Positives are rare, so we judge a campaign after ~3,000 leads contacted and the account after ~10,000, not after day 3. You'll reach 3,000 around [date]."
- "This works on volume and testing: keep 2 to 4 campaigns of 2,000+ leads live, and launch a new test every week."
- "The goal for your first 3 weeks of sending is your first handful of interested replies, answered within minutes."

If the overview's `onboarding` (Step 1) shows the strategy call isn't done, give them the link before you finish and suggest a date. It's part of getting set up, not optional.

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
