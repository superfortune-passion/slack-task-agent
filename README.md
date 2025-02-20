# Slack Task Agent

Turn messy Slack-style conversations into structured action plans — summaries, owners, deadlines, priorities, and ready-to-send follow-ups.

**Works out of the box with no API keys** (demo/mock mode). Add OpenAI or Gemini keys when you are ready for live AI.

---

## Local setup

### Prerequisites

- [Node.js](https://nodejs.org/) **18.18+** (includes npm)

### Commands

```powershell
cd "d:\For dicord"
npm.cmd run dev
```

**PowerShell blocked `npm`?** Use `npm.cmd` instead of `npm`, or run `.\dev.cmd` from the project folder.

Open [http://localhost:3000](http://localhost:3000).

The first `npm run dev` installs dependencies automatically if `node_modules` is missing.

### All scripts

| Script | Purpose |
|--------|---------|
| `npm run dev` | Development server |
| `npm run build` | Production build (run before deploy) |
| `npm run start` | Serve production build locally |
| `npm run lint` | ESLint |

### Verify production build locally

```powershell
npm run build
npm run start
```

---

## Environment variables

**None are required.** Without API keys, `/api/analyze` uses mock analysis (great for demos and hackathons).

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | No | Enables OpenAI analysis |
| `OPENAI_MODEL` | No | Default: `gpt-4o-mini` |
| `GEMINI_API_KEY` | No | Enables Gemini analysis |
| `GEMINI_MODEL` | No | Default: `gemini-1.5-flash` |
| `AI_PROVIDER` | No | Force `openai`, `gemini`, or `mock` |
| `NEXT_PUBLIC_SITE_URL` | No | Canonical URL for Open Graph metadata |

Copy the template:

```powershell
copy .env.example .env.local
```

Provider priority when no `AI_PROVIDER` is set:

1. `OPENAI_API_KEY` → OpenAI  
2. Else `GEMINI_API_KEY` → Gemini  
3. Else → **mock mode**

---

## Deploying to Vercel

### Option A — GitHub (recommended)

1. **Push to GitHub**

   ```powershell
   cd "d:\For dicord"
   git init
   git add .
   git commit -m "Initial commit: Slack Task Agent MVP"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/slack-task-agent.git
   git push -u origin main
   ```

2. **Import on Vercel**  
   Go to [vercel.com/new](https://vercel.com/new) → import your repo → **Deploy**.  
   Framework: **Next.js** (auto-detected). No custom build settings needed.

3. **Optional env vars**  
   Project → **Settings** → **Environment Variables** → add keys from `.env.example` if you want live AI.

### Option B — Vercel CLI

```powershell
npm i -g vercel
cd "d:\For dicord"
vercel
```

Follow prompts. Run `vercel --prod` for production.

### Vercel checklist

- [x] `npm run build` succeeds  
- [x] `vercel.json` sets framework to Next.js  
- [x] API route: `src/app/api/analyze/route.ts`  
- [x] Mock mode works with zero env vars  
- [ ] Add `OPENAI_API_KEY` or `GEMINI_API_KEY` in dashboard (optional)

---

## Future Gemini / OpenAI integration

Live AI is already wired via `POST /api/analyze`. Keys stay on the server (never sent to the browser).

1. Add one key in `.env.local` (local) or Vercel **Environment Variables** (production).
2. Redeploy or restart `npm run dev`.
3. The UI shows which engine ran (OpenAI, Gemini, or mock).

Implementation files:

- `src/lib/ai-providers.ts` — OpenAI & Gemini fetch calls  
- `src/app/api/analyze/route.ts` — server endpoint  
- `src/lib/mock-analyzer.ts` — fallback when no keys  

To force mock mode even with keys set: `AI_PROVIDER=mock`

---

## Features

| Feature | Description |
|--------|-------------|
| Landing page | Hero, feature strip, demo preview |
| Demo selector | Product launch, incident, hiring scenarios |
| Analyze | Paste Slack text → structured action plan |
| Mock mode | No API keys required |
| Dark / light | Toggle with persisted preference |
| Responsive | Mobile-first layout |

## Tech stack

- Next.js 15 (App Router)
- TypeScript
- Tailwind CSS
- No database

## Project structure

```
src/
├── app/
│   ├── api/analyze/route.ts   # Server analysis API
│   ├── layout.tsx
│   └── page.tsx
├── components/
├── lib/
│   ├── mock-analyzer.ts
│   ├── ai-providers.ts
│   └── demo-data.ts
└── types/
```

## License

MIT — use freely for hackathons and MVPs.
