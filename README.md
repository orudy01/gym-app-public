# Rudy's Gym Log

A production fitness platform with public workout history, paid 1-on-1 training bookings, and a full admin back office. Live at **[gymrudy.com](https://gymrudy.com)**.

Built and shipped solo end-to-end: design, frontend, Cloud Functions, payments, email, deploys, and on-call.

---

## What it does

**Public side** — anyone can browse my workout log with weekly/monthly calendar nav, search exercises, view supplement stacks with dosage, and share workout cards as images for social.

**Booking flow** — sign up, verify email, pick a slot (48h–60d out), pay $25 via Stripe Checkout, and I confirm from the admin dashboard. Customer gets a Resend-powered confirmation email; cancellations and refunds are self-service or admin-driven.

**Admin dashboard** — schedule manager (weekly availability), booking manager (confirm / cancel / refund / delete), workout & supplement CRUD with media uploads, and analytics charts.

**AI workout generator** — clients can generate a workout from a prompt; rate-limited (10/hr/user), backed by Groq (LLaMA 3.3 70B) with DeepSeek as a fallback provider.

---

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | React 19, TypeScript, Vite, React Router 6 |
| Styling | Tailwind v4 with `@theme` design tokens |
| State | Zustand (global) + custom hooks (feature-level) |
| Backend | Firebase Cloud Functions (Node 20, TypeScript) |
| Data | Firestore with real-time subscriptions |
| Auth | Firebase Auth (email/password + Google OAuth, email verification gate) |
| Payments | Stripe Checkout + signed webhooks |
| Email | Resend (server-side from Cloud Functions) |
| AI | Groq / DeepSeek |
| Observability | Sentry, structured `logger` utility |
| Testing | Vitest, React Testing Library, Playwright |
| Hosting | Firebase Hosting + Functions |

---

## Engineering decisions worth highlighting

**Server-side email via Cloud Functions, not the client.** Originally used EmailJS from the browser. Migrated to Resend behind Firebase Functions so API keys stay off the client, emails can't be spoofed by a tampered request, and sends are tied to verified payment events instead of trusting the UI.

**Slot reservation during checkout.** Starting Stripe Checkout writes an `abandoned_checkout` doc that holds the slot for 30 minutes. Prevents the double-booking race during payment. Expired reservations get cleaned up automatically.

**Idempotent Stripe webhook.** The handler checks for an existing booking before creating one, so replayed webhook events (Stripe retries, network blips) never create duplicates.

**Email-verification gate enforced server-side.** The checkout function rejects unverified users — the client UI hides the button, but the real check is in the Cloud Function, so it survives a crafted request.

**Guest-booking linking.** If someone books before creating an account, signing up later runs a Cloud Function that matches existing bookings by email and links them to the new UID. Account deletion preserves booking records (admin needs them) while wiping personal data.

**Tailwind v4 migration in flight.** The project started on plain CSS / CSS Modules. I'm migrating incrementally — only files I touch get migrated, app stays functional throughout, no big-bang rewrite. Design tokens live in `@theme` so utility classes reference brand variables instead of hex codes.

**No `any` types, no direct Firestore calls from components, no `console.log`.** Service abstractions in `src/lib/services/`, a typed `logger` utility, and ESLint enforcing it.

---

## Cloud Functions

| Function | Purpose |
|---|---|
| `createStripeCheckoutSession` | Auth + email-verification check, slot availability check, creates Stripe session |
| `stripeWebhook` | Handles `checkout.session.completed`, expirations, refunds — idempotent |
| `sendCustomerConfirmation` | Admin-only, sends confirmation email after manual approval |
| `cancelBooking` | User-initiated cancellation for upcoming bookings |
| `refundPayment` | Stripe refund + frees the slot |
| `linkGuestBookings` | Matches pre-account bookings by email on signup |
| `deleteAccount` | Removes user data, preserves booking records |
| `generateWorkout` | AI generation, rate-limited per user |
| `healthCheck` | Liveness endpoint |

---

## Project layout

```
src/
├── components/      # Shared UI: ui/, layout/, calendar/, charts/, modals/, error/
├── features/        # Feature modules — admin, auth, booking, client, planner, supplements, workouts
│   └── [feature]/
│       ├── components/
│       ├── hooks/
│       ├── types/
│       └── utils/
├── lib/
│   ├── hooks/       # Cross-cutting React hooks
│   ├── services/    # Firebase, Stripe, Zustand store, analytics
│   ├── types/
│   └── utils/       # Validation, error manager, logger, Sentry init
├── pages/           # Route-level components
└── index.css        # Tailwind v4 entry + @theme tokens

functions/src/
├── bookings/        # Booking lifecycle handlers
├── stripe/          # Checkout + webhook
├── email/           # Resend templates and service
├── users/           # Account lifecycle
├── prompts/         # AI workout prompts
└── __tests__/       # Integration tests
```

---

## Testing

- **Vitest + RTL** for unit / component tests
- **Playwright** for E2E
- **Cloud Functions integration tests** cover the parts that have to be right: Stripe webhook → booking creation (including idempotency), email-verification enforcement, slot availability under contention, admin approval → customer email, form validation and sanitization

---

## Skills this project demonstrates

- Shipping and operating a real payment-handling product solo
- React 19 + TypeScript at scale across ~7 feature modules
- Firebase Auth, Firestore security rules, Cloud Functions, Hosting
- Stripe Checkout integration with webhook signature verification, refund flow, and replay-safe handlers
- Transactional email infrastructure (Resend) with server-side gating
- Incremental migrations on a live codebase (CSS → Tailwind v4) without downtime
- Observability and error tracking with Sentry + structured logging
- Practical AI integration with rate limiting and provider fallback

---

## Contact

- **Email:** orudy01@gmail.com
- **Live site:** [gymrudy.com](https://gymrudy.com)
- **GitHub:** [@orudy01](https://github.com/orudy01)
