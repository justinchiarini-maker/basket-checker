# Basket Checker

Enter a UK postcode (or tap **Use my location**) and see which supermarkets are near you and what a weekly basket costs at each one.

- **Location** – postcodes.io turns a postcode into coordinates, or the browser's GPS does it directly.
- **Nearby stores** – OpenStreetMap (via the Overpass API) lists every Aldi, Lidl, Asda, Tesco, Sainsbury's, Morrisons, Waitrose, Co-op, Iceland and M&S Food within 1–10 miles, flagging Express/Local-type shops.
- **Prices** – Open Prices (Open Food Facts) holds prices shoppers have recorded from receipts and shelf labels. For each item and chain the app uses, in order: the median of prices recorded within 15 km of you, then the UK-wide median, then a built-in estimate marked **≈**.

Every price is converted to the same pack size before comparing (e.g. a 1 kg bag of pasta counts as two 500 g bags).

No API keys, no database and no npm dependencies. Node 20+ only.

## Run it locally

```bash
npm start            # http://localhost:3000
npm test             # unit tests + end-to-end against mock data services
```

To work offline or without hitting the real services:

```bash
npm run mock         # fake data services on :4010
POSTCODES_API=http://localhost:4010/pc \
OPEN_PRICES_API=http://localhost:4010/op/api/v1 \
OVERPASS_APIS=http://localhost:4010/overpass npm start
```

## Deploy

Push this folder to a GitHub repo, then pick one:

**Render (free tier, simplest)** – New → Blueprint → select the repo. `render.yaml` sets everything up. Free instances sleep when idle, so the first visit after a quiet spell takes ~30 s.

**Any Docker host** (Fly.io, Railway, AWS App Runner, a VPS):

```bash
docker build -t basket-checker .
docker run -p 3000:3000 -e USER_AGENT="BasketChecker/1.0 (+https://your-domain)" basket-checker
```

Run it behind HTTPS: browsers only allow **Use my location** on secure pages. All the hosts above give you HTTPS automatically.

## Settings (environment variables)

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | 3000 | Port to listen on |
| `USER_AGENT` | BasketChecker/1.0 | Sent to the data services; put your site URL or contact in it |
| `PRICE_MAX_AGE_DAYS` | 180 | Ignore prices older than this |
| `LOCAL_PRICE_KM` | 15 | Distance that counts as "seen near you" |
| `RATE_LIMIT_PER_MINUTE` | 60 | API requests per visitor per minute |
| `POSTCODES_API`, `OPEN_PRICES_API`, `OVERPASS_APIS` | public services | Point at mocks or your own mirrors |

Answers from the data services are cached in memory (postcodes 30 days, stores 7 days, prices 6 hours), so traffic to them stays low.

## Changing the basket

Edit `src/items.js`. Each item lists its reference pack size, Open Food Facts category tags to match, and fallback estimates per chain. Check a category tag returns UK results before relying on it, e.g.
`https://prices.openfoodfacts.org/api/v1/prices?currency=GBP&product__categories_tags__contains=en:baked-beans`

## How good is the price data?

Open Prices is crowd-sourced and still growing in the UK, so expect some items to show **≈** estimates, especially for smaller chains. The page says which prices are recorded and which are estimates, and links people to Open Prices to add their own receipts. Coverage improves as people contribute.

If you later want complete, daily prices, the options are a commercial grocery-price data feed, or a partnership with a price-tracking service. Scraping supermarket websites usually breaks their terms of use, so this app doesn't do it.

## Data licences

- Store locations © OpenStreetMap contributors, ODbL – attribution is shown in the page footer.
- Prices: Open Prices, ODbL.
- Postcodes: postcodes.io, contains OS data © Crown copyright and database right, Royal Mail data © Royal Mail copyright and database right, ONS data © Crown copyright.

Keep the footer attribution if you change the design.
