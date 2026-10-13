# Authorised feed contracts

These adapters make the dashboard independent of a specific commercial provider. They are real HTTP integrations, but you must select a provider and arrange any format conversion and display rights. There is no vendor subscription or proprietary quote key in the ZIP. Credentials are optional Bearer tokens supplied by the Worker, never the browser.

## Markets

Configure MARKET_FEED_URL (HTTPS, no credentials embedded in URL) and optional Worker secret MARKET_FEED_TOKEN. Endpoint returns JSON:

```json
{"quotes":[
 {"symbol":"HSI","price":25000,"changePercent":0.5,"currency":"points","asOf":"2026-10-13T08:00:00Z","delay":"Delayed 15 min"},
 {"symbol":"NVDA","price":100,"changePercent":0.5,"currency":"USD","asOf":"2026-10-12T20:00:00Z","delay":"Previous close"},
 {"symbol":"AAPL","price":100,"changePercent":0.5,"currency":"USD","asOf":"2026-10-12T20:00:00Z","delay":"Previous close"},
 {"symbol":"QQQ","price":100,"changePercent":0.5,"currency":"USD","asOf":"2026-10-12T20:00:00Z","delay":"Previous close"},
 {"symbol":"VOO","price":100,"changePercent":0.5,"currency":"USD","asOf":"2026-10-12T20:00:00Z","delay":"Previous close"}
]}
```

Numbers above are fictional schema examples only and are not loaded by the dashboard. `changePercent` is a percentage (0.5 means +0.5%, not +50%). Supply all symbols, numeric price/changePercent, valid ISO timestamp, currency/points, and an explicit delay label. Provider must account for market holidays, time zones and previous closes. The TV shows asOf and delay so a cached or closed-market quote is not presented as a current tick. Never label a delayed provider as real-time. Backend refresh/cache is 5 minutes and can reset between isolates.

## News

Configure NEWS_FEED_URL and optional secret NEWS_FEED_TOKEN. Endpoint returns:

```json
{"articles":[{"title":"A short authorised headline","source":"Reuters","publishedAt":"2026-10-13T08:00:00Z"}]}
```

Allowed source labels are SCMP, BBC, Reuters, CNBC, Bloomberg. Use only headlines you have rights to display, with valid ISO timestamps. Articles are limited to 60 and rotated on screen; bodies and images are not fetched. This combined endpoint replaces BBC-only RSS rather than being merged with it. Without this endpoint, the backend uses https://feeds.bbci.co.uk/news/business/rss.xml. If you need all five publishers, return all five under the required labels; sources absent from a response are not magically connected.

## Acceptance

Check the endpoint from your server before connecting it: HTTPS, 200 JSON, correct content and permitted usage. Tokens must not be returned in JSON or reflected in error messages. The Worker follows no redirects for feeds, sends credentials only to the configured URL, and suppresses raw error bodies. If your provider accepts API keys only as query parameters, create a server-side adapter and use a Bearer token between it and this Worker.
