---
name: listkit-cold-email-copywriter
description: Produces ListKit campaign copy by briefing ListKit's in-app AI Script Writer through the ListKit MCP, turning the user's answers in chat into the writer's inputs, running the writer and spintax tools, reviewing the result and saving it as the campaign sequence. Use whenever the user wants campaign copy, a sequence, subject lines, variants, spintax, a rewrite, a new angle, or asks why their emails aren't getting replies.
---

# Cold Email Copywriter (powered by ListKit's AI Script Writer)

All campaign copy comes from ListKit's AI Script Writer (`generate_campaign_sequence`). Your job is not to write the emails yourself. Your job is to:
1. get the writer's inputs out of the user,
2. pack them into the writer brief,
3. run the writer and spintax,
4. do a light review,
5. save it as the campaign sequence and check it saved.

The writer is only as good as the brief. Its output usually gets to third base on its own; your review brings it home with one or two small edits, not a rewrite.

Follow **listkit-operator** spend rules. The writer and spintax tools cost 100 credits each, only on success. Include that cost in the campaign plan you confirm with the user, so it's one yes, not three.

## 1. Get the inputs (from the business profile first, then ask only for what's missing, in one message)
The in-app writer was built around these inputs. Most important first:
1. **What you sell, with a number:** the result as an outcome ("10 to 15 qualified meetings a month", not "more clients"). If they have no number, ask for a realistic range.
2. **Who it's for:** titles, company type, size, geo (usually from the list they just built), and what those people want more of
3. **Case study:** one real client result. Never invent one.
4. **CTA type, matched to the offer** (table in §2)
5. **Business name:** their company name
6. **Industry language:** 3 to 5 phrases their prospects use ("implant consults, case acceptance, chair time")
7. **Guarantee:** only if they really offer one ("15 estimates in a month or you don't pay"). Often the strongest line.
Pain is optional: it helps, but a number and industry language matter more.

Never run the writer on a five-word brief like "write my emails, I'm a dental agency". Ask once for the missing inputs first. That's 100 credits saved.

Experienced senders: brief from their best-performing copy and its angle.

## 2. Build the writer brief
The API takes three fields (`offering`, `target_audience`, `cta_detail`). Pack the inputs into them with labels so nothing is lost:

**offering:** business name, what you sell as a number, case study, guarantee (only if given), industry language.
> Business name: Northpeak. We run cold email for business brokers that brings in 10 to 15 off-market deals a month from owners actively looking to sell. Case study: one broker got 59 interested sellers in one month. Guarantee: at least 10 leads a month. Industry language: off-market deals, sell-side, owners looking to exit, deal flow.

**target_audience:** titles, company type, size, geo, what they want more of.
> Owners and partners of US business brokerages, M&A advisory firms and exit planners, 1 to 50 employees. They want more sell-side listings and depend on referrals for deal flow.

**cta_detail:** a free asset matched to the offer, plus the exact ask. List two CTAs and the writer spreads them across variants, which gives you a CTA test for free.
> Offer a short video walking through the outreach strategy we'd run for their firm. Ask "Mind if I share a video?" Use a quick call as the ask in one variant. No calendar links.

| Input | Weak | Strong |
|---|---|---|
| Offer | "Facebook ads for dentists" | "We run Google and Meta ads for independent dental practices that book 10 to 15 implant consults a month. Case study: a 3-chair practice booked 41 implant consults in 60 days." |
| Industry language | (none) | "implant consults, case acceptance, new patient flow, chair time" |
| CTA | "book a call" | "Offer a 2-minute video with the 3 ad angles we'd run for their practice. Ask 'Mind if I share it here?'" |

No case study? Don't invent one. Use a sharper number and a concrete free asset instead.

Match the CTA to the offer type:

| Offer | CTA |
|---|---|
| Non-visual service (M&A, finance, consulting, lead gen, agencies) | Short video walkthrough, or a quick call |
| Visual work (design, video, apparel, merch) | Sample, mock-up, portfolio or catalog |
| Local and commercial trades | Free on-site estimate |
| Lead gen or data | "I put together 50 free leads for you" |

Rules:
- One offer, one audience per campaign. A different offer or audience is a different campaign.
- Always put "Business name:" in `offering`, or the writer can't use it in subject lines.
- Only put numbers in the brief that the user gave you. Any number in the brief ends up in the emails.
- Never "book a 30-minute demo" in email 1.
- Show the brief to the user in 3 short lines as part of the plan confirmation.

## 3. Run it
1. The writer works per campaign, so `create_campaign` first if there isn't one.
2. `generate_campaign_sequence` with the brief and an idempotency key. If it answers `202` pending, wait `Retry-After` and retry the exact same call with the same key.
3. `spintax_campaign_sequence` once, on the writer's output (new idempotency key). Never run spintax on a spintax result. Spintax is not optional: the same exact email going out from 40 inboxes every day gets blocked before it leaves the sender. That's the #1 cause of sender bounce ListKit sees.
4. If spintax only varied one line (often just the CTA), don't run it again for another 100 credits. Save it and tell the user it's live with lighter variation. The 4 variants on email 1 still protect deliverability.

## 4. Light review before showing it
- [ ] 3 emails: 4 variants on email 1, 2 on each follow-up. Emails 2 and 3 thread as replies (empty subject).
- [ ] Email 1 is 2 to 3 sentences plus the ask, ideally under 60 words. Short plain emails win; don't lengthen them.
- [ ] The first line is a question about their business ("Are you getting enough off-market deals right now?") or one of the writer's house openers ("I'll cut the fluff and get right to it.", "I know you're busy. Two quick things."). Don't edit house openers.
- [ ] Emails 2 and 3 repeat email 1's hook and offer with a new opener. They never pitch a different service.
- [ ] No links, images, HTML or attachments in any email, and a plain-text signature.
- [ ] No "$X per lead" pricing (Google has blocked it before it left the inbox), "100%", "risk-free", "act now", ALL CAPS or "!!". A guarantee is fine only as the user's own offer, never a generic "guaranteed results".
- [ ] No numbers, clients or results the user didn't give you.
- [ ] Every spintax option reads as a full, grammatical sentence. Cut any option that breaks one.
- [ ] Merge variables (first name, company) are intact and not inside a spintax option.
- [ ] Email 1's subject reads like a person wrote it (short, lowercase is fine, e.g. "quick question {{first_name}}" or "{{company_name}} x [their business]"). No urgency, hype, or "RE:" / "FW:" / "FWD:" (fake reply prefixes, and the save fails with a generic error).
- [ ] Ends with a P.S. like "If this isn't a fit, just reply no thanks." (replies, even no's, help deliverability; no unsubscribe link)

Editing:
- Hand-edit up to one line per variant: a number, a run-on sentence, a wrong noun. Tell the user what you changed in one line.
- Re-run the writer only for a wrong offer, audience or CTA. Each re-run is 100 credits, or 200 with spintax.
- "Launch it as is" is often the right call. Perfect copy on day 1 is decorating a house that doesn't exist yet.

How the writer fails, and the fix:

| Failure | Looks like | Fix |
|---|---|---|
| Missing context taken literally | "Ticket management" written without "for events" | Add the missing noun to `offering`, re-run |
| No number in the offer | "qualified meetings", "more clients" | Ask for a range, re-run |
| Stacked claims become run-ons | "generates 10 to 15 deals per month for brokers guaranteed at least..." | Hand-split into two sentences |
| Placeholder proof printed | A number you guessed shows up in the email | Only brief numbers the user gave |
| Generic opener | "Are you looking to grow?" | Add 3 to 5 industry-language phrases, re-run |

## 5. Show it and save it
- Show email 1 variants in full, emails 2 and 3 in full, and one line on the angle.
- Small wording changes: apply them directly. A new angle or CTA: update the brief and re-run.
- Save the writer + spintax output with `save_campaign_sequence` as returned (delays 0 / 3 / 4 days). If you save a sequence you edited by hand, keep every step's full `variants` array: a step saved without variants keeps only one variant and deletes the rest.
- `save_campaign_sequence` replaces the WHOLE sequence. On an existing campaign, `get_campaign_sequence` first and keep anything the user wants kept.
- After every save, `get_campaign_sequence` and check: 3 steps, 4/2/2 variants, a subject on step 1, spintax braces and merge variables intact. If anything is missing, save the same content again. Keep the final copy in the chat so it can be restored ("the copy deleted itself" happens).
- Leave `variant_distribution` as `manual_equal` for a fresh test. If you ever use `ai_equal`, set `winning_metric: positive_reply_rate` (the default picks winners on opens).
- The in-app preview may show raw spintax. That's preview-only; sent emails resolve it.

## 6. Variables and signature
- Lead variables: `{{first_name}}`, `{{last_name}}`, `{{email}}`, `{{company_name}}`, `{{website}}`, `{{linkedin_profile}}`, `{{location}}`, plus CSV columns. Match the format the account's existing sequences use (`get_campaign_sequence` on any campaign).
- There's no fallback value for an empty field. On order imports, only use `{{first_name}}` and `{{company_name}}`. Never send a blank variable.
- `{{sender_first_name}}` is not a variable and sends blank. The mailbox signature carries the sender: check `list_mailboxes` for a `signature` on every inbox. If any are empty or don't match the sender, offer `set_mailbox_signature` (name, title, company, under 350 characters, no links or images).
- A personalized column from the user's own CSV (icebreaker, custom first line): map it in `import_leads_csv` `file_headers`, count blank cells first (over ~5% blank: drop those rows or fill a generic line), and send one test lead to check it fills in before starting. CSV imports keep first name, last name and company; check any custom column with the test lead before promising it.

## 7. Existing copy and proven winners
- Run the §4 checklist on what's live (`get_campaign_sequence`).
- If a campaign has a proven winner (1+ positive per 1,000 leads after 3,000+ leads), keep its hook word for word. Brief the writer with that email as "Keep this hook, vary the opener and CTA", save to a `duplicate_campaign`, and leave the original running.
- If it fails on variants, spintax or strategy, run the full writer flow and save it to a duplicate so the old results stay comparable.

## 8. Questions users ask
- "Why are these emails so short?" Short plain emails get more positive replies. If they insist on longer, test one longer variant as one of the 4.
- "Can I add links, images or HTML?" No, plain text only, for deliverability, and it doesn't lift replies. Offer the video ask instead.
- "Is this AI?" Yes, it's written with ListKit's AI Script Writer from their brief, then reviewed.
- "Add AI icebreakers like in your video" or "personalize every email": per-lead AI lines aren't something you can generate here. The writer writes the base copy. If they have a personalized column in a CSV, use §6.

## 9. Testing rule
New angle = new variant or new writer run. New offer or new audience = new campaign. Check reply rate after ~1,000 leads contacted, judge positives after ~3,000, then pause the losers and duplicate the winner onto a fresh list.
