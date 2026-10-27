# POSTCODE LIFELINE

**Postcode-based emergency reporting and dispatch for Nigeria.**

POSTCODE LIFELINE uses Nigeria’s new 11-character NIPOST postcodes (for example `FC 02 A09 DB 09`) to give every emergency report a precise location from the first moment. Callers report with their postcode; the control room sees the exact location, every responder within 20 km ranked by estimated response time, and the fastest route for each.

> **Demo version.** This repository is a working prototype for pitching and testing. The sample postcodes, locations, responder stations and road network are illustrative, not official NIPOST or agency data. Routes and response times are estimates.

## The problem

Many addresses in Nigeria are described by landmarks rather than street names and numbers. In an emergency, callers struggle to give clear directions, control rooms struggle to pass them on, and responders lose time searching. When the location is unclear, it is also hard to send the nearest unit or the agency responsible for the area.

## What the demo does

- **Citizen reporting by USSD (simulated):** dial `*3112#`, choose the emergency type, enter a postcode and confirm the location read back. The short code is a placeholder.
- **Postcode verification:** codes are checked against NIPOST’s Postcode API when a key is configured. The built-in sample codes always work, so the demo runs offline.
- **Control-room dashboard:** live incident map, queue and status tracking from report to resolution.
- **Responders within 20 km:** every fire station, police division, hospital and FRSC unit in range, sorted by estimated response time, with the fastest unit of each needed service pre-selected.
- **Fastest route:** shown on the map for any selected responder, with distance and time.
- **Safeguards:** flags for unverified postcodes and repeat callers, call-back fallback, and a dispatcher’s manual entry for phone calls.

## How it works

```
Citizen (USSD / WhatsApp / call)
        │  postcode
        ▼
Web dashboard (public/index.html)
        │  /api/lookup?code=…
        ▼
Netlify Function (adds the API key on the server)
        │  X-API-Key
        ▼
NIPOST Postcode API v1  →  official location
```

The browser never sees the API key. It is read from an environment variable by the functions.

| Route | Purpose | NIPOST endpoint |
|---|---|---|
| `/api/health` | Tells the page whether live lookups are configured | – |
| `/api/lookup?code=` | Verifies a postcode and returns its location | `/v1/lookup` |
| `/api/autocomplete?q=` | Suggests postcodes from partial input | `/v1/search/autocomplete` |

## Project structure

```
public/index.html            Prototype page (citizen phone + control-room dashboard)
netlify/functions/           health, lookup and autocomplete functions
netlify/shared/nipost.mjs    Shared helpers: postcode checks, NIPOST calls, response parsing
netlify.toml                 Netlify build settings and security headers
.env.example                 Template for local settings (never commit .env)
```

## Run it

**Just the demo (sample data):** open `public/index.html` in a browser.

**With live NIPOST lookups, locally:**

```bash
npm install -g netlify-cli
cp .env.example .env      # then add your key to .env
netlify dev
```

**Deploy:** connect this repository to Netlify (Add new site > Import an existing project), then add `POSTCODE_API_KEY` under Site configuration > Environment variables and redeploy.

| Variable | Required | Default | Notes |
|---|---|---|---|
| `POSTCODE_API_KEY` | Yes, for live lookups | – | Your NIPOST key. Keep it secret. |
| `POSTCODE_LOOKUP_LEVEL` | No | `1` | Passed to `/v1/lookup` as `level`. |
| `POSTCODE_API_BASE` | No | `https://api.postcode.gov.ng/v1` | |
| `DEBUG` | No | off | `1` includes NIPOST’s raw reply in lookups. Testing only. |

## Security and privacy

- API keys live only in environment variables; `.env` is excluded by `.gitignore`.
- Inputs are validated before any call to NIPOST, and calls time out after 8 seconds.
- Strict security headers and a content-security policy are set in `netlify.toml`.
- A live system would follow the Nigeria Data Protection Act 2023, with encryption, role-based access and a full audit trail.

## Known limitations and next steps

- NIPOST’s response format is handled generically until confirmed; adjust `extractLocation` in `netlify/shared/nipost.mjs` once known.
- Routes use a simplified road network; a pilot would use a real road-routing service with traffic data.
- Real USSD and WhatsApp channels need a telecom aggregator, an NCC-assigned short code and the WhatsApp Business API.
- Entering letters over USSD is awkward on many basic phones; menu-based or keypad-mapped entry is being explored.
- No user accounts or rate limiting yet; required before any live use.

## Status

Prototype for a proposed pilot in one FCT district, in partnership with NIPOST, the FCT emergency services, responding agencies and NCC.

**Project lead:** Laban Emmanuel Rakum
