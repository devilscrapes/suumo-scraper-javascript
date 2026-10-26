<p align="center"><a href="https://apify.com/DevilScrapes/suumo-japan-real-estate"><img src="assets/banner.svg" alt="Suumo.jp Japan Real Estate Scraper — JavaScript example" width="100%"></a></p>

# Suumo Scraper for JavaScript and Node.js: Japan property listings in your app or API

<p align="center">
<a href="https://apify.com/DevilScrapes/suumo-japan-real-estate"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/suumo-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/suumo-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Use this repository to call the **[Suumo.jp Japan Real Estate Scraper](https://apify.com/DevilScrapes/suumo-japan-real-estate)** from Node.js. Your code passes a suumo.jp search address, chintai or bukken, and gets back JavaScript objects with typed values: integer yen for rent, sale price and fees, a madori string, floor area as a number and an array of nearby stations with walk minutes. Apify runs the crawl and pagination remotely, so a backend route, a queue consumer or a scheduled function only starts the run and reads the dataset. From the command line, `index.js` writes the same rows to `results.json` and `results.csv`.

> **Actor:** [Suumo.jp Japan Real Estate Scraper](https://apify.com/DevilScrapes/suumo-japan-real-estate) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [suumo-scraper-python](https://github.com/devilscrapes/suumo-scraper-python)

## 🔥 What you get

- **Your search, not ours**: the input is a results URL copied from suumo.jp, so every filter Suumo offers (ward, line, rent band, layout, age) is already applied before the Actor reads a page.
- **One row per room, not per building**: Suumo's rental cards group several vacant units under one building; each unit becomes its own row and inherits the building's name, address, type, age and stations.
- **Yen as integers**: display strings such as `6.5万円` arrive as `65000`. A deposit or key money shown as `なし` becomes `0`, while a dash stays `null`, so "none charged" and "not stated" remain different things.
- **Every station on the card**: `stations` is a list of objects with `line`, `station` and `walk_minutes`. Rental cards list up to three, used-condo cards list one.
- **Rentals and used condos with one schema**: a `/jj/chintai/ichiran/` URL is read as rent mode, a `/jj/bukken/ichiran/` URL as buy mode, and `listing_mode` on each row says which.
- **Agency details when you ask for them**: with `fetchListingDetails` on, the Actor opens each listing page for the handling agency's name (plus phone on rentals, and management fee and floor on condos), billed as a separate event only when a name is found.
- **Caps that hold across URLs**: `maxItems` counts rows over all your search URLs together, and `maxPages` limits how far each one paginates. Pages that answer with temporary errors are retried with growing pauses.

## 🎯 Use cases

### Power a rental search for people moving to Japan

Relocation and expat-housing apps can store the rows keyed by `listing_id` and expose filters on rent, upfront cost and walking time. Japanese building names and addresses stay untouched, ready for your translation layer or a geocoder.

### Alert agents to new units under a target rent

A cron container runs the same Suumo URL each morning, diffs `listing_id` values against yesterday and sends the new units below a price threshold to a webhook. Management companies use the same loop to see when a competing building lists cheaper rooms.

### Collect agency contacts for a B2B pipeline

Switch `fetchListingDetails` on for a narrow search and each row carries the handling agency's name, plus a phone number for rentals. Group by `agency_name` in your service to see which firms hold the most stock in a ward.

### Feed a pricing model behind an API

A serverless job refreshes listings per station on a schedule and writes them to your database. Your own endpoint can then answer "what does a 1LDK near this station cost" from fresh rows instead of stale averages.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/suumo-scraper-javascript.git
cd suumo-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [Suumo.jp Japan Real Estate Scraper](https://apify.com/DevilScrapes/suumo-japan-real-estate), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Here is the integration with nothing else around it. The awaited `call()` returns when Suumo's pages have been processed, and `listItems()` returns the units the run stored.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/suumo-japan-real-estate').call({"searchUrls": ["https://suumo.jp/jj/chintai/ichiran/FR301FC001/?ar=030&bs=040&ta=13&sc=13104&page=1"], "maxItems": 10, "maxPages": 1, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` passes `input.json` to the Actor as written. The example is a Shinjuku, Tokyo rental search capped at one page and ten units, so the round trip is quick and cheap. For production, build the search on suumo.jp with your filters, paste the URL, and tune `maxItems` and `maxPages`.

```json
{
  "searchUrls": [
    "https://suumo.jp/jj/chintai/ichiran/FR301FC001/?ar=030&bs=040&ta=13&sc=13104&page=1"
  ],
  "maxItems": 10,
  "maxPages": 1,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `searchUrls` | array | yes | `["https://suumo.jp/jj/chintai/ichiran/FR301FC001/?ar=030&bs=040&ta=13&sc=13104&page=1"]` | — |
| `maxItems` | integer | no | `100` | — |
| `maxPages` | integer | no | `0` | — |
| `fetchListingDetails` | boolean | no | `false` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Each object is a single unit for rent or a single condo for sale, with `listing_mode` set to `rent` or `buy`. Rent, sale price, kanrihi, shikikin and reikin are integers in yen (`null` when Suumo shows nothing usable, `0` when it explicitly says none). `stations` is an array of `{ line, station, walk_minutes }`, `area_sqm` is a float and `building_age_years` an integer. Rows also carry `listing_id`, `url`, `name`, `property_type`, `address`, `layout`, `floor`, `agency_name`, `agency_phone` and `scraped_at`, with the agency fields filled only when detail fetching is on.

| Field | Type | Always present |
|---|---|---|
| `listing_id` | string | no |
| `name` | string or null | no |
| `url` | string | no |
| `property_type` | string or null | no |
| `price_yen` | integer or null | no |
| `management_fee_yen` | integer or null | no |
| `deposit_yen` | integer or null | no |
| `key_money_yen` | integer or null | no |
| `address` | string or null | no |
| `stations` | array | no |
| `listing_mode` | string | no |
| `agency_name` | string or null | no |
| `agency_phone` | string or null | no |
| `layout` | string or null | no |
| `area_sqm` | number or null | no |
| `floor` | string or null | no |
| `building_age_years` | integer or null | no |
| `scraped_at` | string or null | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.00001 |
| Result emitted | $0.002 |
| Detail page enriched | $0.005 |

Prices as configured on 2026-08-12. The [Actor page](https://apify.com/DevilScrapes/suumo-japan-real-estate) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This ES module, built on Node built-ins alone, turns `results.json` into a shortlist for an app: rental units within a walking limit and an upfront budget, sorted by total move-in cost and written to `shortlist.json`.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const MAX_WALK = Number(process.env.MAX_WALK ?? 10);
const BUDGET_YEN = Number(process.env.BUDGET_YEN ?? 500000);

const rows = JSON.parse(await readFile('results.json', 'utf8'));
const yen = (value) => value ?? 0;

const shortlist = rows
    .filter((row) => row.listing_mode === 'rent' && row.price_yen)
    .map((row) => {
        const timed = row.stations.filter((s) => s.walk_minutes !== null);
        const nearest = timed.sort((a, b) => a.walk_minutes - b.walk_minutes)[0];
        const moveIn = row.price_yen + yen(row.management_fee_yen) + yen(row.deposit_yen) + yen(row.key_money_yen);
        return { id: row.listing_id, name: row.name, layout: row.layout, sqm: row.area_sqm,
            station: nearest?.station, walk: nearest?.walk_minutes, rent: row.price_yen, moveIn, url: row.url };
    })
    .filter((unit) => unit.walk !== undefined && unit.walk <= MAX_WALK && unit.moveIn <= BUDGET_YEN)
    .sort((a, b) => a.moveIn - b.moveIn);

await writeFile('shortlist.json', JSON.stringify(shortlist, null, 2));
console.table(shortlist.map(({ url, ...unit }) => unit));
```

## ❓ FAQ

### Does SUUMO have an official API I could use instead?

SUUMO does not offer an open listings API for third-party developers. This Actor reads the same search result pages a visitor sees, and the only credential your Node code holds is an Apify token.

### Why are deposit and key money `null` on some rows?

Condo rows never have them, because they belong to leases. On rental rows a `null` means Suumo showed a dash or a value the parser could not read, while `0` means the listing states that none is charged.

### Can my TypeScript code rely on the shape?

The field set is the same in rent and buy mode, so one interface covers both. Type the yen fields as `number | null`, `stations` as an array of objects, and `listing_mode` as `'rent' | 'buy'`.

### How do I keep costs predictable while developing?

Stay on one page with `maxPages: 1`, leave `fetchListingDetails` off, and launch with a ceiling such as `node index.js --max-charge 0.3`. Thirty cents is then the most that run can cost.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/suumo-japan-real-estate) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Suumo.jp Japan Real Estate Scraper on Apify Store](https://apify.com/DevilScrapes/suumo-japan-real-estate)
- [Python version of this example](https://github.com/devilscrapes/suumo-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Rightmove Scraper — UK Property Listings](https://apify.com/DevilScrapes/rightmove-uk-property) — UK sale and rental listings from Rightmove.
- [Immowelt Germany Real Estate Scraper](https://apify.com/DevilScrapes/immowelt-germany-real-estate-scraper) — German apartments and houses listed on Immowelt.
- [Otodom Poland Real Estate Scraper](https://apify.com/DevilScrapes/otodom-poland-real-estate-scraper) — Polish property offers from Otodom.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/suumo-japan-real-estate) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with SUUMO, Suumo.jp or Recruit Co., Ltd..
