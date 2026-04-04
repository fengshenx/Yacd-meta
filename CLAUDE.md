# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YACD (Yet Another Clash Dashboard) is a web UI for managing [Clash](https://github.com/yaling888/clash) proxies. It communicates with Clash via its REST API.

## Commands

```sh
pnpm i          # Install dependencies
pnpm start      # Start dev server at http://127.0.0.1:3000
pnpm build      # Build for production (outputs to public/)
pnpm serve      # Preview production build
```

## Architecture

### Path Aliases
- `~` maps to `src/` (e.g., `~/store/proxies` → `src/store/proxies.tsx`)

### State Management
Uses **Recoil** for global state. Key stores:
- `src/store/app.ts` — App-level config (theme, API settings, latency test URL)
- `src/store/proxies.tsx` — Proxy list, groups, latency checks, switching
- `src/store/connections.ts` — Active connections
- `src/store/logs.ts` — Log entries
- `src/store/configs.ts` — Clash configuration
- `src/store/rules.ts` — Rule provider rules
- `src/store/modals.ts` — Modal visibility state

### Routing
React Router v6 with **lazy-loaded pages** (code splitting). Routes defined in `src/app/router.tsx`:
- `/proxies` — Proxy list and groups (default redirect from `/`)
- `/connections` — Active connections table
- `/configs` — Clash config editor
- `/logs` — Log viewer
- `/rules` — Rule providers
- `/about` — About page
- `/backend` — Backend/server selection
- `/style` — Style guide (dev only)

### API Layer
Clash API calls go through `src/api/` files. Each wraps `getURLAndInit` from `src/misc/request-helper.ts` which constructs the fetch options including the `Authorization` header (secret) when configured.

Key API modules: `configs.ts`, `connections.ts`, `logs.ts`, `proxies.ts`, `rules.ts`, `traffic.ts`, `memory.ts`, `rule-provider.ts`, `version.ts`

### Component Structure
- `src/pages/` — One file per route, imports from `src/components/`
- `src/components/` — Shared UI components, some with sub-components
- `src/components/proxies/` — Proxy-specific components (Proxy, ProxyGroup, ProxyList, etc.)
- `src/components/about/`, `src/components/proxies/` — Sub-namespaced components

### Build Output
Production build outputs to `public/` (not the Vite default `dist/`). This is intentional for compatibility with existing deployment setups.

### PWA
Uses `vite-plugin-pwa` with an inject-manifest strategy. Service worker source is `src/sw.ts`.

## No Tests
This project has no test suite. Do not add test files or attempt to run tests.

## TypeScript
Strict mode is not enabled. The tsconfig allows JSX, ESNext, and uses `allowSyntheticDefaultImports` and `esModuleInterop`.
