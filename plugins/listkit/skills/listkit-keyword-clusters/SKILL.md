---
name: listkit-keyword-clusters
description: Builds broad, accurate company keyword clusters for ListKit searches through the ListKit MCP, the terms that go into the keywords filter to find every company in a vertical. Use when the user asks for keywords, a keyword cluster or search terms for a vertical, when a niche has no clean industry, when a search pool is too small and needs widening, or when results pull the wrong kind of company.
---

# Keyword Clusters

Keywords match how companies describe themselves. Never search on a single keyword: in ListKit's own tests a cluster returned 3.9x to 261x the people of one term ("lead generation agency" alone: 6k people; a 250-term cluster: 1.23M), still in under a second. A broad, well-built cluster finds companies that sit under the "wrong" industry or none at all, which is where most of the extra reach in ListKit comes from. A sloppy one pulls insurance agencies into a marketing agency list. Build wide, then prove it with counts and samples before anyone spends a credit.

Follow **listkit-operator** rules. Keywords describe the market the user asked for, so building them is fine; propose any excludes and add them on a yes.

## 1. Seed, then build
1. `suggest_keyword_cluster` with the vertical (and the industry, if there is one) as a seed. Keep what's good.
2. Build the full cluster yourself from the angles below. Most verticals pull from six or more:
   1. **Core identity:** how the company names itself, plus every common synonym and spelling (med spa, medspa, medi spa, medical spa)
   2. **Services and products:** what they sell, at the level it appears on their site
   3. **Sub-categories:** specializations inside the vertical (residential vs commercial roofing; organic vs paid social)
   4. **Verticals they serve:** "healthcare marketing agency", "restaurant staffing", "dental practice broker". These pull the cleanest of any term type.
   5. **Technique, procedure or material terms:** specific enough to catch companies that describe the work, not the category (tpo roofing, morpheus8, hydro jetting)
   6. **Platform, brand and vendor signals:** software they run, equipment, partner badges (shopify plus partner, klaviyo partner, hubspot partner). Often the best proxy for size and funding.
   7. **Certifications:** where the industry has them (iso 13485, gaf master elite)
   8. **Business structure:** franchise, multi location, practice group, mso
   9. **In-industry slang:** what operators call themselves (smma, iso, dme)
3. Aim for 150 to 250 terms in the full cluster, and never more than 250: tested, past 250 adds no meaningful results. Under ~120 leaves reach on the table. Lowercase except proper nouns, max a few words per term, no near-duplicates ("roofing company / roofing companies / roofing firm" adds nothing past the first two). Never include the user's own company or competitors unless asked.
4. Order it core identity → sub-categories → verticals → adjacent, so the tail is easy to cut.

## 2. Prove it before ordering (free)
- Search with the core and sub-category terms first, then add the tail only if the pool is under target (list-builder §6). `count_search_results` or `preview_search` after each change, and show the count with every suggestion.
- `search_people` limit 25 and skim the sample. `get_search_analytics` on industry shows what the cluster actually pulled.
- In ORDERS, industry plus keywords means a company must match BOTH. For a niche, order on keywords alone; for a clear industry, order on the industry alone (list-builder §3). `estimate_order` is the truth.

## 3. Cut the noise (propose these, add on a yes)
- **Vendor side vs operator side:** most clusters grow a tail that pulls companies selling TO the target (roofing software, spa booking platform, ATS software). Name that block and cut it if they want operators.
- **Polysemous words:** *agency* (insurance, staffing, travel, real estate), *coach* (sports, bus companies), *spa* (nail salons, tanning), *broker* (real estate, insurance, freight), *clinic*, *studio*, *group*. Recommend specific `keywords.exclude` terms, don't just flag it.
- **Distinct buyers inside one cluster:** many verticals split into buyers who purchase differently (brands vs contract manufacturers; MCA funders vs ISOs vs banks; decorators vs manufacturers vs merch platforms). Recommend separate pulls, and separate campaigns, when the offer is price-sensitive or compliance-touching.
- **Adjacent trades that don't do the work:** roofing pulls siding and window contractors; plumbing pulls HVAC. Fine for breadth, bad if not wanted.
Say these in two or three short sentences of prose after the counts, not a lecture.

## 4. When the user wants the list itself
Give one flat comma-separated list they can paste into a single filter field: no headers, bullets, numbering or bold, at most one short line before it, then the cut notes from §3. Don't ask clarifying questions first: build the broad version and use the notes to say what to trim.

## 5. Repeat requests and overlaps
- Vertical already covered in this chat: say so in one line and re-send the original list as is if they want it. Don't silently rebuild it; a different list is a different pull, and they may be mid-campaign on the first.
- Offer a **second pass** instead: a fresh cluster from different angles (ownership and structure, staff and role titles, equipment and software vendors, associations and conferences, financing terms, operational language like rebooking rate or caseload). Flag when it's looser: generic words like *framework*, *launch*, *cohort* need a title or size filter alongside.
- When a new cluster overlaps an earlier one (med spa and plastic surgery; social media agency and marketing agency), say so up front. Net-new search already skips people they've bought, but dedupe the campaigns so nobody gets emailed twice.

## 6. Job titles (when they ask for titles, not keywords)
Split by buying center, not seniority:
- the functional champion who evaluates and sponsors,
- the economic buyer or procurement who signs,
- the owner or founder (at small companies, both of the above),
- gatekeepers (QA, regulatory, compliance, legal) who can't buy but can kill a deal.
Then say who to lead with by size: under ~30 employees the founder wears the functional hat; above that, leading with procurement on a new product usually dies on the approved-vendor list. Hand the titles to `expand_job_titles` (list-builder §2).

Title clusters work too, with smaller gains: a ~100-title cluster returned 1.6x to 2.9x the people of a single title for common roles (head of sales, head of marketing), and almost nothing extra for brand-new niche roles (gtm engineer: 1.2x). Build 50 to 100 titles for common roles; for niche roles, keep the user's titles plus a few variants and widen with company keywords instead.
