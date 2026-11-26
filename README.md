# CarbonTrace

CarbonTrace is a local-first environmental decision-support prototype. It estimates activity-related greenhouse-gas emissions, ranks recorded categories, compares a simple hypothetical scenario, and can request an AI interpretation of aggregate totals.

## Features

- Responsive dark dashboard
- Electricity, transport, and purchase activity inputs
- Transparent, editable illustrative calculation factors
- Local browser storage (no account/database required)
- Category breakdown, ranking, and scenario simulator
- Optional Gemini-powered interpretation through a Vercel serverless function
- Server-side API key handling, basic request validation, same-origin browser checks, request-size cap, and best-effort rate limiting

## Deploy to GitHub + Vercel

1. Upload the project contents to the root of a new GitHub repository.
2. Import that repository into Vercel. The project uses static HTML/CSS/JS plus a Vercel Function in `api/analyze.js`; no build command or framework preset is required.
3. In Vercel, open **Project → Settings → Environment Variables**.
4. Add `GEMINI_API_KEY` with your private Google AI Studio key. Select the environments you need (Preview and Production).
5. Redeploy after adding or changing the variable.
6. Open the deployed site and add sample activities. The dashboard works without an AI key; only the AI analysis button needs it.

## Local preview

The API function requires a Vercel-compatible serverless runtime. For a quick static preview, open `index.html` or serve the folder with a local static server. The dashboard and calculations work; `/api/analyze` will only work when deployed to Vercel or run with Vercel's local development tooling.

## Security notes

- **Never put the Gemini key in `app.js`, `index.html`, or any `VITE_`/`NEXT_PUBLIC_` variable.** The browser calls `/api/analyze`; the server reads `process.env.GEMINI_API_KEY`.
- Never commit `.env` files. `.gitignore` excludes them; `.env.example` contains only a placeholder.
- Restrict the API key to the Gemini API in Google AI Studio / Cloud credential settings when that option is available. Set sensible usage quotas and monitor usage.
- The API route does not return provider error bodies or log prompts/keys. It limits request size and validates the aggregate payload.
- The in-memory rate limiter is best-effort only. Serverless instances can scale independently and reset; for a public production launch, add a durable shared rate limiter (for example, a managed Redis/KV service) and abuse monitoring.
- Browser local storage is not encrypted. Do not enter personal, financial, health, or other sensitive information. Clearing browser storage removes locally saved activities.
- The same-origin check is a basic browser-origin control, not user authentication. Anyone who can access a public deployment can call its public AI endpoint, subject to configured limits and provider quotas.

## Calculation assumptions

These are illustrative screening factors, not official country-specific factors or lifecycle assessments:

- Electricity default: 0.45 kg CO₂e per kWh. Users can edit this in the input form.
- Transport: car 0.192, bus 0.105, rail 0.041, flight 0.255 kg CO₂e per passenger-km.
- Purchases: broad spend-based proxies by category, not product-specific footprints.
- Daily and weekly entries are annualized to an approximate monthly total (30 days / 4.345 weeks); monthly entries are unchanged.

For serious reporting, replace these assumptions with documented, location-specific, current factors and product/supplier-specific lifecycle data. Keep factor source, year, geography, units, and methodology alongside every factor. Results should not be represented as audited emissions, regulatory disclosures, or carbon credits.

## Gemini model

The server currently calls `gemini-3.8-flash` using the Gemini REST `generateContent` endpoint. If your Google AI Studio project does not have access to that model or the endpoint has changed, update the model/endpoint in `api/analyze.js` to a currently supported model and API method. The API key remains server-side.

## Project structure

```text
carbontrace/
├── api/
│   └── analyze.js
├── .env.example
├── .gitignore
├── app.js
├── index.html
├── styles.css
├── vercel.json
└── README.md
```
