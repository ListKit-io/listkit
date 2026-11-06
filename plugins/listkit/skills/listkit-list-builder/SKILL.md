---
name: listkit-list-builder
description: Turns a plain-English ideal customer description into a precise, right-sized ListKit lead list and order, using ListKit MCP search tools. Use whenever the user wants to find leads, build or refine a list, target a niche, size an audience, exclude existing customers, or place a lead order. Prevents the two classic failures, lists so loose they're noisy and lists so narrow they're useless.
---

# List Builder

Lead questions about data, filters and list building are the #1 support topic at ListKit. Bad lists also cause the "verified leads bounce" and "I got Germany in my US list" complaints. Build lists like a pro, and prove the list is right before spending a credit.

Follow **listkit-operator** spend rules.

## The recipe

### 0. Start from the number, work backwards
Take the lead target and ICP from the business profile if it has them; otherwise ask once, up front: "How many leads do you want to reach?" (this campaign, or per month). If they don't know, set it from their capacity: each Bundle reaches ~7,000 new leads a month on a 3-step sequence, so 3 months of runway is ~20,000 per Bundle.
Then work backwards: target → how many routes it takes (§3c) → how many orders. Keep adding routes until the target is covered or the market is used up, and say which one happened. Never stop at the first search and call that the market.

### 1. Pin the ICP in one line
"[Titles] at [company type] with [size] employees in [geo]". Confirm it before searching if anything is ambiguous.

Only use filters the user asked for or agreed to. Never add a size band, title excludes or anything else silently. Propose them ("I'd target 11 to 50 employees and leave out assistants, OK?") and add them once they say yes. That's how ListKit search matches the web app.

### 2. Titles
- `expand_job_titles` with the user's words and named titles. Use `include` in `job_title.include`. Show the returned `exclude` titles (so "CEO" never pulls "Assistant to the CEO") and add them to `job_title.exclude` once the user agrees.
- If `expand_job_titles` comes back empty, use the titles the user named plus their obvious variants (CEO, Founder, Owner, Co-Founder, Managing Director) and propose excluding look-alikes (Assistant, Executive Assistant, Chief of Staff).
- Title clusters beat single titles: aim for 50 to 100 variants for common roles (1.6x to 2.9x the people). Brand-new niche roles barely expand, so widen with company keywords instead.
- Title includes match inside longer titles: "President" also pulls every "Vice President", "Director" pulls "Assistant to the Director". Skip ambiguous titles like President, or add the look-alikes to `job_title.exclude` (on a yes), and check the sample.
- Don't add `seniority_level` next to job titles. Pick one approach.

### 3. Company match (one method per list; see §3c to run several lists)
- **Clear industry** (insurance, software, construction): `find_filter_values` field `industry`. In SEARCH, add `suggest_keyword_cluster` phrases too: a company then matches the industry OR any phrase. In ORDERS it's the opposite: industry plus keywords means a company must match BOTH, which shrinks the pool. So order on the industry alone (the order engine adds its own keyword cluster) and treat `estimate_order` as the truth.
- **Niche with no clean industry** (cold email agencies, Shopify supplement brands, fractional CFOs): keywords only, built with **listkit-keyword-clusters** (core terms first, the full 150 to 250 term cluster when you need reach). Propose `keywords.exclude` for look-alikes (e.g. "staffing" when they want marketing agencies).
- `technology` (e.g. Shopify, Klaviyo, HubSpot) is a great tightener for ecom and SaaS ICPs.
- `topics` (intent) cuts the audience hard and is US-only. Use it as its own list (§3c), never stacked on another method.

### 3b. Local businesses (restaurants, bars, gyms, home services, clinics)
- Titles: Owner, Founder, CEO, Co-Owner only. Small businesses use loose titles, so don't expand into managers.
- Keep national chains out: cap company size (up to ~500), and add the big brand names to `keywords.exclude` if they show up in the sample.
- Exclude nonprofits and government with an industry exclude.
- Set person location AND company location to the same state(s), or you'll get owners who live out of state.

### 3c. Exhaust the TAM: several lists, one ICP
Running out of audience is the #1 reason customers churn. One search never finds the whole market, so for every new ICP map every route to it, run each as its own search, and order them as separate lists. Do this by default, not only when a count comes up short.

**Pick the routes for THIS query** (start with the top ones, add more until the target from §0 is covered):

| ICP type | Lead with |
|---|---|
| Local businesses (contractors, clinics, restaurants, salons) | SIC codes (O), city/state slices (M), industry (B), owner titles |
| Niche services and agencies | Keyword cluster (A), multi-thread titles (F), SIC codes (O) |
| SaaS and ecom | Technology (D), keyword cluster (A), revenue bands (K) |
| Clear traditional industry (insurance, construction, manufacturing) | Industry (B), SIC codes (O), size or revenue bands (G, K) |
| Selling to a function (heads of sales, marketing leaders) | Seniority + department (J), title clusters (F), then company routes |
| Timing-sensitive offers (hiring, funding, buying signals) | Intent (E), as its own campaign |
Always turn on every keyword type the search offers: company keywords should match name, description, industry tags and SEO terms, and people keywords should match title and profile. If a search tool exposes keyword-type settings, select all of them.

Routes (run the ones that fit, each with its own count and sample):

| Route | What it finds | Note |
|---|---|---|
| A. Keyword cluster | Companies that describe themselves this way, whatever industry they're filed under | Usually the biggest orderable pool. Take it to 150 to 250 terms (**listkit-keyword-clusters**) before you call the TAM: a 40-term cluster leaves reach on the table |
| B. Industry | Companies filed under the clean industry | Order on industry alone (the engine adds its own cluster) |
| D. Technology | Companies running a tool that signals the ICP (Shopify, HubSpot, Klaviyo) | Great for big platforms (ecom, SaaS). Detection of niche tools is sparse (a few hundred companies), so check the count before relying on it |
| E. Intent (`topics`) | Companies researching the problem right now | US only; smaller, hotter list, worth its own angle |
| F. Multi-thread titles | More people at the same companies: ops, delivery, marketing, partnerships, not just the founder | Often +20 to 50% with zero dilution. Pitch each role on the pain it owns |
| G. Size bands | The next band up or down (1-10, 51-200, 201-500) | Different budgets: adjust the offer |
| H. Geo | Other countries where the offer works (UK, CA, AU, EU) | One campaign per time zone |
| I. Adjacent ICP | Close markets with a different pitch (marketing agencies that could add cold email) | Always its own campaign and copy |
| J. Seniority + department | Decision-makers by level and function (`seniority` C-Level/Owner/VP + `department` Sales/Marketing) instead of a title list | Catches odd titles a title list misses ("Chief Growth Officer", "Partner"). Never stacked on job titles: its own list |
| K. Revenue bands | Companies by `company_revenue` instead of headcount | Finds lean, high-revenue companies the size filter skips (agencies, ecom, brokers). Use instead of `company_size`, not with it |
| L. Company HQ vs person location | People located in the target geo (`location`) vs companies headquartered there (`company_location`) | Run both: remote teams and multi-office companies show up in one and not the other |
| M. City and state slices | Local and regional markets city by city (`find_filter_values` city/state) | For local businesses, lets you go metro by metro and keeps time zones clean |
| O. SIC codes (`company_sic`) | Companies by official industry code (`find_filter_values` field `company_sic` for exact codes) | Cleanest route for traditional and local businesses. Search sizes it and orders take the same filter. If `estimate_order` refuses it, order through the keyword cluster with the SIC descriptions |

**Mine the taxonomy for keywords.** `find_filter_values` can look up SIC codes (`company_sic`), micro-industries (`company_micro_industry`), verticals (`company_vertical`), target audience (`company_target_audience`) and business model (`company_business_model`). Search filters on them (plus growth stage, funding, money raised, last funding date and hiring), and orders take the same filter as search, so order the search as run. Some funding and growth filters may still be refused on orders: if `estimate_order` refuses one, put its labels into the keyword cluster (route A), re-count and order that. Their labels also make great cluster terms on their own.

How to run it:
1. Build route A first (it's the biggest), then each other route. Show the user a TAM map:
```
TAM map: cold email agencies, founders + ops
A Keyword cluster (40 terms)        8,232 people · ~6,400 orderable   core pitch
F + ops/delivery titles             +2,456 people · ~1,190 orderable   core pitch
B Industry: Marketing & Advertising  3,100 net-new after A           core pitch
E Intent: "outbound sales" (US)        900 people                     "hiring SDRs?" angle
I Marketing agencies (adjacent)     24,000 people                     "launch a cold email offer" pitch
Total reachable: ~12k on the core pitch, ~36k with the adjacent ICP
```
2. Same pitch = same campaign. A different pitch (intent angle, adjacent ICP, a new size band) = its own campaign with its own copy.
3. Order the tightest-fit route first, one order at a time: wait for each order to finish before placing the next on an overlapping search, or they take each other's contacts and one can deliver zero. Net-new search skips anyone already bought, so each later route's count shrinks as earlier orders land: re-count right before each order, never add up raw counts as the total.
4. Save every route as its own saved filter (`[ICP] - [route]`) so **listkit-volume-engine** refills from all of them, not just one.
5. Push back on dilution, not on volume: if hitting the user's number means a weaker fit, say which route dilutes it and give it its own campaign instead of padding the core list.
6. When every route is used up, say it plainly with the total reached, and propose the next adjacent ICP before the lists run dry.

### 4. Size and geo
- `company_size`: if the user didn't say, propose `11-50` (extend to `51-200` if the pool is small) and use it once they agree. Owners of 1 to 10 companies rarely have budget unless the offer is cheap.
- Geo: always `find_filter_values` (field `country` / `state` / `city`) and use the exact spelling. States are 2-letter lowercase codes.
- Person location (`location`) vs company HQ (`company_location`): use person location by default for outreach timing; use company location when the buyer is "companies based in X".
- A campaign sends on one time zone. If the geos span more than ~5 hours (US + UK, US + Australia), plan one campaign per region, with one order per region (imports can't split an order by country).
- Order quantity is contacts, not companies. `max_per_company` caps contacts per company.

### 5. Count, then check for leaks (free)
- `preview_search` (it shows how many contacts can actually be delivered). Run counts one at a time (a new count cancels an older one). Search, count and estimate can disagree today: treat `estimate_order`'s `estimated_pool` as the number that matters for ordering.
- `get_search_analytics` on `personal_country` (or `company_country`) and on `job_title`. If more than ~5% sits outside the requested countries or roles, tighten the filters before ordering. Customers have churned over EU contacts in "US only" lists. Describe it to the user as refining the targeting ("narrowed to US-based owners only"), never as a data problem.
- Never suggest a filter change without running the count first, and show the count with every suggestion.
- `search_people` limit 25 and show a sample table. Ask: "Do these look like your buyers?" Skim it yourself for obvious misses.

### 6. Right-size it

| Net-new pool | Verdict | Action |
|---|---|---|
| Under 500 | Too narrow | Widen: `widen_search` (it suggests "change X for +N contacts"), the full keyword cluster (**listkit-keyword-clusters**), more titles, size up to 200, more geos. Don't launch a campaign on 67 people. |
| 500 to 1,000 | Too thin to launch | Widen to 1,000+ before ordering (same levers as above) |
| 1,000 to 3,000 | Test batch (2,000 to 3,000 is ideal) | Order it |
| 3,000 to 50,000 | Healthy market | Order a 2,000 to 3,000 test batch now, save the filter for refills. For the account as a whole, keep ~2 weeks of sending loaded (about 3,300 leads per Bundle) |
| 50,000+ | Too broad | Narrow by niche keywords or size so the copy can be specific |

Precision beats raw numbers, but a list too small to test teaches nothing.

### 7. Exclusions BEFORE ordering
The account exclude list does not yet apply to orders. Protect the user:
- Ask once: "Any customers, open deals or past contacts to keep off this list?"
- Domains/LinkedIn/emails: `manage_exclusions(action=add_entries)` (or `manage_exclusions(action=create_list)` + `manage_exclusions(action=import_list)` via `create_upload_link` for a CSV). This filters search results.
- For orders, pass up to 1,000 emails in `exclude_emails`. For bigger lists, run `get_exclusions(action=check)` on delivered contacts before importing them to a campaign and drop matches.

### 8. Estimate and order
- Search counts people; orders deliver verified emails, so delivered is lower than the search count. Before ordering, say "search shows 4,100 people; expect about [expected_delivered low to high] verified emails" (fall back to `estimated_pool` if the range is missing). If a preview says you already bought them all but `estimate_order` shows a pool, trust the estimate.
- Sizing can fail (an upstream error, a null estimate, "count unavailable", or "not found" on accounts with an exclude list). Retry once, and if a count fails, try `estimate_order` for the number. If there's still no number, say so, use the preview count as approximate, and order a smaller first batch (1,000 to 2,000) rather than guessing high. Never invent a pool size.
- `estimate_order` with the same targeting (orders use `job_titles`, `excluded_job_titles`, `industry`, `keywords`, `countries`, `states`, `company_size`... not the search field names). Respect `refusal` and never exceed `estimated_pool`.
- `fields: ["work_email"]` by default. Add `mobile` only if they'll call AND the list is US (mobile is US only, 3 credits each).
- `max_per_company`: 2 to 3 for SMB lists so one company doesn't eat the batch.
- Name it `[ICP] [geo] [Mon DD]`.
- Confirm cost, then `find_and_order_leads` with an idempotency key. Poll `get_order` until `completed` or `partial`. If partial, explain shortfall in one line (pool exhausted or per-company cap). If the Order History page errors in the app, give the contacts with `get_order_contacts`.

### 9. Save and recycle
- `save_search_filter` for every route (`[ICP] - [route]`) so refills are one click.
- Net-new results already exclude people they've bought before. If lists keep shrinking, that's dedupe working: open new keywords, titles or geos.
- Recycling: wait 2 to 4 months, remove everyone who replied, re-verify, and use a new angle.

### 10. Bad rows
If more than ~10% of the sample is off-target, tighten the filters and keep going, without commentary on data quality. Offer once to send ListKit feedback (`send_feedback`, kind `off_target`, filters used, share of bad rows): tell the user what it contains and send it only if they agree. Never include company names or contact data.

## Imported lists (their own CSV)
Verify before sending: `create_upload_link` (purpose `verification`), give them the link, then `upload_verification_file`, poll `get_verification_files(action=file)`, and only import the `good` rows. Never import unverified scraped lists into ListKit inboxes. Uploaded CSVs drive a lot of the "high bounce" complaints.

## Questions users ask
- "What database is this?" ListKit's own database, compiled from multiple sources and triple-verified. Don't name vendors.
- "Is quantity by company or by contact?" By contact (see `max_per_company`).

## Output
End with: the table, "X verified leads ordered at Y credits, Z credits left", the saved filter name, and the next step (usually: put them in a campaign).
