# What's On in Bermuda — Workspace Spec (v1)

**Status:** draft · **Date:** 2026-05-23
**Synthesized from:**
- Brainstorm: *What's On in Bermuda — core product shape* (`pkg-1ed9b488`)
- Research: *Bermuda event scraping sources — survey & ranking* (`pkg-556813a0`)
- Artifact: *Bermuda event scraping sources — ranked shortlist (v1)* (`.daveclaw/research/bermuda-event-scraping-sources-survey-ranking/bermuda-event-scraping-sources-ranked-shortlist-v1.md`)

---

## 1. Product summary & positioning

What's On in Bermuda is a comprehensive, scraper-driven aggregator of events happening on the island. It tells locals what is on this week, surfaces events that match their stated interests, and offers a calendar view to plan ahead. The end-state experience is a personalised push notification feed; the v1 surface is a web app showing the full scraped firehose plus a basic logged-in filter.

**Positioning:**
- **Direct competitor — whatson.bm.** Modern (Next.js + Supabase), user-submission driven, but small (~22 individual + 4 recurring events live). The product gap to attack is **comprehensiveness via scraping**, not novelty of concept.
- **Incumbent — bermuda.com/things-to-do/events-calendar/** (also powers the Royal Gazette calendar). Submission-driven, established but dated. Compete on freshness, coverage breadth, and personalisation.
- **Differentiator:** scrape-first comprehensiveness + dedupe + interest-driven push notifications. No other operator on the island is doing this.

## 2. Audience & jobs-to-be-done

**Primary audience: locals.** Design tradeoffs favour local patterns over visitor patterns. Visitors are a welcome secondary audience but should not drive the core UX.

**Three core jobs:**

1. **Discovery** — "what is happening that I didn't already know about?" Aggregate everything; surface broadly.
2. **Curation / notification** — "tell me when something I'd actually go to is coming up." Driven by user-stated interests (tags, categories, venues — model TBD).
3. **Calendar** — "let me plan." Browse by date, week, month.

**Local-flavoured design implications:**
- Default view = "this week" / "this weekend", not date-range trip planning.
- Surface recurring favourites (Wednesday Night Racing, Harbour Nights, Snorkel Park nightlife) prominently.
- Venue browsing matters (locals already know the venues).
- Visitor concerns like "near my hotel" filters are explicitly out of scope for v1.

## 3. v1 scope

### In scope
- Scraping the **6 must-have sources** (see source plan, §5).
- Two scraper archetypes built well: WordPress + The Events Calendar (REST/JSON) and PTIX (headless browser).
- LLM extraction pipeline for Bernews.
- **Entity resolution / dedupe** (title fuzzy-match + date + venue). This is v1 scope, not v2 cleanup.
- Public web surface showing the unfiltered firehose (anonymous users).
- Logged-in user accounts with stated interests (tags/categories/venues).
- Personalised feed for logged-in users based on stated interests.
- **Notification stand-in for v1:** email digest OR in-app feed (decision pending — see §7).
- Polite-crawler infrastructure (rate limits, UA spoofing where needed, robots.txt compliance).

### Out of scope (deferred to later phases)
- User-submitted events.
- Partner feeds / official data sharing agreements.
- Editorial curation layer.
- Native mobile app.
- Push notifications via mobile push or WhatsApp/Telegram bot (end-state target, but not v1).
- Visitor-specific features (near-hotel filters, trip-date planning, multilingual).
- The 5 nice-to-have sources (Events Bermuda, City of Hamilton, RHADC, Cricket Board, BSoA).
- Facebook events scraping (skip indefinitely — hostile platform, legally thin ice).

## 4. Architecture sketch

```
┌─────────────────┐    ┌──────────────────┐    ┌────────────────┐    ┌────────────┐
│ Source adapters │ →  │ Normaliser       │ →  │ Dedupe / ER    │ →  │ Event DB   │
│                 │    │ (canonical event │    │ (fuzzy match:  │    │            │
│ - WP+Tribe REST │    │  schema)         │    │  title+date+   │    │            │
│ - PTIX headless │    │                  │    │  venue)        │    │            │
│ - Bernews LLM   │    │                  │    │                │    │            │
│ - BTA HTML      │    │                  │    │                │    │            │
│ - Chamber HTML  │    │                  │    │                │    │            │
│ - Snorkel Park  │    │                  │    │                │    │            │
└─────────────────┘    └──────────────────┘    └────────────────┘    └─────┬──────┘
                                                                          │
                          ┌───────────────────────────────────────────────┘
                          ▼
                    ┌─────────────────┐    ┌──────────────────┐
                    │ Public API      │ →  │ Surfaces         │
                    │ (anon firehose, │    │ - Web (v1)       │
                    │  auth interests)│    │ - Email digest / │
                    │                 │    │   in-app feed    │
                    │                 │    │ - Push (v2+)     │
                    └─────────────────┘    └──────────────────┘
```

**Load-bearing components:**

- **Source adapters** — pluggable; two archetypes cover most sources (Tribe REST handles BUEI, Events Bermuda, likely RHADC; PTIX adapter is bespoke).
- **Polite fetcher** — rate-limited, browser-UA-spoofing, respects per-source crawl-delay (Bernews enforces 3s).
- **Normaliser** — canonical event schema: title, start/end, venue, address, category, source, source-id, source-url, image, ticket-url, description, scraped-at.
- **Dedupe / entity resolution** — fuzzy-match on (title, date, venue). Keep one canonical record per real-world event; preserve source links for provenance.
- **Public API** — `/events` (anon firehose) and `/feed` (auth, filtered by user interests).
- **Notification dispatcher** — v1 stand-in (email or in-app); architected for swap-in of push / WhatsApp / Telegram later.

**Crawl cadence (rough):**
- Daily: PTIX, Bernews, Snorkel Park.
- Weekly: BUEI, BTA, Chamber.

## 5. Source plan

See the ranked-shortlist artifact for full per-source detail:
`.daveclaw/research/bermuda-event-scraping-sources-survey-ranking/bermuda-event-scraping-sources-ranked-shortlist-v1.md`

**v1 must-have (build all six):** PTIX, BTA/gotobermuda, Bernews, BUEI, Snorkel Park Beach, Bermuda Chamber.

**Deferred to v2+:** Events Bermuda (sports), City of Hamilton, RHADC, Cricket Board (PDF), BSoA, BNG, gov.bm calendar, hotels, Earl Cameron Theatre, BFA.

**Skipped:** Facebook events, Bernews-syndicated lists (captured transitively).

**Engineering takeaways embedded in the plan:**
1. Two adapter archetypes cover ~80% of yield.
2. Dedupe is the real product, not a polish-phase task.
3. Bernews needs LLM extraction, not DOM parsing — budget model cost.
4. PTIX 403s default UAs; headless browser + UA spoofing needed.
5. whatson.bm is the real competitive yardstick — beatable on comprehensiveness.

## 6. Personalisation & notifications

**Anonymous users:** see the full comprehensive firehose. No filtering, no login wall.

**Logged-in users:**
- Declare interests via tags / categories / venues (exact model TBD — see §7).
- Personalised feed filters the firehose to matching events.
- Receive notifications for new matching events.

**Notification delivery — phased:**

| Phase | Channel | Notes |
|-------|---------|-------|
| v1 | Email digest **or** in-app feed | Decision pending; favour whichever is cheapest to ship and proves the personalisation engine |
| v2+ | Mobile push **or** WhatsApp/Telegram bot | End-state target; choice between native app vs messaging-bot still open |

The interest model and the notification engine should be designed so the delivery channel is swappable — that is the v1→v2 hinge.

## 7. Open questions

These should be answered before — or during — plan-doc construction:

1. **v1 notification stand-in:** email digest vs in-app feed. (Email = lower friction, push to inbox; in-app = retains users on the surface but requires them to revisit.)
2. **End-state notification channel:** native mobile app push vs WhatsApp/Telegram bot. (App = higher build cost + stores; bot = lower build cost, lives in messaging surfaces locals already use.)
3. **Interest model:** tags only? categories + venues? free-text keyword follow? Affects schema and onboarding UX.
4. **Auth approach:** email/password? magic-link? social? Affects compliance and friction.
5. **Stack choice:** Next.js + Postgres? Something else? whatson.bm uses Next.js + Supabase — choosing the same stack would let us read their data shape if a partnership ever opens.
6. **Partnership posture with whatson.bm:** treat as competition (compete on comprehensiveness) or seek data partnership (mutual import)? Affects scrape ethics of their tiny dataset.
7. **Image handling:** rehost (storage cost) or hot-link (broken-image risk)?
8. **PTIX legal risk:** robots.txt permits public-event scraping but ToS not yet reviewed. Verify before launch.
9. **Hosting / ops:** where does it live, what's the deploy story, what's the budget for headless-browser + LLM compute?

## 8. Phasing sketch

**v1 — comprehensive firehose + basic personalisation** (this spec)
- 6 must-have scrapers live
- Dedupe layer working
- Anonymous firehose web view
- Logged-in interests + filtered feed
- Email digest OR in-app feed as notification stand-in

**v2 — push-native delivery**
- Native mobile app **or** WhatsApp/Telegram bot
- Replace v1 notification stand-in
- Polish UX based on v1 usage patterns

**v3 — coverage breadth + community**
- Add nice-to-have sources (Events Bermuda, City of Hamilton, RHADC, etc.)
- User-submitted events
- Editorial curation layer
- Visitor-targeted features (if demand emerges)

---

## Appendix: source materials

- Brainstorming package: `pkg-1ed9b488-9e9b-4f60-a927-f94e1fcf6495`
- Research package: `pkg-556813a0-7c5f-4833-80d8-a6c09d77b80b`
- Ranked source shortlist: `.daveclaw/research/bermuda-event-scraping-sources-survey-ranking/bermuda-event-scraping-sources-ranked-shortlist-v1.md`
- Known blocker on draft-from-idea: `.daveclaw/incidents/2026-05-23-draft-from-idea-stack-validation.md` (stack param ignored by planner — not blocking spec authoring, but will need a workaround when promoting to a Daveclaw plan)
