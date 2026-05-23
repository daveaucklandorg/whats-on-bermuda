# Bermuda Events Aggregator — Source Shortlist (v1 scraper-first)

## Pre-existing competition / aggregators worth knowing about first

- **whatson.bm** — *direct competitor*. Next.js + Supabase stack, user-submittable, ~22 individual + 4 recurring events live (May–Dec 2026), tagged (sports, concert, party, drinks, arts, dining, history, fitness, fundraising, cruise, tour, wine). Active and modern. Worth treating as a benchmark and a candidate for partnership/import — not a scrape target (small dataset, ToS unclear). https://whatson.bm/
- **bermuda.com/things-to-do/events-calendar/** — same operator as the old **nothingtodoinbermuda.com** (301 redirect confirmed). This same calendar is also embedded as the official **Royal Gazette** calendar ("powered by Nothing to Do in Bermuda"). So bermuda.com IS the de facto incumbent aggregator. Submission-driven; structure couldn't be fully verified — the actual listings render dynamically. **Needs deeper inspection.** https://www.bermuda.com/things-to-do/events-calendar/ , https://www.royalgazette.com/calendar/
- **eventsbermuda.com** — narrow vertical (athletics only: cycling/running/swim/tri/golf/etc.), WordPress + The Events Calendar plugin, 40+ organisers feeding it. Easy scrape. https://eventsbermuda.com/

## MUST-HAVE for v1

### 1. PTIX (Premier Tickets) — ptix.bm
- **URL:** https://www.ptix.bm/Category/Events , https://www.ptix.bm/Calendar
- **Coverage:** Concerts, theatre, galas, fundraisers, sport, festivals, movies, community, parties — anything with paid tickets in Bermuda flows through here. 10,000+ events lifetime per their own marketing.
- **Cadence:** Daily — it's a live ticketing platform.
- **Structure:** ASP.NET site (.aspx, AdminPanel/, App_Code/ in robots.txt). `/Calendar` returned 403 to WebFetch — likely UA/bot filtering, **needs deeper inspection** with a real browser/headless. Expect HTML listing pages plus per-event detail pages; no public API mentioned.
- **Robots/ToS:** robots.txt only blocks admin/login/checkout paths — public event pages are fair game for crawlers. ToS not reviewed; verify before launch.
- **Duplication:** High canonical authority — PTIX is the source-of-truth for ticketed events. Other sites (BTA, Bernews) link out to it.
- **Verdict:** **Must-have.** The single highest-yield source on the island. Budget engineering time for headless rendering + anti-bot tolerance.

### 2. Bermuda Tourism Authority — gotobermuda.com
- **URL:** https://www.gotobermuda.com/things-to-do/events , detail pattern `/event/[slug]/[id]` (e.g. `/event/cup-match-cricket-classic/3538`)
- **Coverage:** Curated "signature" events — Harbour Nights, Bermuda Day, Carnival, Cup Match, Restaurant Weeks, Kite Festival, festivals, museums, signature sports. Quality > quantity (~4–20 cards visible at any time).
- **Cadence:** Slow/curated — weeks to months.
- **Structure:** Server-rendered HTML, clean card layout, detail pages expose date/time/address/phone/description/ticket link as plain text. **No JSON-LD** found — would have made life easier. Predictable URL pattern.
- **Robots/ToS:** robots.txt permissive on public content; blocks admin/search only.
- **Duplication:** Almost everything here also appears on PTIX, City of Hamilton, or venues. Use BTA mainly to enrich (canonical descriptions, images) rather than as primary feed.
- **Verdict:** **Must-have** for enrichment and signature-event canonical metadata.

### 3. Bernews (tag-based) — bernews.com
- **URL:** No /events/ section exists. Use tag archives: e.g. `/tag/charityevents/` (4,685 posts), `/tag/seminarsinbermuda/` (1,035), plus periodic posts like "City Announces 2026 Events Calendar" and the recurring "Weekend Reports". https://bernews.com/
- **Coverage:** Broadest editorial coverage of events on the island — but unstructured (announcements buried in news articles).
- **Cadence:** Multiple posts daily.
- **Structure:** Standard WordPress. No Events Calendar plugin, no JSON-LD events. Will need NER/LLM extraction from prose. 3-second crawl-delay enforced; Googlebot partially blocked, several SEO scrapers fully blocked — be a polite crawler.
- **Robots/ToS:** Restrictive on aggressive bots, otherwise OK. 3s delay mandatory.
- **Duplication:** High — Bernews announces events that originate elsewhere. Use as a discovery feed pointing to canonical sources.
- **Verdict:** **Must-have**, but as an LLM-extraction pipeline, not a structured scrape.

### 4. BUEI — buei.bm
- **URL:** https://buei.bm/events/
- **Coverage:** Lectures, film nights, kids' programming, sunset cruises, glow worm tours, Harborside Market. Single venue but high-volume programming.
- **Cadence:** Weekly.
- **Structure:** WordPress + **The Events Calendar (Tribe)** plugin confirmed. Standard URLs (`/events/list/`, `/event/[slug]/`) and very likely the JSON REST endpoint `/wp-json/tribe/events/v1/events` is open (default behaviour). **Verify endpoint** but if open this is a clean JSON scrape — trivial.
- **Robots/ToS:** Standard WP, no special restrictions expected.
- **Duplication:** Low — most BUEI events appear only here and on PTIX (when ticketed).
- **Verdict:** **Must-have.** Cheapest, highest-ROI venue scrape on the island.

### 5. Snorkel Park Beach — snorkelparkbeach.com
- **URL:** https://snorkelparkbeach.com/events/ , https://snorkelparkbeach.com/events-flyers/calendar-of-events/9-upcoming-events/
- **Coverage:** Nightlife, beach parties, club nights, Carnival/Cup Match weekend events, family daytime events. The dominant nightlife venue.
- **Cadence:** Weekly, seasonal peak May–Oct.
- **Structure:** Dedicated `/events/` URL exists. **Needs deeper inspection** — likely WordPress + flyer-image-heavy listings (the URL hints at "events-flyers"); may need OCR or careful metadata extraction.
- **Verdict:** **Must-have** — nightlife is otherwise underserved by BTA/Bernews.

### 6. Bermuda Chamber of Commerce — bermudachamber.bm
- **URL:** https://www.bermudachamber.bm/events/calendar
- **Coverage:** Networking, Chamber events, community, educational, festivals, Harbour Nights (they host it).
- **Cadence:** Weekly.
- **Structure:** **GrowthZone/ChamberMaster** platform — well-known SaaS with predictable URL patterns (`/events/details/[slug]`). Server-rendered. Volume appears modest (~2 events in observed month) but always includes Harbour Nights and similar canonical entries.
- **Verdict:** **Must-have** — small but reliable, fills the business/community gap.

## NICE-TO-HAVE

### 7. Events Bermuda — eventsbermuda.com
- WordPress + The Events Calendar; sports-only. Either scrape or seek data partnership. Trivial via Tribe REST. **Nice-to-have** because category is narrow and already partly covered by PTIX. https://eventsbermuda.com/

### 8. City of Hamilton — cityofhamilton.bm
- `/events/index.php` and `/calendar.php` on the **Revize** government CMS. Featured-event style, lightly structured, likely few entries beyond signature events. **Nice-to-have** for municipal authority. https://www.cityofhamilton.bm/events/index.php

### 9. RHADC — rhadc.bm
- Has `/events-calendar/` and `/sailing/regattas/`. Site timed out under WebFetch — **needs deeper inspection** for platform. Likely WordPress. Wednesday Night Racing + regattas. **Nice-to-have** for sailing coverage. https://www.rhadc.bm/events-calendar/
- Pair with **Royal Bermuda Yacht Club** https://www.rbyc.bm/on-the-water/domestic-racing for full sailing picture.

### 10. Bermuda Cricket Board — cricketbermuda.com
- Schedule page is just a **PDF link** (e.g. "2025 T20 Schedule 22.4.25"). **Nice-to-have** but requires PDF parsing + likely yearly manual mapping. Cup Match itself is easier to source from BTA/Bernews. https://cricketbermuda.com/schedule/

### 11. Bermuda Society of Arts — bsoa.bm
- WordPress, custom post types, last visible exhibition was 2023 in the indexed page. Updates sporadic. **Nice-to-have**, low ROI. https://bsoa.bm/upcoming-exhibitions/

## DEFER

### 12. Bermuda National Gallery — bng.bm
- Exhibition-driven, very low frequency. Not verified structurally. Defer until v1 stable. https://bng.bm/

### 13. Gov.bm National Events Calendar 2026
- Currently a **submission portal only** (Google Form behind it), no live listings. Watch for relaunch; if it ever publishes structured data it becomes a must-have. **Defer.** https://www.gov.bm/bermuda-national-events-calendar-2026

### 14. Hamilton Princess / Fairmont Southampton
- No public event-calendar pages found — events live inside hotel marketing pages and seasonal package promos. Fairmont Southampton is mid-renovation (reopening phased). **Defer.** https://www.thehamiltonprincess.com/ , https://fairmontsouthamptonbermuda.com/

### 15. Earl Cameron Theatre / City Hall Arts Centre
- No standalone events page; bookings show up on PTIX (ticketed) and City of Hamilton calendar. Covered transitively. **Defer.** https://www.cityofhamilton.bm/explore_the_city/city_hall_arts_center.php

### 16. Bermuda Football Association — bermudafa.com
- Webflow site; nav has Premier/First/U23 divisions but no centralised fixtures list rendered on the homepage. Fixture announcements show up faster on Bernews. Revisit if a fixtures page is discovered. **Defer.** https://www.bermudafa.com/

## SKIP

### 17. Facebook events
- Meta actively hostile: extreme rate limits, advanced fingerprinting, regular ToS escalation. Third-party tools (Apify, Bright Data) exist but require residential proxies, are expensive, and put you on legal thin ice for a consumer product. **Skip for v1.** Revisit only if a sanctioned partner channel opens, or for a tiny manual-list of priority venue pages via official Graph API with page-owner OAuth.

### 18. Bernews-syndicated lists ("Heritage Month Calendar", "City Announces 2026 Events")
- These are derivative articles republishing other calendars. **Skip** as distinct sources — they'll be captured via your Bernews tag pipeline and would only inflate duplication.

---

## Engineering takeaways

1. **Two scraper archetypes will cover ~80% of yield:** (a) WordPress + The Events Calendar → REST/JSON; (b) PTIX ASP.NET → headless browser + careful pagination. Build those two adapters well.
2. **Dedupe is the real product.** Almost every signature event will arrive via BTA, PTIX, Bernews, the Chamber, and the venue itself. Plan an entity-resolution layer from day one (title fuzzy-match + date + venue).
3. **Bernews requires LLM extraction**, not DOM parsing. Plan model cost.
4. **Polite crawling:** Bernews enforces 3s delay; PTIX 403s default UAs. Build a rate-limited, browser-UA-spoofing fetcher early.
5. **whatson.bm is your real competitive yardstick**, not bermuda.com. They're modern, small, and beatable on comprehensiveness if your scraper coverage is honest.
