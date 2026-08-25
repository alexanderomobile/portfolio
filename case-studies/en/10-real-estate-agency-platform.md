🇷🇺 [Русский](../ru/10-real-estate-agency-platform.md) · 🇬🇧 [English](../en/10-real-estate-agency-platform.md) · 🇪🇸 [Español](../es/10-real-estate-agency-platform.md)

# Case Study: Real Estate Agency Platform

**Status:** ✅ Production

---

## Problem

An estate agency on the Red Sea coast sold property through chat apps and folders of photographs. A website existed on paper: a bought theme still full of its demo content — an address in Miami, six hundred invented listings, an enquiry form that mailed the theme's author. Not one real property on it.

One pass had to deliver all of it: move everything onto owned infrastructure, build a real catalogue out of raw material, bring the site to a state the advertising platforms will accept for review — and leave a way to keep developing without using the live site as the workbench.

One more constraint: the agency sells to buyers from four language groups, Arabic among them with right-to-left text, and negotiates in three currencies.

## What it does

**Catalogue.** 96 listings, 552 photographs, 17 videos, a map point on every one. Gallery, video with a poster frame, attributes, price. The owner edits all of it in the ordinary admin screens, with no developer involved.

**Search.** Filters by district, type, status, rooms, area and price, plus a dedicated "everything on the map" page with clustering and a list that moves with it.

**Four languages.** Interface, legal pages, mail and labels in English, Russian, German and Arabic, with a full RTL layout. Listings themselves are deliberately not translated: one catalogue serves all four versions.

**Multi-currency.** One base currency, rates refreshed daily, the visitor reads prices in their own. Each listing carries the caveat that a converted figure is indicative and the contract is written in the base currency.

**Contact channels.** Phone, mail and three messengers — in the header, the footer, the listing page and the share menu, with links that work in every language.

**Analytics behind consent.** Not one third-party request leaves the browser before the visitor answers the banner. In parallel, visits are counted from the web server's own log — no cookie, nothing shipped off the machine.

**A staging copy.** A full copy of the site on its own subdomain, with its own database, behind a password, closed to search engines, with mail switched off — so changes are checked against live data yet cannot be indexed or write to a client.

## Business value

The agency got a shopfront it can show a buyer and submit for platform review: a real catalogue instead of a demo one, contacts that are actually answered, and legal pages that match what the site really does. The catalogue is maintained in-house. The infrastructure is owned outright — no monthly hosting bill, no dependence on someone else's mail service, and not a single inbound port open to the internet.

## Stack

**Platform.** Proxmox, one container per role: web, database, mail, tunnel, tooling. Daily snapshots, restore tested.

**Web.** Apache with mod_php, PHP 8.3, a CMS with an industry theme. All development lives in a child theme and a purpose-built plugin, so the parent stays updatable.

**Perimeter.** The server opens the connection outwards through a tunnel; there are no inbound ports at all. TLS and DNS at the edge node, and the routing config is validated before it is applied — a mistake in that one file takes down every site on the node at once.

**Mail.** The ISP blocks port 25 and the host sits behind NAT. Incoming mail arrives at the edge MX, a handler signs it with a shared secret and posts it over HTTPS through the same tunnel to a receiver; without the secret the receiver answers 403. From there: Maildir, IMAP, webmail. Outgoing goes through a transactional provider, DKIM-signed.

**Data.** MariaDB on a separate node, reachable only through two independent firewall layers — one inside the container, one at the hypervisor.

**Material processing.** Python: Pillow for photographs and documents, ffmpeg for frames pulled out of video, import scripts in PHP through the platform's CLI.

## Engineering decisions

**The catalogue was built from whatever existed.** The source was 53 folders: photographs mixed with video, prices buried in text files and post captions, and half the folders holding two or three different properties at once. The parse ran as a fan of independent passes — each reads its own folder, separates the objects, and pulls out type, area, rooms, district and price. The traps were named up front: a deposit is not the price, a monthly rent is not a sale, a price per square metre is not the price of the property. The result was 96 listings instead of 24, and not one invented attribute.

**Photographs were made safe before publication.** Images were resized, EXIF stripped wholesale — otherwise the camera's geotag leaves the building along with a stranger's address — and watermarked. Listings with no stills got a cover pulled from their video a quarter of the way in: not the first second, where the camera is still shaking.

**The gallery was stored in a shape nothing reads.** The theme reads a gallery as *several rows* of a meta field; the import had written one row holding a comma-separated list — so a listing with 28 photographs displayed exactly one. Diagnosis was harder than the fix: the normal meta read passes through a compatibility filter that casts the value to an integer and returns the first id, so from the outside it looked as though the database really did hold a single image. The table had to be read directly.

**A listing is not language-specific content.** The multilingual plugin treated listings as translatable, all 96 carried the English language term, and in three of the four versions the catalogue, every archive and the map came out empty. Unticking the box in the plugin's settings was not enough: the parent theme declares its own types translatable in its own config file, and the plugin obeys the file over the settings. The lists are filtered out in the child theme, at a priority that is certain to run last.

**A hand-written translation layer instead of the shipped one.** The catalogues that come with the theme are roughly half machine-translated: "Currency Switcher" turns into something no native speaker would write, and printf placeholders are mangled into "% 2 $ s" — a translation like that damages a page more than English would. What replaced it is 253 strings per language covering four different sources of labels at once: the theme's own strings, the values stored in its settings, taxonomy term names, and specific post meta.

**When a translation is applied matters more than the dictionary.** The first version of that layer hooked the option read — which means it ran during bootstrap, before the page's language was known. Its fallback was the engine's own locale, and that equalled the language the owner reads the admin in. English listing pages came out with Russian headings, and helper scripts that read and saved settings then wrote the wrong language into the database as if it were the original. Diagnosis took an evening; recovery came from reversing the project's own dictionaries and reading defaults out of the theme. The substitution now runs once the page language is settled. The lesson was worth writing down: a translation has a dictionary *and* a moment.

**A `meta_key` is an INNER JOIN.** The catalogue sorts "featured first", and the theme implements that with a `meta_key`. WordPress turns a `meta_key` into an inner join — a listing without that row does not sort last, it disappears from the result set. 15 of 96 had no such row: the city archive said "no listings found" while the term counted three, and the status archive showed 75 of 90. The symptom looked like an indexing problem; the cause was a missing field.

**Price search only speaks the base currency.** The theme converts the price slider's bounds for display and then submits those converted numbers to a query that compares them against prices stored in the base currency. With dollars selected, search honestly found nothing. The bounds are now converted back before the query runs, using the same daily rates.

**A promise in the copy is part of the code.** The privacy policy said in plain words that nothing is measured in the visitor's browser, and that the text would be updated *before* that changed. Adding analytics called that promise in: eight legal pages across four languages were rewritten in the same change that switched the counter on.

**Fonts brought in-house.** Every page — including the ones a visitor sees before answering the banner — asked a third-party CDN for its fonts and handed it the visitor's IP address on the way. 38 files now live on our own server, and references to the external source are not rewritten but dropped at the point stylesheets are registered, so a plugin added later cannot quietly reopen the channel.

**QA was built as an adversarial harness.** Readiness was not checked by clicking around. Independent reviewers ran per dimension — galleries, currency, languages, analytics, SEO, contacts — and every claimed finding was handed to a separate sceptic whose job was to refute it. Only what was reproduced counted. Across two rounds: 40 confirmed defects fixed, 4 rejected as non-defects — including a "truncated description" that turned out to be the template working exactly as designed.

## Functional blocks

Catalogue · Media pipeline · Search and map · Multilingual layer · Multi-currency · Contact channels · Consent and analytics · Legal surface · Perimeter and mail · Staging copy · Adversarial QA

## Metrics

| metric | value |
|---|---|
| Listings published | **96** from 53 folders of raw material |
| Photographs · videos · map points | 552 · 17 · 96 |
| Interface languages | **4**, RTL included |
| Inbound ports open | **0** |
| Third-party requests before consent | **0** |
| Defects found and fixed by the harness | **40** confirmed, 4 rejected |
| Listings dropping out of results before the fix | 15 of 96 |
| Archives returning an empty list | 3 |
| Web font files moved in-house | 38 |
| Sitemap URLs, all answering 200 | 165 |

## Roadmap

This section describes what is **designed but not yet built**.

**Voice-driven property search in a chat app.** A buyer speaks a request — "a two-bedroom in this district under a hundred thousand dollars" — and gets a shortlist back. The pipeline: voice message → speech recognition in four languages → an LLM fills the query slots (budget and currency, district, type, rooms, sale or rent) → a call into the catalogue through the REST layer that already exists → a reply of cards with a photograph, the price in the currency the buyer named, and a map link → a spoken answer synthesised back. Its own container, webhook through the same tunnel, still zero inbound ports. Two decisions are settled in advance: on an ambiguous request the bot asks one clarifying question rather than guessing, and prices convert through the same daily rates as the website, so bot and shopfront can never disagree on a number.

**Saved searches and alerts.** Subscribe to a set of criteria and be told, by mail or message, when something matching appears.

**Lead pipeline.** Enquiries from forms and messengers in one queue with states, instead of a conversation smeared across four apps.

**Messenger business API integration.** Template messages and viewing confirmations; requires completed platform verification, which is in progress.

**Photo scoring and de-duplication.** Automatically drop duplicates and obviously poor frames as a new folder is ingested.

**Continuous checks on the child theme.** Linting, type checks and smoke tests of the key pages on every change — today the adversarial harness fills that role, run by hand.

**Content-Security-Policy.** Deliberately not set yet: on this combination of theme and visual editor it cannot be switched on blind, and it needs its own pass with every screen verified.

## Screenshots

![Architecture](../../assets/real-estate-platform-diagram.svg)

## Run

The infrastructure runs on an owned Proxmox node. Configuration, addresses and credentials are not published: the system serves a live business.

---

[← Back to portfolio](https://github.com/alexanderomobile/portfolio/blob/main/README.md)
