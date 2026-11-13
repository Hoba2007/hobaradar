# Hoba Radar — هوبا رادار | Tech News Agency

A complete, production-ready bilingual (English / Arabic RTL) technology news platform built with **Next.js 16 (App Router) + TypeScript + Tailwind CSS 4 + shadcn/ui + Lucide Icons + Firebase**.

## Brand

| Token | Value |
| --- | --- |
| Background | `#FFFFFF` pure white |
| Cards / sections | `#F8FAFC` light gray |
| Radar Orange (primary) | `#FF6B00` |
| Dark Charcoal (typography) | `#1E293B` |
| Logo | `public/logo.jpeg` (navbar + footer) |

## Routes

| Route | Description |
| --- | --- |
| `/` | Front page: breaking-news ticker, hero (top story + secondary), Radar Pulse (trending), category tabs, studio promo, newsletter |
| `/article/[slug]` | Full bilingual article: structured body (headings, quotes, lists, stat cards), share buttons, related coverage, "Open in Video Studio" hand-off |
| `/studio` | **Creator Video Studio** — generate 60-second vertical video scripts (Hook 0–6s + Core Breakdown 6–48s + CTA 48–60s), export `.SRT` subtitles, adjustable-speed teleprompter (0.5×–3×, mirror mode, 9:16 preview, fullscreen, countdown) |
| `/api/studio/script` | Server-side script generation (z-ai-web-dev-sdk LLM) with a deterministic template fallback, zod validation and per-IP rate limiting |

Hand-off deep link: `/studio?article=<slug>` pre-loads any article into the studio.

## Categories

- AI & Automation — الذكاء الاصطناعي والأتمتة
- Aviation & Defense — الطيران والدفاع
- Mechatronics & Robotics — الميكاترونيات والروبوتات
- Gadgets & Software — الأجهزة والبرمجيات

## Firebase integration

`.env.local` ships pre-configured with the Hoba Radar project (`hoba-radar`):

- **Analytics** — page-view tracking on every route (`src/lib/firebase.ts`, booted by `src/components/site/firebase-boot.tsx`)
- **Cloud Messaging (FCM)** — `public/firebase-messaging-sw.js` handles background pushes (notification click routes to the linked story). The navbar bell (`Breaking Alerts`) requests permission, registers the SW, resolves the token and stores it in the `fcm_tokens` Firestore collection. Foreground messages surface as toasts.
- **Firestore** — newsletter subscribers are written to `subscribers`; all writes are best-effort with graceful fallback so the site stays fully functional even before Firestore rules are opened.

### To enable push notifications end-to-end

1. Firebase Console → Project Settings → **Cloud Messaging** → Web Push certificates → copy the **VAPID key**.
2. Paste it into `.env.local` as `NEXT_PUBLIC_FIREBASE_VAPID_KEY=...` and restart the dev server.

### Firestore security rules

Recommended rules are in `firestore.rules`. Deploy them with the Firebase CLI:

```bash
firebase deploy --only firestore:rules
```

They follow the principle of least privilege: subscribers/tokens are append-only, articles are read-only to clients, everything else is denied.

## Content layer

Articles live in `src/lib/data/articles-a.ts` and `articles-b.ts` (typed, bilingual, structured blocks) and are aggregated in `src/lib/data/articles.ts`. The 12 seeded stories are realistic editorial demo content across all four categories — replace or extend them there, or mirror the same document shape in a Firestore `articles` collection when moving to a live newsroom.

## Security notes

- Firebase web keys are public-by-design; access is controlled by Firestore rules (`firestore.rules`) and authorized domains in the Firebase console.
- `.env.local` is git-ignored; `CRON_SECRET` and `ADMIN_EMAIL` are reserved for server-side jobs and admin flows.
- The studio API validates input with zod, rate-limits by IP (12 req/min) and never exposes provider keys to the client.

## Development

```bash
bun install
bun run dev        # http://localhost:3000
bun run lint       # ESLint (passes clean)
```

Production:

```bash
bun run build && bun run start
```
