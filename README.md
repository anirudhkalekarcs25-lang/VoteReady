# VoteReady

A neutral, non-partisan guide for first-time voters in India. It covers eligibility, Form 6 registration, polling booth lookup, accepted ID on polling day, and state-level official resources. Every action link leads to an official Election Commission of India (ECI) service.

VoteReady is independent and is not affiliated with the ECI or any government body.

## Pages

| Path | Purpose |
|---|---|
| `index.html` | Home |
| `checklist.html` | Eight-step readiness checklist, progress saved on the device |
| `qa.html` | Voting-process Q&A backed by Gemini through a server proxy |
| `map.html` | State and union territory resources, optional directions to the CEO office |
| `privacy.html` | Privacy policy |
| `terms.html` | Terms and conditions |

## Stack

- Plain HTML, CSS and JavaScript. No build step, no frameworks.
- One design system in `assets/site.css`. Shared header, footer and English/Hindi switching in `assets/site.js`.
- Checklist content in `data/checklist.json` and `data/checklist_hi.json`, each step with an official link, source and verification date.
- Vercel serverless functions in `api/`.
- D3 and TopoJSON from cdnjs for the state map. Map data is served from `assets/india.json`.

## Backend

`api/gemini.js` (POST)
- Accepts `{ userQuestion, lang, userContext: { completedSteps } }`. No voter identifiers are accepted or forwarded.
- Validates length (500 characters), type and allowlisted step IDs.
- Checks `Origin` against `ALLOWED_ORIGIN`, applies a best-effort per-IP rate limit, and times out upstream calls at 20 seconds.
- Sends the question to Gemini with a constrained system prompt. Out-of-scope or political questions return a fixed fallback pointing to the official portal.
- Returns `{ answer, unsupported, source, officialUrl }` or `{ error: { code, message } }`.

`api/maps-config.js` (GET) returns the browser Maps key. The browser only requests it after the user presses "Show directions from my location".

The in-memory rate limit is per serverless instance. For a hard limit, add a Vercel WAF or edge rate-limit rule on `/api/gemini`.

## Run locally

Static pages: `python3 -m http.server 8000`, then open http://localhost:8000.

With the API: install the Vercel CLI, copy `.env.example` to `.env.local`, fill in the keys, then run `vercel dev`.

## Environment variables

| Name | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini API key (server only) |
| `GEMINI_MODEL` | Optional, defaults to `gemini-2.5-flash` |
| `GOOGLE_MAPS_KEY` | Maps JavaScript and Directions key. Restrict by referrer and API |
| `ALLOWED_ORIGIN` | Comma separated allowed origins, for example `https://www.example.org,https://example.org` |

## Before launch

1. Add the custom domain in Vercel (Project, Settings, Domains) and point DNS as Vercel instructs.
2. Set `ALLOWED_ORIGIN` to the final origins and restrict the Maps key to the same domain.
3. Set `CONTACT_EMAIL` at the top of `assets/site.js`. The privacy policy, terms and footer show it once set.
4. Add the operating organisation's legal name to `privacy.html` and `terms.html` if you want it named there.
5. Re-verify every checklist step against the official source and update `verified_on`.
6. Check that the favicon appears in a private browser window on the live domain.

## Content rules

- Primary official sources only. Remove a claim rather than keep a guess.
- No candidate, party or outcome content.
- Do not store or forward voter identifiers.
- No sample voter data. The checklist sends people to the official Electoral Search service.
