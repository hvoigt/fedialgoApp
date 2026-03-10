# CLAUDE.md — FediAlgo Demo App

This file provides context for AI assistants working in this codebase.

**Important**: Keep this file up to date. Whenever you make changes that affect the architecture, directory structure, tech stack, or key patterns described here, update the relevant sections of this file in the same commit.

## Project Overview

**fedialgo-demo-app-foryoufeed** is a React + TypeScript Progressive Web App that serves as the reference frontend for the [`fedialgo`](https://github.com/michelcrypt4d4mus/fedialgo) library — a client-side Mastodon timeline ranking algorithm. It authenticates users via OAuth, fetches their Mastodon home timeline, runs it through the algorithm, and renders a scored, filterable feed.

- **Live URL**: https://hvoigt.github.io/fedialgoApp
- **Core dependency**: `fedialgo` (GitHub package, version-pinned: `github:michelcrypt4d4mus/fedialgo#v1.2.39`)
- **License**: GPL-3.0-only

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI framework | React 18 + TypeScript |
| Build tool | Webpack 5 with ts-loader |
| Styling | Bootstrap 5.2.3 (CDN) + react-bootstrap + Bird UI CSS |
| Routing | React Router DOM v7 (HashRouter — required for GitHub Pages) |
| Mastodon API | `masto` v7 |
| State management | React Context API |
| PWA | Workbox service worker |
| Testing | Jest + React Testing Library (configured, no tests written yet) |
| Linting | ESLint 9.x flat config + typescript-eslint |
| Deployment | GitHub Actions → GitHub Pages (`docs/` directory) |

---

## Development Commands

```bash
npm install           # Install dependencies
npm run start         # Dev server on http://localhost:3000 (hot reload)
npm run start_clean   # Dev server without env var flags
npm run build         # Production webpack build → build/
npm run lint          # ESLint on src/**/*.ts and src/**/*.tsx
npm run tsc           # TypeScript type-check (no emit)
npm test              # Jest test runner
npm run analyze_bundle  # Webpack bundle analyzer
```

**Important**: Bootstrap is loaded via CDN in `src/index.html`, not imported as a module.

---

## Environment Variables

Managed via `dotenv-flow`. Override with `.env.development.local` or `.env.production.local`.

| Variable | Development | Production | Description |
|---|---|---|---|
| `FEDIALGO_DEBUG` | `true` | `false` | Verbose algorithm logging |
| `QUICK_MODE` | `true` | `false` | Shorter feed loading for dev speed |
| `LOAD_TEST` | `false` | — | Load testing mode |

Additional variables may come from the `fedialgo` package's own `.env.example`.

---

## Repository Structure

```
fedialgoApp/
├── src/
│   ├── index.tsx              # React DOM entry point
│   ├── index.html             # HTML template (Bootstrap loaded via CDN here)
│   ├── App.tsx                # Root component: routing, OAuth redirect fix, service worker
│   ├── config.ts              # All app-wide configuration constants and enums
│   ├── types.ts               # Shared TypeScript types (User, App, ModalProps, etc.)
│   ├── birdUI.css             # Bird UI theme styles
│   ├── default.css            # Base Mastodon-style CSS
│   ├── App.css / index.css    # Minor overrides
│   │
│   ├── pages/
│   │   ├── Feed.tsx           # Main timeline page — infinite scroll, algorithm controls
│   │   ├── LoginPage.tsx      # Server selection + OAuth login UI
│   │   └── CallbackPage.tsx   # OAuth callback handler
│   │
│   ├── hooks/
│   │   ├── useAlgorithm.tsx   # AlgorithmProvider context: TheAlgorithm state, timeline, feed ops
│   │   ├── useAuth.tsx        # AuthProvider context: user/server storage, login/logout
│   │   ├── useLocalStorage.tsx # Persistent localStorage hooks for user prefs
│   │   └── useOnScreen.tsx    # IntersectionObserver hook for infinite scroll
│   │
│   ├── components/
│   │   ├── status/            # Toot/status rendering
│   │   │   ├── Status.tsx     # Main status card component
│   │   │   ├── ActionButton.tsx
│   │   │   ├── AttachmentsModal.tsx
│   │   │   ├── MultimediaNode.tsx
│   │   │   ├── Poll.tsx
│   │   │   ├── PreviewCard.tsx
│   │   │   └── ReplyModal.tsx
│   │   ├── algorithm/         # Feed algorithm control UI
│   │   │   ├── WeightSetter.tsx
│   │   │   ├── WeightSlider.tsx
│   │   │   ├── Slider.tsx
│   │   │   ├── FeedFiltersAccordionSection.tsx
│   │   │   ├── BooleanFilterAccordionSection.tsx
│   │   │   ├── FilterAccordionSection.tsx
│   │   │   └── filters/       # Individual filter controls (checkboxes, numeric, etc.)
│   │   ├── helpers/           # Utility UI components
│   │   │   ├── ErrorHandler.tsx    # Global error context (useError hook)
│   │   │   ├── LoadingSpinner.tsx
│   │   │   ├── persistent_checkbox.tsx
│   │   │   ├── JsonModal.tsx
│   │   │   ├── Confirmation.tsx
│   │   │   ├── TooltippedLink.tsx
│   │   │   ├── BugReportLink.tsx
│   │   │   └── MinTootsSlider.tsx
│   │   ├── experimental/      # Experimental/optional features
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── ProtectedRoute.tsx
│   │   ├── ApiErrorsPanel.tsx
│   │   ├── TrendingSection.tsx
│   │   └── TrendingInfo.tsx
│   │
│   └── helpers/               # Pure utility functions (no React)
│       ├── log_helpers.ts     # Logger factory wrapping fedialgo's Logger
│       ├── mastodon_helpers.ts # Mastodon API utilities (MIME types, server validation)
│       ├── number_helpers.ts  # Score formatting, MB constant
│       ├── string_helpers.ts  # Case conversion, locale info, timestamp formatting, Events enum
│       ├── style_helpers.ts   # CSS-in-JS style objects, THEME config, SwitchType enum
│       ├── react_helpers.tsx  # Toot opening, status rendering utilities
│       ├── tooltip_helpers.ts # Tooltip configuration types and objects
│       └── error handling     # Via useError from ErrorHandler context
│
├── .github/workflows/deploy.yml  # CI/CD: builds to docs/ and deploys to GitHub Pages
├── webpack.config.js
├── tsconfig.json
├── eslint.config.mjs
├── package.json
├── .env.development
├── .env.production
└── deploy.sh / tag_and_deploy_release.sh / bump_fedialgo_commit_hash.sh
```

---

## Key Architectural Patterns

### Context Provider Hierarchy

The app uses React Context for global state with this nesting order in `App.tsx`:

```
ErrorHandler (useError hook — global error display)
  └── AuthProvider (useAuthContext hook — user/server/OAuth state)
        └── [Routes]
              └── AlgorithmProvider (useAlgorithm hook — TheAlgorithm + timeline state)
                    └── Feed
```

Access these contexts with the exported hooks:
- `useError()` from `components/helpers/ErrorHandler`
- `useAuthContext()` from `hooks/useAuth`
- `useAlgorithm()` from `hooks/useAlgorithm`

### Routing

Uses `HashRouter` (not `BrowserRouter`) specifically to support GitHub Pages, which cannot serve arbitrary paths. OAuth redirects use a workaround in `App.tsx` to convert `?code=` params before the hash to `#/callback?code=`.

Routes: `/` (Feed, protected), `/login`, `/callback`, `*` (redirects to `/`).

### The `fedialgo` Library

`TheAlgorithm` (from the `fedialgo` package) is the core engine. It is instantiated in `useAlgorithm.tsx` and exposes:
- Timeline fetch + scoring
- Feed filtering (boolean, type, hashtag, language, source, numeric)
- Score/weight configuration

Import from `fedialgo` for types like `Toot`, `ScoreName`, `BooleanFilterName`, `TypeFilterName`, `TagTootsCategory`, `TrendingType`, and the `TheAlgorithm` class itself.

### Logging

Always use the logger factory instead of bare `console.*`:

```typescript
import { getLogger } from "./helpers/log_helpers";
const logger = getLogger("MyComponent");
logger.log("message", variable);
logger.debug("debug info");
logger.warn("warning");
logger.error("error", errorObj);
```

### Persistent User Preferences

Use `persistentCheckbox` from `components/helpers/persistent_checkbox` for checkboxes that survive page reloads. Configuration-level persistent state lives in `useLocalStorage.tsx`.

### Styling

- Bootstrap grid/utilities via class names (loaded via CDN, not import)
- `react-bootstrap` components for UI elements
- CSS-in-JS style objects defined in `helpers/style_helpers.ts` (use `THEME` and named style exports like `blackBackground`, `centerAlignedFlexCol`)
- PurgeCSS removes unused CSS in production builds automatically

---

## Code Conventions

### Naming

| Thing | Convention | Example |
|---|---|---|
| React components | PascalCase | `LoadingSpinner`, `WeightSetter` |
| Helper/utility files | snake_case | `log_helpers.ts`, `persistent_checkbox.tsx` |
| Constants | UPPER_SNAKE_CASE | `REQUIRED_OAUTH_SCOPES`, `HOMEPAGE` |
| Enums | PascalCase name, camelCase members | `GuiCheckboxName.hideSensitive` |
| Types/Interfaces | PascalCase with descriptive suffix | `ModalProps`, `ServerUser`, `AlgoContext` |
| Context hooks | `use` + PascalCase | `useAlgorithm`, `useAuthContext`, `useError` |

### Unused Variables

ESLint enforces that intentionally unused variables/parameters must be prefixed with `_`:

```typescript
// OK — unused param explicitly ignored
function handler(_event: MouseEvent) { ... }

// OK — unused destructure
const { used, _unused } = someObj;

// Error — unused without underscore prefix
function handler(event: MouseEvent) { ... }
```

### TypeScript

- Strict mode is **disabled** (`strict: false` in tsconfig). Avoid introducing `any` unnecessarily but don't force strict patterns onto existing code.
- Target: ES2016, module: esnext
- Use `type` imports when importing only types: `import type { Foo } from "..."` or `import { type Foo } from "..."`
- `ReactState<T>` helper type is defined in `types.ts` as `ReturnType<typeof useState<T>>`

### Component Structure

Functional components only (no class components). Follow the pattern:

```typescript
import { getLogger } from "../helpers/log_helpers";
const logger = getLogger("ComponentName");

export default function ComponentName(props: Props): React.ReactElement {
    // hooks at top
    // derived state / handlers
    // return JSX
}
```

---

## Deployment

**GitHub Actions** (`.github/workflows/deploy.yml`) triggers on push to `master`:

1. `BUILD_DIR=docs NODE_ENV=production npx webpack --mode production`
2. Deploy `docs/` directory to GitHub Pages

**Manual release**: Use `tag_and_deploy_release.sh` to tag and trigger deploy.

**Local fedialgo development**: Use `link_local_fedialgo.sh` to symlink a local clone of the fedialgo package instead of the GitHub version.

**Bundle analysis**: `npm run analyze_bundle` opens the webpack bundle analyzer in a browser.

---

## What This App Does NOT Have

- **No backend server** — entirely client-side; all processing in the browser
- **No password storage** — only OAuth access tokens, stored in `localStorage`
- **No test suites** — Jest is configured but no tests exist yet
- **No SSR** — pure SPA with hash-based routing
