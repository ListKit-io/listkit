---
name: listkit-business-profile
description: Builds and keeps a ListKit user's business profile (what they sell, offer, proof, ideal customers, goals, lead target, booking and reply details) by researching their website and connected tools first and asking only for the gaps, then stores it so every later ListKit task reuses it. Use at a user's first ListKit session, when no profile exists, when the user says their offer, pricing, ICP or goals changed, or before launch, copy, list building, inbox routine or planning if the profile is missing a field those need.
---

# Business Profile

Every ListKit task needs the same context: what they sell, to whom, with what proof, toward what goal. Ask once, store it, reuse it everywhere, so the user never answers the same question twice. Speed matters more than completeness: the profile exists to get a campaign live faster, never to slow it down.

Follow **listkit-operator** rules.

## 1. Load first
At the start of a session, look for the stored profile (§4). If it exists, use it silently and only ask about fields the current task needs that are empty. If it's older than ~90 days, confirm the offer and goal in one line.

## 2. Research before asking
1. **Website:** read their site with your web tools (home, services, pricing, case studies, about). Free. If you can't read the site, ask the user for a one-line description instead.
2. **Connected tools, if available and the user agrees:** ask once before reading their other connectors ("Can I check your CRM, email, calendar and docs to fill this in?") and read only what they OK. CRM (won deals: industries, titles, company sizes, deal values), email (their own sales emails and best-performing outreach), calendar (booking link, call length), docs (pitch deck, case studies), and anything they've already told you in this or past chats.
3. **ListKit account:** `get_account_overview`: one read for the user and plan, the campaigns (what they've run), and how many saved searches (ICPs they've built) and exclude lists there are. When those counts are above 0 and you need the names, read `list_filters(action=saved)` and `get_exclusions(action=lists)`. If `get_account_overview` is not in your tools, the plan has no Email Engine or the connection lacks one of its scopes: read `get_me` and `get_account_snapshot` instead. A `plan_required` answer from `get_account_overview` means the same thing, not that the account has no Bundle.
Fill every field you can, and mark where each came from.

## 3. Ask only the gaps, in one message
Show what you found in 3 to 5 short lines ("Here's what I've got: you sell X to Y, best proof is Z") and ask only what's missing or unclear, at most 6 questions. Ask in this priority order:

| Field | Feeds |
|---|---|
| What you sell and the result, as a number ("10 to 15 meetings a month") | copy, reply desk |
| Who buys: titles, company type, size, geo, and the words they use | lists, copy |
| Best proof: one client result, and a guarantee if real | copy |
| How many leads to reach, or the goal (new clients or revenue per month) | lists, planner |
| Deal value, and close rate if known | planner |
| Booking link, who takes calls, time zone, pricing you're OK sharing | inbox routine |
| Customers and open deals to keep off lists (domains or a CSV) | exclusions |
| Agency? Then list the clients you run outbound for | one profile per client |
| Done cold email before? Best copy and numbers | copy, expectations |

Never block a launch on the profile. If the user wants to go now, ask the first three fields and fill the rest later as tasks need them.

## 4. Store it
Write it as one compact record (template below) and, with the user's OK, save it where it survives across chats:
- in the AI tool's memory or project knowledge, if available (titled "ListKit business profile: [business name]"),
- otherwise offer to save it as a doc in their connected drive, and re-read it at the start of each ListKit session.
Agencies: one record per client, plus one for their own agency.

```
ListKit business profile: [Business] (updated [date])
Website: ...
Offer: ... (result as a number) · Price/deal value: ... · Close rate: ...
Proof: ... · Guarantee: ... · Assets: video / case study links
ICP: titles ... · company type ... · size ... · geo ... · their words: ...
Goal: ... new clients/month · Lead target: ... leads/month
Calls: booking link ... · taken by ... · time zone ... · pricing to share: ...
Exclusions: ... (on ListKit exclude list: yes/no)
Cold email history: ...
Sources: website, HubSpot, user (date)
```
Never store passwords, API keys, card or bank numbers, or anything the user asks you not to keep.

## 5. Use it everywhere
- **listkit-launch-first-campaign:** skip questions the profile answers.
- **listkit-cold-email-copywriter:** the writer brief comes straight from Offer, Proof, Guarantee, ICP words and Assets.
- **listkit-list-builder:** ICP and lead target set the routes and the number to work back from. Exclusions go on the exclude list (`manage_exclusions(action=add_entries)` or `manage_exclusions(action=import_list)`).
- **listkit-reply-desk:** inbox routine setup comes from the Calls line.
- **listkit-outbound-planner** and **listkit-results-report:** goal, deal value and close rate.

## 6. Keep it fresh
When the user mentions a new offer, price, ICP, result or goal, update the record in the same turn and say so in one line ("Updated your profile with the new $3k offer."). New proof from their own campaigns (a client win, a winning angle) goes in too.
