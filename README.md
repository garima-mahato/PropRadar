# PropRadar – AI Property Finder

> A Chrome extension that finds flats, houses, and land for rent or sale across 99Acres, MagicBricks, and NoBroker — ranked by road distance from your key locations, powered by Google Gemini AI and OpenStreetMap.

---

[![Watch the video](https://img.youtube.com/vi/uPs78Qf4vJA/0.jpg)](https://www.youtube.com/watch?v=uPs78Qf4vJA)


## What it does

You pin up to 5 locations on a map (your office, gym, school — anywhere you commute to regularly). PropRadar asks Gemini AI to find properties near all of them simultaneously, ranks results by a weighted commute score, and plots them as markers on an interactive map with a detail panel on the right.

---

## Features

- **Location-aware search** — add up to 5 key locations via autocomplete; properties are scored and ranked by proximity to all of them combined
- **AI-powered** — Gemini searches 99Acres, MagicBricks, and NoBroker and returns structured listings with distance estimates, metro proximity, and match scores
- **Interactive map** — custom dark-themed tile map built on OpenStreetMap and CartoDB Dark Matter tiles; click any marker to see property details
- **Rich filters** — transaction type (rent/buy), property type, BHK configuration, budget slider, max road distance, metro preference, city, and results count
- **Model selector** — choose from 4 Gemini models with live rate-limit info (RPM / TPM / requests per day) shown on each card; model preference persists across sessions
- **Model ID editor** — override the API model string per model without touching code, for when Google renames or updates model identifiers
- **Encrypted key storage** — your Gemini API key is encrypted with AES-256-GCM before being written to browser storage; the encryption key is derived from your unique extension ID via PBKDF2, so the stored blob is useless outside your install
- **Token-aware** — `maxOutputTokens` scales with the results count you select; thinking is automatically disabled for Gemini 2.5/3.x models to prevent the thinking token budget from consuming your output allowance
- **Truncation recovery** — if a response is still cut off, the parser salvages any fully-formed property objects rather than failing silently
- **Zero cost map** — no Google Maps API key, no credit card for the map; OpenStreetMap + Nominatim geocoding are completely free

---

## Requirements

| Requirement | Notes |
|---|---|
| Google Chrome (or Chromium) | Version 88+ for Manifest V3 support |
| Gemini API key | Free tier available — no credit card needed for the key itself |
| Internet connection | For map tiles, geocoding, and AI search |

The map, geocoding, and tile rendering are all free with no signup. Only the Gemini AI call requires an API key.

---

## Installation

**1. Download and extract**

Download `propradar-v2-final.zip` and extract it. You should have a folder called `propradar-v2` containing `manifest.json`, `app.js`, and the other files.

**2. Load into Chrome**

1. Open Chrome and navigate to `chrome://extensions`
2. Enable **Developer mode** using the toggle in the top-right corner
3. Click **Load unpacked**
4. Select the extracted `propradar-v2` folder

The PropRadar icon (🏠) will appear in your Chrome toolbar.

**3. Get a Gemini API key**

1. Go to [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
2. Sign in with a Google account
3. Click **Create API key** and copy it

**4. Configure the extension**

1. Click the PropRadar icon in the toolbar
2. Click **Configure Gemini Key**
3. Paste your API key and click **Save & Open PropRadar**

The settings page opens in the current tab. After saving, PropRadar opens automatically.

---

## Using PropRadar

### Adding locations

Type a locality, landmark, or address into any location field. A dropdown of suggestions appears (powered by Nominatim / OpenStreetMap). Select one and a coloured numbered pin drops on the map. Add up to 5 locations — each gets its own colour.

### Setting filters

| Filter | Options |
|---|---|
| Transaction | Rent · Buy/Sale · Both |
| Property type | Flat/Apartment · Independent House · Villa · PG/Co-living · Plot/Land |
| Configuration | 1 BHK · 2 BHK · 3 BHK · 4+ BHK · Studio · Any |
| Budget | Slider from ₹5,000/mo to ₹5L+/mo (rent) or ₹20L to ₹10Cr+ (buy) |
| Max road distance | 3 / 5 / 8 / 10 / 15 / 20 km · Any |
| Metro preference | Any · Near metro (≤1 km) · Walkable metro |
| Results count | 3 · 5 · 8 · 10 · 15 — fewer results use fewer tokens |
| City | Bengaluru · Mumbai · Delhi/NCR · Hyderabad · Chennai · Pune · Kolkata · Ahmedabad |
| Sources | 99Acres · MagicBricks · NoBroker (toggle each on/off) |

### Running a search

Click **Find Properties with AI**. The right panel slides open and shows a loading state while Gemini searches. Results typically arrive in 10–20 seconds.

### Reading results

Each card shows the price, source platform, BHK/area/furnishing, road distance to each of your key locations, nearest metro station, and an AI-generated match score (0–100). Click a card to fly the map to that property and open its popup. Use the sort tabs to re-order by Best Match, Price, or Distance.

### Clicking map markers

Property markers are colour-coded by source (red = 99Acres, blue = MagicBricks, teal = NoBroker). Click any marker to see a summary popup. Clicking a property card also centres the map on it.

---

## Managing your API key

Click the **🔑 Manage API Key** button at the bottom of the sidebar at any time to:

- View your current key (masked by default)
- Toggle show/hide
- Save a new or updated key
- Delete the key (redirects to the settings page)

---

## Choosing a Gemini model

The **🤖 Gemini Model** section in the sidebar lists available models with their free-tier rate limits. The default is **Gemini 3.1 Flash Lite** (500 requests/day) which offers the highest free-tier daily allowance.

| Model | RPM | TPM | Req/Day | Notes |
|---|---|---|---|---|
| Gemini 3.1 Flash Lite | 15 | 250K | 500 | Default · best free quota |
| Gemini 2.5 Flash Lite | 10 | 250K | 20 | Balanced |
| Gemini 2.5 Flash | 5 | 250K | 20 | More capable |
| Gemini 3 Flash | 5 | 250K | 20 | Latest Flash |

Click a card to switch models. Your selection persists between sessions.

### Editing model API IDs

If Google renames a model or you want to test a preview model not in the list, click the **✏** pencil icon on any model card to open the inline editor. Type the exact API model string (e.g. `gemini-2.5-flash-preview-05-20`), click **Save**, and PropRadar uses that ID for all subsequent calls to that card. An edited indicator (✎) appears on the card. Click **↩ Reset to default** to revert.

---

## Token usage

PropRadar is designed to minimise token consumption:

- The prompt is compact (~270 input tokens)
- `maxOutputTokens` scales with results count: `(count × 420) + 512`
- Thinking is disabled for Gemini 2.5/3.x models (`thinkingBudget: 0`) — these models otherwise spend thousands of tokens on internal reasoning that counts against your output budget
- `responseMimeType: application/json` is set to encourage clean structured output

Approximate output tokens per results count:

| Results | Output tokens | Fits free tier? |
|---|---|---|
| 3 | ~1,260 | Yes |
| 5 | ~2,100 | Yes |
| 8 | ~3,360 | Yes |
| 10 | ~4,200 | Yes |
| 15 | ~6,300 | Yes (with budget) |

---

## Handling quota errors

If you see a quota exceeded error:

1. **Switch model** — select Gemini 3.1 Flash Lite (500/day) in the model selector
2. **Reduce results** — choose 3 or 5 in the Results Count filter
3. **Wait** — free-tier quotas reset at midnight Pacific time
4. **Upgrade** — a paid Gemini API key removes free-tier restrictions entirely

---

## Debugging

Open Chrome DevTools (`F12`) on the PropRadar tab and check the **Console**. PropRadar logs:

- `[PropRadar] Starting search…` — model name and selected locations
- `[PropRadar] Gemini Request` — the full prompt (API key is redacted)
- `[PropRadar] Gemini Response` — raw model output, finish reason, and token usage
- Warnings if JSON parsing or truncation recovery runs

The finish reason is especially useful: `STOP` is normal, `MAX_TOKENS` means the response was cut off.

---

## Privacy and security

- Your Gemini API key is encrypted with **AES-256-GCM** before storage
- The encryption key is derived from your Chrome extension's unique install ID via **PBKDF2** (120,000 iterations, SHA-256)
- Even if `chrome.storage` is dumped, the ciphertext cannot be decrypted without the exact same extension installation
- The storage field name is SHA-256 hashed so it is not labelled `geminiKey` in storage
- The API key is sent **only** to `generativelanguage.googleapis.com` — never to any other server
- Map tiles load from CartoDB's servers; no user data is sent with tile requests
- Location geocoding goes to `nominatim.openstreetmap.org` — only the search query text is sent, with no user identifiers

---

## File structure

```
propradar-v2/
├── manifest.json       Chrome extension manifest (MV3)
├── app.html            Main application page
├── app.js              Core application logic (~1,100 lines)
├── minimap.js          Self-contained tile map library (~460 lines)
├── crypto.js           AES-256-GCM key encryption utilities
├── styles.css          All styles
├── popup.html          Extension toolbar popup
├── popup.js            Popup logic
├── settings.html       API key configuration page
├── settings.js         Settings page logic
└── icons/
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

All scripts are local — no CDN dependencies, no external script loading. The extension satisfies Chrome's Manifest V3 content security policy with no CSP exceptions.

---

## Limitations

- Property listings are AI-generated estimates based on Gemini's training data and market knowledge, not live scraped data. Always verify on the source platform before visiting a property.
- Distance estimates are straight-line approximations, not real road routing. Actual travel times will vary with traffic.
- Nominatim geocoding quality varies for very new developments or informal locality names. If a location is not found, try a nearby landmark or the full address.
- Free-tier Gemini models have daily request limits. For heavy use, a paid API key is recommended.

---

## Acknowledgements

- Map tiles by [CartoDB](https://carto.com) under [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/)
- Map data by [OpenStreetMap](https://www.openstreetmap.org) contributors under [ODbL](https://opendatacommons.org/licenses/odbl/)
- Geocoding by [Nominatim](https://nominatim.org) / OpenStreetMap
- AI search powered by [Google Gemini](https://ai.google.dev)
