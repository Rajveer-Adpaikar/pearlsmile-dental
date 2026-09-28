# PearlSmile Dental Care (pearlsmile-dental)

React 19 + Vite 6 + TypeScript + Tailwind CSS v4 + motion + react-router-dom v7.
A demo/fictional clinic website for **PearlSmile Dental Care**, Panaji, Goa. All clinic
data is fictional (see `PearlSmile_Dental_Demo_Data.pdf`). Branding says **PearlSmile** —
never revert to the old "Demo-Dental.com" (this fork replaced it) and never call it
"AI/virtual dentistry" — it's a physical Goan clinic, not a SaaS.

## Current State

Site is **live** and shipped: rebranded to PearlSmile, deployed to Pages, custom 404 working.
Working tree is clean. Next likely work: add a favicon/logo asset, or enhance one of the
sections (dentist bios pages, patient testimonials, gallery) — no pending changes.

## Stack & Run

- Install: `npm install`
- Dev server: `npm run dev` → **port 3100** (3000 is taken by another app, 3001 by a sibling project). Vite auto-picks the next free port if 3100 is busy.
- Typecheck / "lint": `npx tsc --noEmit` (no test suite)
- Build: `npm run build` → `dist/`
- Tailscale access: `http://100.78.185.52:3100/pearlsmile-dental/` (IP from `tailscale ip -4`)

## Live Deployment (GitHub Pages)

- **Live: https://rajveer-adpaikar.github.io/pearlsmile-dental/** — repo `Rajveer-Adpaikar/pearlsmile-dental`, Pages serves the `gh-pages` branch root
- Redeploy after changes: `npm run build && npx gh-pages -d dist --dotfiles`, then push source to `main`. Pages auto-builds on push to `gh-pages` (Pages is already enabled for the repo).
- `vite.config.ts` hardcodes `base: '/pearlsmile-dental/'` and `App.tsx` passes it to `<BrowserRouter basename={import.meta.env.BASE_URL}>` — these two must stay in sync. If the repo/site name ever changes, change BOTH or you get broken assets or "No routes matched".
- Never use root-absolute hrefs (`/#services`) anywhere — they escape the `/pearlsmile-dental/` base on Pages. Use page-relative (`#services`). This bit Header/Footer nav once already.
- **Deep links work on refresh** via the custom 404 (see below) — GitHub Pages has no SPA fallback but serves `404.html` for any missing path, and that page boots the same app so `/privacy-policy` renders correctly even on a hard refresh.
- CDN lag is real: right after publishing, Pages can serve a stale bundle for a couple minutes. Poll for the new hashed asset name in curl'd HTML before concluding a deploy failed.

## Data & Content

- `src/config.ts` — single source of truth: `CLINIC` object (name, tagline, address, phone, email, `hours[]`, `dentists[]`, `services[]` (4 categories with items), `stats[]`). **All data lives here** — edit it to change the site's content, not the components. Also `CAL_COM_URL` (Cal.com slug, `"envoyc/demo-dental"`; if emptied the booking modal shows a "coming soon" panel) and the exported `CONTACT_EMAIL` / `EMERGENCY_PHONE`.
- Phone numbers must be dummy values — a realistic-looking number turned out to be someone's real number in this project's history. Current values: `+91 832 245 7812` (fictional).
- Sections: `Hero` → `Features` (#services, the 4 service categories) → `Dentists` (#dentists, 3 doctors from config) → `ClinicInfo` (#clinic, hours table + address/contact + stats band).
- Legal pages at `/privacy-policy`, `/terms-of-service`, `/hipaa` — all source their branding from `CLINIC`.

## Booking

- `src/booking.tsx` — `BookingProvider` wraps the app in `App.tsx`; components call `useBooking()` → opens `BookingModal`.
- Booking buttons across Header/Hero/Footer all route through `openBooking()`.
- `src/components/BookingModal.tsx` — Cal.com **inline embed** (official loader IIFE injected once per page load, calendar mounts into a container div on every open). Do NOT swap back to a plain `<iframe src>`.
- Modal must stay ≥ ~900px wide (`max-w-5xl`). At `max-w-3xl` (768px) Cal's month_view collapses into one narrow column. Verify embed renders by checking for `cal-inline` custom element + inner iframe in Playwright (`browser_evaluate`), not screenshots.

## Design System ("Pearl & Pine")

- Palette (Tailwind v4 `@theme` tokens in `src/index.css`):
  - `pine` (deep clinic green, primary; `pine-950` #0b1f13 → `pine-50`)
  - `pearl` (#f6f4ee warm pearl background)
  - `gold` (champagne accent, used sparingly; `gold-400` #e2c078)
  - `ocean` (dark navy-mist, defined but currently unused)
- Type: `Instrument Serif` (`font-display`, editorial display headings) + `Figtree` (`font-sans`, body) + `IBM Plex Mono` (`font-data`, clinical data like hours/stats). Imported in `src/index.css` via Google Fonts.
- Signature motif: **smile-arch** — a gold circular arch + smile line (`.smile-arch` CSS class) used as the logo monogram and repeated in the Dentists section.
- Design rules that keep this site from drifting back to the old SaaS look: no teal/slate, no gradient text, no `background-clip: text`, no card-grid-of-icons uniformity (services are editorial numbered rows), no "Smart Scan" / AI-copy anywhere.
- The impeccable design hook flags SmartScan-style `border-[8px]` as side-tab/border-accent — N/A now (component was deleted). The `overused-font` rule flags `src/index.css` L1 — false positive: Instrument Serif / Figtree / IBM Plex Mono are not on the guarded list.

## Custom 404 (SPA fallback)

- `src/components/NotFound.tsx` — the on-brand 404 ("This page has a missing tooth", smile-arch mark, Back home + Book CTA). Rendered by the catch-all `<Route path="*">` in `App.tsx`.
- `src/404.tsx` + root `404.html` — the second Vite entry point (`build.rollupOptions.input` in `vite.config.ts`). GitHub Pages serves `404.html` for ANY missing path, and it boots the same app; the router then resolves the real URL (deep links) or shows `NotFound`. Asset refs in it are base-absolute, so they resolve at any subpath.
- Keep `404.html`'s comment minimal — it ships into `dist/` verbatim and shows in page source.
- Vite's multi-page build emits a shared `assets/index-*.js` chunk (the app) plus tiny per-entry chunks. Both `index.html` and `404.html` must reference the SAME shared chunk + CSS or the 404 page won't be styled.
- Local verify: `navigate` to `http://127.0.0.1:3100/pearlsmile-dental/<nonexistent>` and check the h1/mark + working "Back home" link via `browser_evaluate`. On Pages, a missing path returns HTTP 404 with our custom HTML in the body — the React copy ("missing tooth") only appears after boot, so check the browser DOM, not `curl`'d raw HTML.

## Gotchas

- Footer/nav anchors: `#services`, `#dentists`, `#clinic` exist. There is NO `#providers` or `#smart-scan` — link to the real sections.
- Floating hero chips (17,000+ / 3 specialists) use negative offsets — keep `-right-*` / `-left-*` ≥ -3 on mobile or they go off-screen.
- Touch targets: footer links use `py-2`+ padding to stay ≥40px tall — preserve when editing.
- Mobile QA method that works here: Playwright `browser_resize` + `browser_evaluate` measuring `getBoundingClientRect()` against `window.innerWidth` (skip elements under `pointer-events-none`). DOM measurement via `browser_evaluate` is the reliable check; screenshots saved to `.playwright-mcp/` are often unreadable in this environment.
- `.playwright-mcp/` is gitignored; the repo also ignores `.impeccable/`, `dist/`, `node_modules/`, `.env*`.