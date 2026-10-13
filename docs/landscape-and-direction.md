# Seasonal: landscape and proposed direction

_Discussion draft, 2026-10-13. Nothing built yet; no repository attached._

## 0. Decisions so far (Chris, 2026-10-13)

1. **Start in Europe, with Switzerland first.** Grown in Switzerland is good; grown in the shopper's own canton is better.
2. **"Local" means within 400 miles (about 640 km).** From central Switzerland that radius reaches much of France, Germany, Austria and northern Italy. So the app uses a ladder: same canton > Switzerland > within 640 km > further away.
3. **Flag air-freighted and heated-greenhouse items: yes.**
4. **Form factor: web app (PWA), as recommended.** One codebase, installable, works offline, no app-store gatekeeping, and easy de/fr/it/en localisation.

### Switzerland-specific notes (to verify while building)
- Seasonality sources to curate from, with attribution: the [Swiss Farmers seasonal calendar](https://www.swiss-farmers.ch/seasonal-calendar/), the [WWF Switzerland Saisontabelle](https://www.wwf.ch/sites/default/files/doc-2017-08/2014-01-saisontabelle.pdf) and the [Ebenrain (BL) Saisonkalender](https://www.baselland.ch/politik-und-behorden/direktionen/volkswirtschafts-und-gesundheitsdirektion/ebenrain/ernaehrung/wissen/saisonkalender). None looks openly licensed, so we hand-curate facts and cite them rather than copy tables.
- Swiss law requires country of origin on fresh produce. In-store "local" signals: Suisse Garantie, Bio Suisse (Knospe), regional brands (regio.garantie labels, Migros "Aus der Region. Für die Region.", Coop "Miini Region").
- An air-freight declaration rule ("per Flugzeug transportiert") was the subject of a federal consultation in 2023-24 (see e.g. the [Swiss Farmers' Union response](https://www.sbv-usp.ch/fileadmin/sbvuspch/04_Medien/Stellungnahmen/Stellungnahmen_2023/240103_Stellungnahme_Deklaration_Flugtransporte.pdf)). Its current status needs checking, since a legal label would make air-freight flagging much more reliable.
- The region model becomes country > canton, with a distance radius on top. US states can follow later.

## 1. The problem, restated

At the shelf, a shopper wants to answer two questions fast:
1. **Is this in season where I live right now?**
2. **Was this grown near me (and how was it grown)?**

Question 1 is answerable with open data today. Question 2 is the hard part, because the origin information mostly lives on a sticker or a sign in the store and rarely in any machine-readable feed.

## 2. What already exists

| Thing | What it does | Gap |
|---|---|---|
| [Seasonal Food Guide](https://foodprint.org/eating-sustainably/seasonal-food-guide/) (FoodPrint / GRACE) | US state-by-state, half-month seasonality for ~140 items; iOS/Android app | Not open data/licence-clear; knows nothing about what a store actually stocks |
| [Seasonal Foods Calendar](https://f-droid.org/packages/flunzmas.seasoncalendar/) (F-Droid) | Open-source Android seasonal calendar | Coarse regions (mostly European countries), no origin info |
| [Open Food Facts](https://world.openfoodfacts.org/) + Eco-Score/Green-Score | Open database of packaged products, origins of ingredients, environmental score | Built around barcodes; loose produce has no barcode, only PLU codes |
| USDA [Local Food Directories](https://www.usdalocalfoodportal.com/) | Farmers markets, CSAs, on-farm markets, food hubs, with an API | Not grocery stores |
| USDA AMS Market News | Terminal-market and shipping-point reports: which commodities are moving from which origin, weekly | Wholesale level; not per store, but a good proxy for "what's local vs. shipped in right now" |

Takeaway: seasonality calendars are a solved-ish but fragmented problem, and nobody has open, grocery-store-level origin data.

## 3. How origin actually shows up in a US store

- **Country of Origin Labeling (COOL)** ([USDA AMS FAQ](https://www.ams.usda.gov/node/13237)): retailers must label fresh and frozen fruits and vegetables with country of origin (sign, sticker, or package). Country only, not state or farm. Processed/mixed items, restaurants and small retailers are exempt.
- **PLU codes** (IFPS): 4 digits = conventional, 5 digits starting with 9 = organic. They identify commodity/variety/size, **not origin**. Useful for "is this organic" and for identifying the item, not for "local".
- **State "grown in" programs** (e.g. Jersey Fresh, Pride of NY, Georgia Grown, Texas Grown): voluntary marks on signs and packaging. These are the most practical "local" signal in-store.
- **Packaged produce** has a UPC, often with a packer/grower address on it, and sometimes a GS1 traceability code (PTI) that encodes the grower.

So in practice "local" has to be inferred from a mix of: the sign/sticker the shopper sees, the packer address, state marks, and seasonality plus wholesale shipping data.

## 4. Retailer data availability

- **Kroger** has a public developer API (products, prices, store locations), but product records don't carry reliable origin.
- **Instacart/Walmart/others**: partner-only APIs or scraping (ToS risk). Online listings occasionally show "Product of Mexico" text, inconsistently.
- **Co-ops and independent grocers** are the realistic partners: they care about local, often label farm names, and are small enough to share a weekly list.

Conclusion: don't build v1 on retailer feeds. Build it so retailer feeds can plug in later.

## 5. A framing caution worth deciding on early

"Local" is not automatically "lower impact". Lifecycle studies (e.g. Poore & Nemecek, Science 2018) find transport is usually a small share of a food's footprint; what is grown and how (and whether it was air-freighted or grown in a heated greenhouse) matters more. Out-of-season local tomatoes from a heated greenhouse can beat or lose to field-grown imports depending on energy source.

Suggestion: the app's core signal is **"in season here, grown in the open, not air-freighted"**, with local as a strong positive and organic as a separate, clearly labelled attribute. That keeps the app honest and still pushes toward local seasonal eating.

## 6. Proposed product (v1)

**A mobile-first, offline-capable web app (PWA) plus an open dataset.**

1. **"What's in season near me"**: pick a location (or use GPS), see a list for this half-month: peak, available, storage crop, and out of season. Each item has a short note on what to look for.
2. **Shelf check**: type or scan an item (PLU from the sticker, a UPC barcode, or just "tomatoes"), optionally tap the origin printed on the sign (country / state / farm). The app answers: in season here? likely local? organic? plus a one-line why.
3. **Where to buy local**: nearby farmers markets, CSAs, farm stands, food co-ops (from USDA Local Food Directories and OpenStreetMap).
4. **Community sightings (later)**: shoppers report "Store X has local NJ peaches this week". This is the only scalable path to store-level origin data without retailers.

## 7. Proposed architecture

```
 seasonality-data/ (open dataset, CC BY-SA, YAML/CSV in git)
   crops.yaml        canonical produce list, PLU + synonyms
   regions/          seasonality per region (US state first; climate zone fallback)
   sources.md        provenance per row
        |
        v
 build step -> versioned JSON bundles per region (static, CDN-hosted)
        |
        v
 PWA client (TypeScript; works offline from the bundle)
   - season view, shelf check, PLU/UPC lookup
   - calls optional APIs:
       USDA Local Food Directories, OSM Overpass  -> where to buy
       Open Food Facts                            -> packaged item origin
       (later) small sightings API                -> community reports
```

Why this shape:
- **Data as the product.** A clean, openly licensed, git-versioned seasonality dataset is the thing nobody has, and other apps can reuse it. Contributions are PRs.
- **Static first.** No backend needed for v1, so it is cheap to host and easy to fork for another country.
- **Region model**: start with US states (matches existing guides and state "grown" programs), but key the data so a region can be a country, state, or climate zone, so Europe/UK can be added by contributors.
- **Pluggable "origin signals"**: each signal (user-entered origin, PLU organic flag, UPC lookup, state mark, sightings, a retailer feed) contributes to a "likely local" verdict with an explanation, rather than a black-box score.

## 8. Open questions for Chris

1. **Where first?** US only (state-level), a single state/metro as a pilot, or something else (UK/EU)?
2. **What does "local" mean** for the app: same state, within ~250 miles, or same country?
3. **How opinionated on impact?** Pure "seasonal + local" signal, or also flag air-freight and heated-greenhouse items (my recommendation)?
4. **Organic**: a first-class filter, or a secondary badge?
5. **Form factor**: PWA (recommended, one codebase, no app-store friction) vs. native apps?
6. **Community sightings**: in v1, or wait until the core works?
7. **Licence**: MIT/Apache for code plus CC BY-SA or ODbL for data (recommended), or something else?
8. **Bootstrapping data**: hand-curate from extension-service calendars (slower, licence-clean) vs. try to license the FoodPrint dataset?
