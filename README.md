# Point of AC Office Command Centre — version 2

A 1920 × 1080 office TV dashboard and Cloudflare Worker backend. Designed for a 55-inch Skyworth in landscape. Includes weekly Calendar, Notion tasks, announcements, milestones, Hong Kong weather, news, HSI / NVDA / AAPL / QQQ / VOO, Hong Kong / London / New York clocks, and demo mode.

Start with **SETUP.md**, which teaches the Notion and Calendar setup and takes you through deployment. No credentials are included. Do not put credentials in GitHub, browser code, screenshots, or chat.

## Quick start

Install Node.js 22 or later. Extract this ZIP, open a terminal in its folder:

```sh
npm install
npm test
npm run dev
```

Open the local URL printed by Wrangler. Everything starts in demo mode. `?demo` forces a visual demo. Preview requires an HTTP server; do not double-click index.html. The production package uses one Cloudflare Worker for both the website and private API: no separate CORS setup and no GitHub Pages deployment.

```sh
npx wrangler login
npm run deploy
```

This initially publishes **sample data only**. Follow SETUP.md to configure Access before switching on live connections. A Cloudflare account and your own domain are recommended. Deployment has not been performed on your behalf.

## What is connected when configured

| Module | Backend implementation | Refresh/cache |
|---|---|---|
| Calendar | Google Calendar API; recurring instances, pagination, Monday–Sunday HK week | 2 minutes |
| Tasks / announcements / milestones | Notion database → single data source; paginated, TV Visible filter | 2 minutes |
| Weather | Hong Kong Observatory current conditions, no key | 15 minutes |
| News | BBC business RSS by default; optional authorised JSON feed for five requested publishers | 10 minutes |
| Markets | Authorised JSON feed adapter for all five symbols | 5 minutes |
| Clocks | Browser clock with IANA time zones | Every second |

The browser requests the combined API every 2 minutes; backend caches set minimum provider refresh intervals (a 5-minute cache may therefore refresh on the next 2-minute poll). In-memory caches can reset between Worker instances; this is not a global quota guarantee. Choose provider limits accordingly. Data fetches happen while a screen is open, without a scheduled task.

Each module shows DEMO / LIVE / ERROR and last successful retrieval time. Quote rows show their provider timestamp, currency, and delay label. A LIVE market module means a connected feed, not a guarantee of real-time prices. Missing credentials leave that module in demo. Failed configured connections show ERROR and no invented replacement data. Browser connection loss preserves its last response in memory with a visible warning; a reload starts demo if the backend is unavailable. Nothing private is stored in localStorage or service-worker caches.

Lists rotate every 15 seconds, including calendar days with more than two events. Long display text is clipped to fit: use short TV-friendly titles. Calendar week is recalculated continuously. Announcements use inclusive start/end calendar dates in Hong Kong; leave either blank for no boundary.

## Branding and provenance

The original version-1 source was unavailable in the referenced chat, so version 2 is a reconstruction rather than an in-place patch. The attached POAC PDF was recovered from the chat attachment and inspected. See BRAND.md for exact colours and logo extraction details. Font files are not included because font redistribution rights were not supplied.

## Verification

Automated checks cover HK week boundaries, exclusive all-day ends, overnight events, demo isolation, fail-closed Access, private event filtering, API pagination, announcement date filters, and incomplete quote rejection. Run `npm test`. Account-specific live connections still need your acceptance checks in SETUP.md. This package does not imply the TV browser or your provider accounts have been tested.
