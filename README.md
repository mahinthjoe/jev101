## What this is
A small Next.js demo app that compares two classification models—“Jev” and a standard chat LLM—by running the same synthetic support emails through both and showing side-by-side outputs, confidence, calibration and stress tests. It's a live-workshop / demo project used to evaluate accuracy, confidence calibration, and runtime behavior of the two model approaches.

### Stack
- **Language(s):** TypeScript (frontend + server), React
- **Framework / runtime:** Next.js (app router, Next 16.x), React 19
- **Notable libraries:** @typesafe-ai/sdk (Jev integration), openai (LLM calls), yaml (data/config parsing). Dev/test: tsx, vitest.

## How it's organized
```
.env.example           example env vars (model/API keys)
AGENTS.md              Next.js agent-related comment block
next.config.ts         Next.js configuration
package.json           scripts/deps (dev, build, start, smoke, dataset, review)
tsconfig.json
vitest.config.ts

app/                   Next.js app directory (routes + pages)
  layout.tsx           top-level layout
  page.tsx             home page listing demos
  globals.css
  icon.svg
  api/                 server endpoints (classification, health, etc.)
  race/                demo route (race test)
  stress/              demo route (stress / side-by-side)
  confidence/          demo route (confidence visualization)
  calibration/         demo route (calibration tools)
  dataset/             dataset viewer / dataset endpoints
  health/              pre-flight / health checks

components/            React UI and client logic
  ui.tsx               shared UI primitives (PageHeader etc.)
  ResultCard.tsx       result rendering for each model
  ModeToggle.tsx
  Nav.tsx
  Tiers.tsx
  format.ts
  useClassify.ts       client-side hook that streams /api/classify results

lib/                   Core TypeScript logic used by server endpoints
  calls.ts             code to call the two models (Jev + LLM)
  race.ts              domain types and decision logic for “race” experiments
  calibration.ts       calibration utilities / metrics
  stats.ts             statistics and aggregation helpers
  retry.ts             retry logic for model calls
  decision.ts, cost.ts, tiers.ts, jsonl.ts
  models/              (folder for model definitions / adapters)

scripts/               utility scripts (scripts/*.ts — e.g., make_dataset.ts, smoke.ts, review.ts)
data/                  dataset and static test data
tests/                 test suite (Vitest)
runs/                  (runtime outputs / example runs)
```

How it fits together:
- The Next.js app (app/) is the runtime: pages render UI and call server endpoints under app/api. The client hook components/useClassify.ts sends POSTs to /api/classify and consumes a streaming response, updating UI tiles (components/ResultCard.tsx) as each model answers.
- Server-side logic lives in lib/: model adapters and decision logic (lib/calls.ts, lib/race.ts) actually invoke Jev and the LLM, aggregate results, compute confidence/calibration (lib/calibration.ts, lib/stats.ts), and stream events back to the client.
- scripts/ contains helper tools to build the synthetic dataset and run quick smoke/review tasks (scripts are TypeScript run with tsx).

## How to run it
Install deps, run dev server (app listens on port 4100 per package.json):
```
npm install
npm run dev        # starts Next dev on :4100 (next dev -p 4100)
# build / production
npm run build
npm run start      # next start -p 4100
# utility scripts
npm run smoke      # tsx scripts/smoke.ts
npm run dataset    # tsx scripts/make_dataset.ts
npm run review     # tsx scripts/review.ts
# tests
npm test           # runs vitest
```
Notes:
- There is a .env.example at the repo root — set the required environment variables (API keys / model endpoints) before running. The app uses both a Jev integration and an OpenAI-compatible LLM, so expect env vars for those connectors.
- The client expects a streaming response from /api/classify (see components/useClassify.ts), so run in an environment that supports streaming responses (Next dev/production server).

## Try asking
- Where is the /api/classify handler implemented and which lib/*.ts files does it call to run Jev vs the LLM?
- How does lib/race.ts represent a PublicDecision and where is the mapping from model raw output → that decision?
- The README is missing — can you add a short Getting Started that documents required env vars from .env.example and what scripts/make_dataset.ts produces?
