# Slack Task Agent — Architecture

AI productivity MVP that turns messy Slack-style conversations into structured action plans.

## Overview

```
Browser (React / Next.js App Router)
        │
        ▼ POST /api/analyze  { conversation: string }
        │
┌───────┴───────────────────────────────────────┐
│  src/app/api/analyze/route.ts (Node runtime)   │
│  resolveProvider() → openai | gemini | mock   │
└───────┬───────────────────────────────────────┘
        │
   ┌────┴────┬────────────┐
   ▼         ▼            ▼
OpenAI    Gemini      mock-analyzer
(fetch)   (fetch)     (keyword heuristics)
        │
        ▼
 ConversationAnalysis JSON
 (summary, actionItems, suggestedFollowUp, analyzedAt)
        │
        ▼
 AnalysisResults UI (cards, copy follow-up)
```

## Tech stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS 3 + custom design tokens |
| AI | OpenAI Chat Completions or Gemini REST (optional) |
| Fallback | In-process mock analyzer (no keys required) |
| Database | None — stateless MVP |

## Directory layout

```
src/
├── app/
│   ├── api/analyze/route.ts   # Server-only analysis endpoint
│   ├── layout.tsx             # Root layout, fonts, theme shell
│   ├── page.tsx               # Landing + analyzer sections
│   └── globals.css            # Tailwind + animation utilities
├── components/
│   ├── ConversationAnalyzer.tsx  # Main interactive panel
│   ├── AnalysisResults.tsx       # Summary, cards, follow-up copy
│   ├── AnalysisLoader.tsx        # Staged loading UX
│   ├── DemoSelector.tsx          # Pre-built demo threads
│   ├── ActionItemCard.tsx        # Single task card
│   └── …                         # Hero, Header, theme, etc.
├── lib/
│   ├── mock-analyzer.ts       # Demo-mode analysis (default)
│   ├── ai-providers.ts        # OpenAI + Gemini + normalization
│   ├── analyze-client.ts      # Browser → /api/analyze fetch
│   ├── demo-data.ts           # Product / incident / hiring demos
│   ├── prompts.ts             # Shared system prompt for live AI
│   └── analysis-delay.ts      # Min delay + loading stage labels
└── types/
    ├── analysis.ts            # ConversationAnalysis, ActionItem
    └── api.ts                 # AnalyzeResponseBody, AiProvider
```

## Data model

Core types live in `src/types/analysis.ts`:

- **Priority** — `low | medium | high | urgent`
- **ActionItem** — `id`, `title`, `description`, `owner`, `deadline`, `priority`
- **ConversationAnalysis** — `summary`, `actionItems[]`, `suggestedFollowUp`, `analyzedAt`

The API wraps results in `AnalyzeResponseBody` (`analysis` + `provider`).

## Analysis flow

1. User pastes text or picks a demo in `ConversationAnalyzer`.
2. Client calls `analyzeViaApi()` → `POST /api/analyze`.
3. Server picks provider via `resolveProvider()`:
   - `AI_PROVIDER` env override, else
   - `OPENAI_API_KEY` → OpenAI, else
   - `GEMINI_API_KEY` → Gemini, else
   - **mock** (no keys)
4. Live providers use `ANALYSIS_SYSTEM_PROMPT` and normalize JSON into `ConversationAnalysis`.
5. Mock path uses keyword heuristics (product / incident / hiring / generic).
6. Response renders in `AnalysisResults` with priority-sorted cards.

## Security notes

- API keys are **server-only** (`ai-providers.ts` is never imported from client components).
- No user data is persisted; each request is independent.
- Mock mode requires no external services — suitable for demos and Vercel deploys without env vars.

## Deployment (Vercel)

- Framework auto-detected via `vercel.json`.
- Build: `npm run build` (Next.js static + serverless API route).
- Optional env vars documented in `.env.example`.
- `NEXT_PUBLIC_SITE_URL` or `VERCEL_URL` drives Open Graph metadata in `layout.tsx`.

## Extension points

| Goal | Where to change |
|------|-----------------|
| Add a new AI provider | `src/lib/ai-providers.ts` + `resolveProvider()` |
| Tune extraction prompt | `src/lib/prompts.ts` |
| Add demo scenarios | `src/lib/demo-data.ts` |
| Change mock behavior | `src/lib/mock-analyzer.ts` |
| New UI sections | `src/app/page.tsx` + components |

## Scripts

| Command | Purpose |
|---------|---------|
| `npm run dev` | Dev server (`predev` runs `scripts/ensure-deps.mjs`) |
| `npm run build` | Production build |
| `npm run start` | Serve production output |
| `npm run lint` | ESLint via `eslint-config-next` |
