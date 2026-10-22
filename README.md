# GCC Atlas

A one-page tool for a company deciding where in India to open a Global Capability Centre (GCC). It ranks six cities by what you care about, links every figure to its source, and shows where published reports disagree.

**Live:** https://gccweb-eight.vercel.app

## How it works

1. Describe your company in the box at the top, or click one of the sample cards.
2. The page sets four priority sliders (talent cost, GCC ecosystem depth, state policy support, sector fit) and shows:
   - **Your shortlist**: Bengaluru, Hyderabad, Pune, Chennai, Delhi NCR and Mumbai ranked, each with reasons and a "Watch out" line
   - **City sheet**: the raw figures, each with a source tag and a confidence dot
   - **Where the sources disagree**: claims that conflict across reports, with how Atlas treats each one
   - **Ask Atlas**: questions answered only from the page's data, with source citations
3. Text that ends in `?` goes straight to Ask Atlas.

### Ranking

The ranking is plain JavaScript, so the same sliders always give the same order. Each factor is rescaled to 0–100 across the six cities, and the score is a weighted average using the slider values:

| Factor | Measure | Source |
|---|---|---|
| Talent cost | Median software engineer pay (lower is better) | Flexiple |
| Ecosystem depth | Share of GCC office leasing, 2022 to H1 2026 | CBRE |
| State policy | Dedicated GCC policy / framework / incentives only | CBRE |
| Sector fit | Whether your sector is a noted strength of the city | InTech, CBRE |

The top card explains itself when it wins on one factor while being the weakest of the six on another, or when it is under 5 points ahead of the next city.

### Where AI is used

Two features call [Groq](https://groq.com) through one serverless endpoint, `/api/ask`:

- **Reading a description**: turns free text into slider weights and a one-line reason. It runs at temperature 0 with a fixed seed. The sample cards use fixed weights and make no AI call.
- **Ask Atlas**: answers from the page's data only, cites `[S#]` after each fact, flags low-confidence sources, and says when sources disagree instead of picking one.

The ranking, explanations, city sheet and disagreement cards never use AI. If the assistant is unavailable, the rest of the page still works and the sliders can be set by hand.

## Data

Nine public sources collected on 21 October 2026, stored in `index.html`. Each source is graded as an advisory or industry report, platform or vendor data, or a single blog (indicative only). Figures are as reported and are not independently verified. Rankings are a starting point for discussion, not legal, tax or investment advice.

Not yet covered: provider track records, incentive amounts by state, and Tier-2 cities.

## Project structure

```
index.html    The whole page: layout, data, ranking and UI
api/ask.js    Serverless endpoint that forwards to Groq and streams the answer back
.env.example  Environment variables to copy into .env
```

## Run locally

Requires Node 18 or later and a Groq API key from https://console.groq.com/keys.

```bash
cp .env.example .env      # then put your key in GROQ_API_KEY
npx vercel dev            # serves the page and /api/ask on http://localhost:3000
```

`vercel dev` asks you to log in to Vercel the first time. Opening `index.html` directly shows the page, but the description box and Ask Atlas need `/api/ask` running.

## Deploy

The project deploys on Vercel with no build step. Pushing to `main` deploys to production.

Set these in Vercel → Project Settings → Environment Variables, then redeploy:

| Variable | Required | Notes |
|---|---|---|
| `GROQ_API_KEY` | Yes | The key value only, without quotes |
| `GROQ_MODEL` | No | Defaults to `llama-3.3-70b-versatile` |

## Limits

- `/api/ask` allows 20 requests per visitor per 10 minutes and 300 per day. These limits are kept in memory per serverless instance, so they are a brake rather than a hard cap.
- Groq's free tier allows 8,000 tokens per minute, about 3 to 4 questions a minute across all visitors.
- The API key stays on the server; the page never sees it.
