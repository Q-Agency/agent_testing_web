# Project Constitution — Web Frontend

## Metadata

| Field | Value |
|-------|-------|
| Project Name | AutoMarket — Car Selling Platform |
| Project Type | GREENFIELD |
| Status | VALID |
| Author | AI Agency OS / BA Agent |
| Date | 2026-03-12 |
| Stack | Web |
| Label | Web |

---

## Overview

AutoMarket web frontend is the primary buyer and seller interface for the car selling platform. It provides vehicle search and discovery, listing creation with multi-image upload, buyer-seller messaging, offer management, dealership storefronts, and an admin moderation dashboard. The app communicates exclusively with the FastAPI backend via REST APIs.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | TypeScript (strict mode) |
| Framework | React + Next.js (App Router) |
| Styling | Tailwind CSS |
| Component Library | shadcn/ui |
| State Management | TanStack Query (server state) + Zustand (client state) |
| Forms | React Hook Form + Zod |
| HTTP Client | ky |
| Testing | Vitest (unit) + Playwright (e2e) |
| Linting | ESLint (flat config) + typescript-eslint |
| Formatting | Prettier |
| Package Manager | pnpm |

---

## Environments

| Environment | Purpose | Config Source |
|-------------|---------|--------------|
| development | Local dev, mock API or local backend | `.env.development` |
| staging | Pre-production, real backend staging | `.env.staging` via CI |
| production | Live | `.env.production` via CI |

Client-side env vars prefixed with `NEXT_PUBLIC_`. Server-side env vars (API keys, internal URLs) never exposed to the browser. Switching is automatic via Next.js env file convention.

---

## Coding Standards

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files (components) | PascalCase | `VehicleCard.tsx` |
| Files (hooks) | camelCase, `use` prefix | `useVehicleSearch.ts` |
| Files (utils/lib) | camelCase | `formatPrice.ts` |
| Components | PascalCase | `VehicleListingGrid` |
| Hooks | camelCase, `use` prefix | `useVehicleFilters()` |
| Functions | camelCase | `formatMileage()` |
| Variables | camelCase | `listingCount` |
| Constants | UPPER_SNAKE_CASE | `MAX_UPLOAD_SIZE_MB` |
| Types / Interfaces | PascalCase, no `I` prefix | `Vehicle`, `SearchFilters` |
| Route segments | kebab-case | `/vehicle-listings/[id]` |

### File Structure

```
src/
├── app/                  # Next.js App Router (layouts, pages, route handlers)
├── components/
│   ├── ui/               # shadcn/ui primitives
│   ├── shared/           # Cross-feature (Pagination, EmptyState, ErrorBoundary)
│   └── [feature]/        # Feature-specific components
├── hooks/                # Custom hooks (data fetching, auth, media)
├── lib/
│   ├── api/              # ky client instance + typed API modules
│   ├── supabase.ts       # Supabase client init
│   ├── validators/       # Zod schemas for forms
│   └── utils/            # Formatters, helpers
├── types/                # Shared TypeScript types
├── constants/            # App-wide constants
├── styles/               # Tailwind globals, CSS custom properties
e2e/                      # Playwright e2e tests
```

### Patterns to Use

- **Server Components by default** — only add `"use client"` when the component needs interactivity, hooks, or browser APIs.
- **TanStack Query for all API data** — no `useEffect` + `useState` for data fetching. `useQuery` for reads, `useMutation` for writes.
- **Zustand for client-only state** — search filter UI state, UI preferences. Keep stores small and focused.
- **Zod schemas shared with React Hook Form** — single source of truth for form validation. Extract types with `z.infer<>`.
- **Colocation** — components used by one route live next to that route. Shared components in `components/shared/`.
- **Composition over configuration** — compound component patterns over massive prop APIs.
- **Error boundaries per feature** — wrap each major section in its own error boundary.
- **Optimistic updates** — for offers and messages, update UI immediately and reconcile on server response.
- **Suspense for loading states** — `<Suspense>` boundaries with skeleton components. No spinner waterfalls.

### Patterns to Avoid

- **`any` type** — never. Use `unknown` and narrow, or define proper types.
- **`useEffect` for data fetching** — use TanStack Query.
- **Prop drilling beyond 2 levels** — use context, Zustand, or composition.
- **Inline styles** — Tailwind classes only. Exception: truly dynamic values.
- **Barrel index files** — import directly from source files to avoid bundle bloat.
- **Non-accessible markup** — all interactive elements keyboard-navigable, all images have `alt`, use semantic HTML.
- **Hardcoded user-facing strings** — use constants or a key-value map, extractable for future i18n.

---

## Git Conventions

| Convention | Value |
|------------|-------|
| Branch Naming | `feature/AM-{ticket}-short-desc`, `fix/AM-{ticket}-short-desc`, `chore/`, `hotfix/` |
| Commit Format | Conventional Commits: `feat(search): add filter panel`, `fix(auth): handle token refresh race` |
| PR Size Guideline | ≤400 lines changed. Larger PRs must be split or justified. |
| Default Branch | `main` |
| Merge Strategy | Squash merge to `main` |

---

## Error Handling

- Global ky error interceptor catches API errors and maps them to typed `ApiError` objects.
- TanStack Query `onError` callbacks handle per-query error display.
- Error boundaries per feature section — one broken feature doesn't crash the page.
- Network errors (offline, timeout) show a retry-able toast notification, not a full error page.
- 401 responses trigger silent token refresh via Supabase client. If refresh fails, redirect to login.
- 429 responses show user-friendly "slow down" message with retry timer.
- Form validation errors shown inline per field via React Hook Form + Zod.
- Never show raw error messages from the backend to users. Map error codes to user-friendly text.

---

## Logging & Observability

| Convention | Value |
|------------|-------|
| Log Format | `console.error` / `console.warn` in dev; structured reporting service in production |
| Log Levels | `error` (crashes, API failures), `warn` (degraded UX, retries), `info` (key user actions for analytics) |
| What Gets Logged | Unhandled errors, API failures with status/endpoint, critical user actions (listing created, offer submitted) |
| What Never Gets Logged | Auth tokens, passwords, PII, full API response bodies |
| Tracing | Error boundary catches feed into error reporting service. Request IDs from backend propagated for correlation. |

---

## Testing Standards

| Requirement | Value |
|-------------|-------|
| Minimum Coverage | 70% (components + hooks) |
| Unit Tests Required | YES |
| Integration Tests Required | YES (Playwright e2e for critical flows) |

### Test File Conventions

- Unit tests colocated: `VehicleCard.tsx` → `VehicleCard.test.tsx` in same directory.
- Hook tests next to hook file.
- E2E tests in `e2e/` at project root.
- `@testing-library/react` for component tests. Test behavior, not implementation details.
- MSW (Mock Service Worker) for API mocking. Never mock `fetch` directly.
- E2E tests cover critical journeys: search → view → offer → messaging.
- Naming: `describe("VehicleCard")` → `it("displays price formatted with currency")`.

---

## API Standards

| Convention | Value |
|------------|-------|
| HTTP Client | ky instance with base URL from `NEXT_PUBLIC_API_URL` |
| Auth | JWT in `Authorization` header, refreshed via Supabase client |
| Error Handling | Global interceptor in ky, per-query error boundaries |
| Response Types | Typed manually in `types/` or generated from backend OpenAPI spec |
| Caching | TanStack Query with per-resource stale times |

All API modules live in `lib/api/`. Each module exports typed functions. Hooks in `hooks/` wrap API calls with TanStack Query. Specific endpoints and response shapes defined by SA Agent in `design.md`.

---

## Security Standards

- No secrets in client-side code. All sensitive operations through the backend.
- Supabase Auth handles JWT lifecycle. Access tokens in memory, not localStorage.
- All user-generated content rendered with proper escaping. No `dangerouslySetInnerHTML` without DOMPurify sanitization.
- Image uploads validated client-side (type, size) before backend validation.
- CSP headers configured in Next.js middleware.
- No sensitive data in URL parameters.
- Rate-limit feedback: user-friendly message on 429 responses.
- Dependency audit via `pnpm audit` in CI.

---

## Accessibility Standards

- Lighthouse Accessibility score ≥ 95 enforced in CI.
- All interactive elements keyboard-navigable with visible focus indicators.
- All images have descriptive `alt` text. Decorative images use `alt=""`.
- Semantic HTML throughout (`<nav>`, `<main>`, `<article>`, `<section>`).
- Color contrast meets WCAG 2.1 AA minimum (4.5:1 for normal text).
- Form fields have associated `<label>` elements. Error messages linked via `aria-describedby`.
- Modal dialogs trap focus and are dismissible via Escape.
- Screen reader testing for critical flows (search, listing creation, offer submission).

---

## Documentation Standards

| What | Requirement |
|------|-------------|
| Components | Props documented via TypeScript types. Complex components get a brief JSDoc comment explaining purpose. |
| Hooks | JSDoc with description, params, return value, usage example for non-trivial hooks. |
| API modules | Each exported function has JSDoc describing the endpoint it calls and what it returns. |
| Architecture decisions | ADR in `docs/adr/` for non-trivial choices (state management, SSR strategy, etc.). |

---

## Quality Gates

Before merge, code must pass:

- [ ] `pnpm typecheck` — zero TypeScript errors (strict mode)
- [ ] `pnpm lint` — zero ESLint errors
- [ ] `pnpm format:check` — Prettier verified
- [ ] `pnpm test` — all unit tests pass
- [ ] Coverage ≥ 70% on changed files
- [ ] E2E smoke tests pass for affected journeys
- [ ] No `any` types without justification comment
- [ ] No `eslint-disable` without justification comment
- [ ] Lighthouse: Performance ≥ 90, Accessibility ≥ 95
- [ ] All interactive elements keyboard-accessible
- [ ] PR description includes context, screenshots, and testing notes

---

## Open Questions

- [ ] SSR / SSG / ISR strategy for listing pages (affects caching, SEO, and data freshness architecture)
- [ ] Real-time messaging transport (Supabase Realtime vs. backend WebSocket — affects state sync model)
- [ ] OpenAPI codegen for frontend types vs. manual type maintenance
