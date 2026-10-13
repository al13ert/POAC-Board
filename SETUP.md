# Set up your live office TV — guided steps

You can use the sample dashboard immediately. Real staff data stays disconnected until you configure and protect the deployment. You do not need to connect ChatGPT apps for this permanent TV connection.

## 1. Create your Notion workspace content

Your connected Point of AC workspace already has a **Current Tasks** database, and this package is preconfigured to use it. Its existing fields map as follows:

| Current Tasks property | Dashboard field |
|---|---|
| Name | Task title |
| Assignee | Owner |
| Due | Due date |
| Status | Status |
| Priority | Priority |
| Project | Relation field; not currently rendered by the dashboard |

The database ID is already filled in as non-secret configuration. The backend discovers its single data source automatically. It deliberately does not display the **File Path/Link**, **Parent task**, or **Sub-task** fields.

For safe office-screen publication, add one checkbox property named **TV Visible** to Current Tasks. Check it only for tasks approved for the TV. The backend queries only checked rows. This property is not currently present in the database, so live tasks remain disconnected until you add it and share the database with the restricted integration below. Do not copy confidential tasks into a public feed.

If you are new to Notion or prefer a separate display database, sign in at https://www.notion.so, create a page named **POAC Command Centre**, type `/database`, choose a table database and create **Project Tasks**. Add these properties with exact names and types (case matters):

| Property | Type | How to use it |
|---|---|---|
| Task | Title | Rename the existing title column; do not add a second one |
| Project | Select | E.g. Studio, Brand Strategy, Campaign |
| Assigned To | Person | Choose the team member responsible |
| Status | Status | To do, In progress, Done |
| Priority | Select | High, Medium, Low |
| Due Date | Date | Target completion date |
| TV Visible | Checkbox | Check only after approving the title for office display |

Add two sample rows, assign people, set due dates, and check TV Visible. Done, Complete, Completed and Cancelled statuses are hidden by the backend. A Notion view filter alone does not control publication; the TV Visible property does.

Create **Company Announcements**, also a table database:

| Property | Type | How to use it |
|---|---|---|
| Announcement | Title | Short headline |
| Message | Text (rich text) | Optional short supporting message |
| Start Date | Date | First visible HK day; optional |
| End Date | Date | Last visible HK day; optional |
| Priority | Select | High, Medium, Low; High goes first |
| TV Visible | Checkbox | Publication switch |

Create **Project Milestones** if desired: Milestone (Title), Project (Select), Target Date (Date), Status (Status), TV Visible (Checkbox). Open milestones are sorted by date. Leave its ID blank if you want to retain the milestone demo.

For everyday use, staff update tasks in Notion as usual. Announcements require no code change: edit the row, set its dates, and check TV Visible. Allow about two minutes for the TV to update. The dates are publication days, not precise times. Prefer short headlines and messages that can be read across a room.

## 2. Make a restricted Notion integration

1. Visit https://www.notion.so/profile/integrations and create an **internal integration** named `POAC Office TV` in the correct workspace.
2. Enable **Read content** only. Do not enable insert/update content. If person names are not returned, check the integration's user-information capability; only enable names needed for display, without email access where possible.
3. In each TV database, open its menu → Connections → add your integration. Grant only the TV databases, not your whole workspace. Menu wording may vary with Notion updates.
4. Copy each database link. The database ID is the 32 hexadecimal characters (possibly with hyphens) in its URL before `?`; the `v=` query parameter is a view ID and must not be used.
5. Put database IDs in the matching fields of `wrangler.jsonc`: NOTION_TASKS_DATABASE_ID, NOTION_ANNOUNCEMENTS_DATABASE_ID, and optionally NOTION_MILESTONES_DATABASE_ID. IDs identify content; they are not access tokens, but can still be kept out of public repos if you prefer.
6. Keep each TV database to one data source. This backend retrieves its data-source ID automatically using Notion API version 2025-09-03. Multiple data sources produce ERROR instead of choosing an unintended source.
7. After the initial Cloudflare deployment in step 4, store the internal integration token using:

```sh
npx wrangler secret put NOTION_TOKEN
```

Paste the token only at the secret prompt in your own terminal, or enter it directly in the Cloudflare dashboard under Worker Settings → Variables and Secrets → Secret. Do not paste it in this chat or into wrangler.jsonc. Rotate/revoke this integration if the token is ever exposed.

## 3. Create a shared Google Calendar

1. Open https://calendar.google.com on your computer. Settings → Add calendar → Create new calendar.
2. Name it **Point of AC — Office Display** and select **Asia/Hong_Kong** time zone.
3. In its settings, Share with specific people or groups. Editors can have Make changes to events; the account used for TV authorisation needs See all event details. Keep public sharing OFF.
4. Add two sample events to this calendar, checking that you selected the new calendar when saving. Use short titles, e.g. `INTERNAL · Weekly planning` or `TEAM · Studio sharing`.
5. Settings → Integrate calendar → copy **Calendar ID** into GOOGLE_CALENDAR_ID in wrangler.jsonc. Do not use the secret iCal URL or an embed link.
6. Only put TV-appropriate meetings on this calendar. The backend removes events marked Private or Confidential, and does not publish descriptions, guests, meeting links, locations or attachments. It expands recurring events and displays Monday–Sunday in Hong Kong. All-day events use Google's exclusive end date.

The OAuth permission below can read events across calendars accessible to the authorising account. For least exposure, authorise a dedicated display account that can only read the office display calendar. The backend itself always requests the configured calendar ID.

## 4. Deploy the safe sample first

Install Node.js 22+ from https://nodejs.org and create a Cloudflare account at https://dash.cloudflare.com. Extract the ZIP and open a terminal in the extracted folder (the one containing package.json):

```sh
npm install
npm test
npx wrangler login
npm run deploy
```

Wrangler prints a Worker URL. Open it: you should see DEMO labels, sample tasks and no numeric market quotes. No live private integrations run while LIVE_ENABLED is false.

For reproducible installs, this package includes pnpm-lock.yaml. You can instead install pnpm and run `pnpm install --frozen-lockfile`; approve the esbuild and workerd build scripts if pnpm requests it. The npm path is simpler for beginners; its generated package-lock.json can be committed after a successful install.

## 5. Protect the entire website with Cloudflare Access

1. Add a domain you control to Cloudflare if you do not have one already. In your Worker → Settings → Domains & Routes, add a Custom Domain such as `office.yourcompany.com`.
2. In Cloudflare Zero Trust, set up your team domain, e.g. `your-team.cloudflareaccess.com`.
3. Access → Applications → Add application → Self-hosted. Set the application hostname to your complete office TV hostname. Protect **all paths**, including `/api/dashboard`, not just the home page.
4. Create an Allow policy for named staff email addresses or your company identity group. Do not use Everyone, Bypass, or a public allow rule. Email one-time PIN or your company identity provider can be used.
5. Choose a session duration permitted by your company policy; test how the TV browser signs in again after expiry. An external HDMI browser device is a fallback if the Skyworth browser cannot handle modern sign-in or JavaScript.
6. Copy the application's **Application Audience (AUD)** to ACCESS_AUD in wrangler.jsonc; set ACCESS_TEAM_DOMAIN to the team hostname only, without `https://` or a trailing slash.
7. Leave LIVE_ENABLED false for now. Verify the custom domain challenges an incognito browser to sign in. Sign in from your TV browser to confirm compatibility.
8. Disable alternate workers.dev and preview addresses in Worker settings or protect them with Access too. If you set `workers_dev` false in wrangler.jsonc, retain the custom domain route in the configuration, e.g. add `"routes": [{"pattern":"office.yourcompany.com","custom_domain":true}]`. Use your actual domain. Preview URLs are already disabled in this package.

The Worker verifies the Access JWT issuer, audience, expiry and signature for every request in live mode, including static assets. This also blocks direct origin access without a valid token. Merely sending an email header does not grant access. LIVE_ENABLED true without Access settings fails closed with 403.

## 6. Authorise Google securely on your computer

1. Open https://console.cloud.google.com, create/select a dedicated project, and enable **Google Calendar API**.
2. Under Google Auth Platform, configure Branding, Audience and Data Access. Choose Internal for an eligible company Google Workspace project; otherwise External. For testing, add only the authorising display account as a test user.
3. Add scope `https://www.googleapis.com/auth/calendar.events.readonly`. Configure a Web application OAuth client under Clients/Credentials.
4. Add this exact authorised redirect URI:

```
http://127.0.0.1:8788/oauth/callback
```

This loopback address is only for the local setup helper. Do not add an OAuth callback on the TV website. No JavaScript origin is needed for this server-side code flow.

5. Keep the Client ID and Client Secret open in your browser. Do not download them into this repository.
6. After `npx wrangler login` and the sample deployment, run:

```sh
npm run google:connect
```

7. Enter the client ID and secret at the hidden terminal prompts. The helper prints a Google consent URL. Open it in your own browser and sign in to the dedicated account that can read the shared calendar. Approve the event-read permission.
8. Google returns to the loopback listener on your computer. The helper checks a random OAuth state, uses PKCE, exchanges the code, and uploads GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET and GOOGLE_REFRESH_TOKEN directly to Cloudflare using Wrangler stdin. It does not write tokens to disk or print them. Keep the terminal open until all three Stored messages appear. If upload fails, rerun after fixing Wrangler login; a partial upload is not a complete connection.
9. Google External apps in Testing can issue refresh tokens that expire after seven days for these scopes. Use the appropriate production/Internal configuration and complete Google's verification requirements if applicable. A production app can still have tokens revoked; ERROR on the TV may require reauthorising. Do not treat a successful one-time consent as permanent access.

The Worker obtains short-lived access tokens from the refresh token and keeps these only in memory. To revoke, remove the app from the Google account's third-party access page and remove/replace the Worker secrets.

## 7. Weather, news and markets

**Hong Kong weather** needs no key. When live mode is enabled, the backend retrieves current temperature at Hong Kong Observatory and humidity from HKO. The interface labels current conditions; it is not a weather forecast or warning panel. HKO is credited in the source documentation.

**News** defaults to BBC Business RSS. The other requested publishers remain unconnected until you supply a licensed/authorised feed. Set NEWS_FEED_URL and optionally the secret NEWS_FEED_TOKEN for a combined JSON endpoint you control or a provider that supports the schema in FEEDS.md. Adding a normal publisher homepage URL will not work. No paywalls are scraped or bypassed. Confirm the provider permits office TV display of headlines. The footer of the news panel shows connected publishers.

**Markets** need an authorised quote provider that covers all five instruments, including Hang Seng Index and US ETFs. HSI is an index in points; do not substitute a stock or ETF with a similar name. Provider symbol mappings differ. Set MARKET_FEED_URL and optionally secret MARKET_FEED_TOKEN to an endpoint returning the documented schema. It must explicitly supply quote timestamps, currencies/points and delay descriptions. All five quotes are validated; incomplete coverage shows ERROR. Until this feed is supplied, market rows show DEMO with dashes. No market subscription or provider adapter is included for a specific paid vendor; FEEDS.md gives the complete integration contract.

Add feed URL fields inside the `vars` object of wrangler.jsonc. Use HTTPS URLs without credentials or query-string tokens. Store feed tokens only with `npx wrangler secret put MARKET_FEED_TOKEN` / `NEWS_FEED_TOKEN`. A provider that uses a different response format needs a small server-side adapter to this schema; never expose its key in browser JavaScript.

## 8. Switch on live mode and verify

Once Access, secrets and IDs are ready, set `"LIVE_ENABLED": "true"` in wrangler.jsonc and deploy:

```sh
npm test
npm run deploy
```

Open the protected custom domain and sign in. Check:

- Incognito and alternate hostnames cannot read the dashboard or `/api/dashboard` without signing in. Test a staff account that should be denied.
- Calendar shows only the office display calendar. Add a public-to-team event and a Private event: only the former should appear after two minutes. Check a recurring and an overnight event.
- An unchecked TV Visible task stays absent. Check it and confirm it appears; mark it Done and confirm it disappears.
- A future or expired announcement stays absent; today's checked announcement appears. Keep sensitive sample titles out of screenshots.
- Weather becomes LIVE; BBC headlines become LIVE. Other publishers and markets only connect when their feeds are configured.
- Open the page on the actual Skyworth. Choose 16:9 landscape and a 1920×1080 display signal or better; disable overscan/zoom if available. Use Full screen or your browser's kiosk/fullscreen control. Test readability at your normal seating distance.
- Disconnect the internet: the visible connection warning should appear on the next poll. Reconnect: it retries automatically. Test reloading during an outage and Access session expiry.

If a configured provider shows ERROR, confirm IDs, exact Notion property types, database sharing, API activation, Google consent and feed subscription. The public API deliberately omits raw provider errors and credentials. Check provider dashboards for troubleshooting without posting tokens. Revoking an integration should cause its module to go ERROR on the next cache refresh.

## 9. GitHub safely

A GitHub username is not required for this ZIP or a direct Wrangler deployment. If you want source control, create a private repository, then commit the source and lockfile. The included .gitignore excludes node_modules, .wrangler, .dev.vars, .env, private keys and common credential filenames. Ignoring patterns is a convenience, not a guarantee: inspect staged files before pushing.

Commit only code, public branding assets, docs and non-secret configuration. Never commit actual Notion tokens, Google client secrets/refresh tokens, Cloudflare API tokens, real API response exports, or copied staff data. Keep secrets in Cloudflare Workers Secrets. Cloudflare Access protects the hosted site even if the code repository is later made public. GitHub Pages alone is not suitable for this private dashboard.

For GitHub deployment automation later, place the Cloudflare deployment token in GitHub Actions Secrets with the minimum deployment permissions. Do not add your business integration secrets to Actions; they already reside in the Worker. No CI deployment workflow is included or needed initially.

## Reference documentation

Verified against official sources on 13 October 2026:

- Google server-side OAuth, offline refresh tokens and testing restrictions: https://developers.google.com/identity/protocols/oauth2/web-server
- Google Calendar events list: https://developers.google.com/workspace/calendar/api/v3/reference/events/list
- Notion data-source query: https://developers.notion.com/reference/query-a-data-source
- Notion version migration: https://developers.notion.com/docs/upgrade-guide-2025-09-03
- Cloudflare Worker static assets: https://developers.cloudflare.com/workers/static-assets/
- Cloudflare Access on Workers: https://developers.cloudflare.com/workers/configuration/cloudflare-access/
- Worker secrets: https://developers.cloudflare.com/workers/configuration/secrets/
- HKO Open Data API: https://data.weather.gov.hk/weatherAPI/doc/HKO_Open_Data_API_Documentation.pdf
