# CogniTrap AI — Insider Threat Detection System (OPEN TRACK)

### 🔗 Project Quick Links
* **Live Application:** [View Live Deployment](temporary-prompt-alder-kwvyazb.vercel.app)
* **Presentation Deck:** [View Project PPT / Pitch Deck](https://docs.google.com/presentation/d/1LhZBUfExZ7oTNMM7sS3Y31xHEixStNWa/edit?usp=sharing&ouid=100842268062784211024&rtpof=true&sd=true)
* **Demo Video:** [Watch 5-Minute Walkthrough](https://drive.google.com/file/d/1scCu9AUwmi_i0gint2y3bm9thHIKPzlN/view?usp=sharing)
* **ARCHITECTURE:** ![COGNITRAP PICTORIAL ARCHITECTURE](<ChatGPT Image Sep 5, 2026, 08_54_20 PM-2.png>)

A front-end prototype of a SOC-style dashboard that demonstrates a six-step insider-threat detection loop:

**Learn → Detect → Decoy → Observe → Confirm → Contain**

It simulates a company with employees, file-system activity, and a behavioral anomaly model. When an account drifts from its normal pattern, the system plants a decoy file. If the account opens the decoy, a canary token fires and the session is contained.

> This is a **simulation**. There is no real identity provider, endpoint agent, or database behind it. All users, logs, and events are generated in the browser.

---

## What the app does

1. **Landing page (`/`)**
   - Explains the detection loop in plain language.
   - Shows a small, honest experiment: 10 scripted insider runs and 10 normal-day runs.
   - Has a sign-in card with demo credentials for an Admin and an Analyst.
   - Clicking **View Demo** scrolls to the loop explanation.

2. **Live SOC Dashboard (`/dashboard`)**
   - Authenticated-looking view reached after signing in.
   - Top bar shows the CogniTrap AI title on the left and the **Admin SOC** profile menu on the right.
   - Four tabs:
     - **Live Behavioral Network Map** — employees grouped by department with risk/status badges.
     - **Insider Threat Radar** — streaming SIEM-style log feed and anomaly index.
     - **LLM Deception Vault** — decoy file inventory + a manual honeypot generator.
     - **Dashboard** — charts for event volume, department risk, identity status, and recent signals.
   - **Simulate Insider Attack** runs a scripted scenario: anomaly → decoy deployment → canary trip → incident summary.

3. **How It Works (`/demo`)**
   - Static walkthrough of the pipeline in more detail.

---

## Architecture

```
┌─────────────────────────────────────────┐
│  Browser                                │
│  • React 19 + TanStack Router           │
│  • Tailwind CSS v4 + shadcn/ui          │
│  • Recharts for analytics charts        │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│  TanStack Start (Vite 7)                │
│  • File-based routing under src/routes/ │
│  • SSR entry wrapped by src/server.ts   │
│  • src/start.ts adds CSRF middleware    │
└─────────────────────────────────────────┘
```

### Routing

TanStack Router generates the route tree from files in `src/routes/`:

| File | URL | Purpose |
|------|-----|---------|
| `index.tsx` | `/` | Landing page with sign-in, loop explanation, and FAQ |
| `dashboard.tsx` | `/dashboard` | SOC dashboard component wrapper |
| `demo.tsx` | `/demo` | Detailed pipeline walkthrough |
| `__root.tsx` | layout | Shared HTML shell, meta tags, Toaster, `<Outlet />` |

`src/routeTree.gen.ts` is auto-generated — do not edit it by hand.

### State & data flow

The demo keeps all state in React on the client:

- `src/components/cognitrap/data.ts` — types, initial employee list, initial decoy list, and a `randomLog()` generator.
- `src/components/cognitrap/Dashboard.tsx` — main dashboard state (`users`, `decoys`, `logs`), simulation orchestration, and tab layout.
- `src/components/cognitrap/NetworkMap.tsx` — renders employees grouped by department.
- `src/components/cognitrap/LogStream.tsx` — renders the scrolling log feed and anomaly index.
- `src/components/cognitrap/DeceptionVault.tsx` — renders decoy cards and the custom honeypot generator UI.

The simulated attack flow in `Dashboard.tsx`:

1. Set a target employee to `suspicious` and bump their baseline score.
2. Log an unusual login event.
3. Generate and display a new decoy file in the vault.
4. Log a critical `READ` event on that decoy.
5. Mark the employee as `trapped` and open the incident modal.

### The one real server call

Everything above is client-side state, deliberately — this app runs on serverless hosting with no persistent filesystem, so a fake "database" would silently lose data between requests, which is worse than not having one. One thing is genuinely server-side: `src/lib/decoyGenerator.server.ts`, called via a TanStack Start server function (`generateDecoyServerFn`) from the custom honeypot generator in the Deception Vault.

- If `ANTHROPIC_API_KEY` is set in the environment, it calls the Anthropic API to generate a contextual decoy name and description from the analyst's department and prompt, and validates the response against a banned-pattern list (no real credential-shaped strings allowed) before using it.
- If the key is absent, the call fails, or the response fails validation, it falls back to a deterministic decoy name automatically — the app never breaks or blocks on this.
- The API key is never sent to or readable from the browser; confirmed by checking the built client bundle contains no reference to the Anthropic API or the key.
- The scripted "Simulate Insider Attack" flow (above) intentionally does **not** call this — a live demo in front of judges shouldn't depend on a network call succeeding. The AI call is reserved for the one place where a user's free-text input actually needs it.

### Styling

- Tailwind CSS v4 is configured through `src/styles.css`.
- Theme variables define a dark SOC palette: background panels, muted text, border lines, and semantic neon accents:
  - `--cyan-neon` — primary action/info
  - `--amber-neon` — warning / simulation
  - `--crimson-neon` — critical / trapped
  - `--emerald-neon` — safe / active
- Custom utilities: `.panel`, `.panel-elevated`, `.glow-*`, and keyframe animations for the live log stream and alert pulse.
- `src/components/ui/` holds shadcn/ui building blocks such as Button, Dialog, Tabs, DropdownMenu, and Slider.

---

## Project structure

```
├── public/                      # Static assets
├── src/
│   ├── components/
│   │   ├── cognitrap/           # Dashboard + sub-panels
│   │   │   ├── Dashboard.tsx
│   │   │   ├── DeceptionVault.tsx
│   │   │   ├── LogStream.tsx
│   │   │   ├── NetworkMap.tsx
│   │   │   └── data.ts
│   │   └── ui/                  # shadcn/ui components
│   ├── lib/                     # Utilities and error handling
│   ├── routes/                  # TanStack file routes
│   │   ├── __root.tsx
│   │   ├── index.tsx
│   │   ├── dashboard.tsx
│   │   └── demo.tsx
│   ├── router.tsx               # Router + QueryClient setup
│   ├── server.ts                # SSR entry wrapper
│   ├── start.ts                 # Start config + middleware
│   └── styles.css               # Tailwind v4 theme and utilities
├── package.json
├── tsconfig.json
└── vite.config.ts
```

---

## Getting started

Requirements:

- Node.js 20+
- npm, pnpm, or bun

Install dependencies:

```bash
npm install
```

Run the development server:

```bash
npm run dev
```

The app runs fully with zero configuration. Optionally, copy `.env.example` to `.env` and set `ANTHROPIC_API_KEY` to enable live AI decoy generation in the Deception Vault — everything else works identically with or without it.

Build for production:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

Lint and format:

```bash
npm run lint
npm run format
```

---

## Demo credentials

The sign-in card has pre-filled demo accounts. You can also type any email/password and submit — there is no real authentication check.

| Role | Email | Password |
|------|-------|----------|
| Admin | `soc.admin@mirrorcorp.io` | `demo-access-2026` |
| Analyst | `analyst1@mirrorcorp.io` | `demo-access-2026` |

---

## Key dependencies

- `@tanstack/react-start` — full-stack React framework / server functions
- `@tanstack/react-router` — type-safe file-based routing
- `@tanstack/react-query` — server-state management
- `react` / `react-dom` — UI library (React 19)
- `tailwindcss` — utility-first CSS framework (v4)
- `recharts` — data visualization
- `lucide-react` — icon set
- `sonner` — toast notifications
- `zod` — schema validation

---

## Notes & limitations

- All data is simulated. No real SIEM, IdP, or database is connected.
- The “model” described is an Isolation Forest conceptually; the scoring in the demo is driven by scripted state changes.
- The experiment numbers on the landing page come from 20 manually-run simulated scenarios, not a live deployment.
- AI-assisted tooling was used for parts of the interface, but the detection logic, experiment framing, and content were reviewed and edited to remove invented quotes or benchmarks.

