# Arunachalam approval prototype

Local-only Next.js App Router, TypeScript and Tailwind application. Requires Node 20.9+ (Node 22 LTS recommended).

```powershell
cd D:\project\arunachalam
npm install
npm run dev
```

Open http://127.0.0.1:3000. Production checks: `npm test`, `npx tsc --noEmit`, `npm run build`. Run a built version with `npm start`.

## Walkthrough

1. Switch Japanese/English in the header; the current page and cart remain.
2. Open Menu, add butter chicken with medium spice and naan. Review/edit the cart.
3. Checkout with fictional name and phone, select a live Japan-time pickup slot. Leave payment-failure unchecked. No card fields or real charges exist.
4. Keep the tracking page open. Open `/admin` in another tab or browser and enter the explicitly labelled demo dashboard.
5. Accept the new order (optionally adjust pickup). Payment changes from authorized to captured. Progress preparing → ready → collected. The tracking page polls every three seconds.
6. Try payment failure at checkout, reject a pending order, or cancel an accepted order for a simulated refund. Test availability and pause controls under the owner dashboard. Configure acceptance timeout to one minute to demonstrate expiration.
7. Enable sound in the dashboard. Only orders arriving after its initial load sound. Keep it open; background push is not supported. Reset asks for confirmation and restores the seed menu/settings.

## Persistence and architecture

`lib/store.ts` is the data-access boundary. Integer-yen prices are recalculated from the seed menu; order lines snapshot names/options/prices. A synchronous atomic-replace JSON file in `data/demo.json` keeps the modest demo dependency-free and shares orders across sessions. Cart and language alone are stored in browser localStorage. Run **one local server process**; this file adapter is not suitable for multi-process production. Replace it with transactional PostgreSQL (including unique idempotency keys and provider event records) before Railway deployment. No credentials or external services are required. Optional `DEMO_DATA_DIR` changes the data directory.

Settings use daily opening hours in Asia/Tokyo (default 11–22), 15-minute pickup slots, 30-minute preparation/cutoff buffers, and a 15-minute acceptance timeout. Expiry is evaluated on server reads and owner actions. No background worker is needed for this demonstration. Existing history is retained until reset. Daily totals use Japan creation date and exclude refunded payments.

## Demo limitations and approvals

- All payments, authorizations, capture, releases and refunds are simulated. Verify the intended authorization/capture window and cancellation/refund behavior with a real JPY payment provider.
- Demo owner access is deliberately not secure authentication. Bind to loopback and use fictional customer data. Do not expose publicly.
- Food artwork is CSS illustration, not real restaurant photography. Menu, prices, opening hours, restaurant name and Japanese translations require client approval. Address/phone await supply; no reviews, certifications or allergen assurances are invented.
- Production checklist: approved bilingual menu/content and allergen information; verified restaurant details; secure owner authentication/authorization and audit trail; payment webhooks/reconciliation and idempotency; privacy, cancellation, refund and statutory commerce policies; database migrations/backups/restore tests; notification delivery, retries and sound testing; accessibility/security review and responsive device QA.
- Domain is unconfirmed. No domain purchase, real notifications, public deployment, native app, delivery, POS or QR ordering is included.

## Verification performed

- Four server tests pass: authoritative totals/options, idempotent duplicate checkout, failure/pause/sold-out rejection, valid transitions/refund, rejection and timeout release.
- Production webpack build and its TypeScript check pass. Webpack is explicitly selected because Turbopack crashed in this Windows environment.
- Browser walkthrough passed: Japanese → English, medium butter chicken + garlic naan, ¥1,730 cart, guest checkout, authorized payment, owner acceptance/capture, preparing → ready → collected, and automatic customer status update.
- Desktop screenshot inspected. Responsive CSS is implemented; the browser viewport override did not change the actual viewport, so phone-size visual verification remains outstanding.
- Windows sandbox blocked atomic rename and some build output operations; tests, final build and running server were verified outside that sandbox.
