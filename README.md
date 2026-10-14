# Souqify

Source code of [souqify.fr](https://souqify.fr), a French online shop selling overstock and open-box garden, furniture and home items (many of them vidaXL articles), with free pickup at the warehouse in Bondues (59) or delivery anywhere in France.

Company identifiers in this public copy (`lib/company.ts`) are placeholders.

## What the code does

**Catalogue**
- Products live in MongoDB (Mongoose schemas in `lib/schemas/`). Each article is a single unit with a `sellable` or `sold` status.
- Products are imported from a supplier XLSX sheet (`lib/xlsxImport.ts`). A re-import never wipes fields the sheet does not carry, and never touches the price of an existing product.
- Photos are uploaded through Uploadthing, including a bulk upload route matched to products.
- Customer-facing references (`SQ-XXXXX`) and the decorative barcode are hashed from the product id, so the real EAN is never shown (`lib/internalRef.ts`).

**Pricing and VAT**
- Prices are stored TTC, not computed at render time, so the product page and the basket can never disagree (`lib/pricing.ts`).
- A comparison price is shown only with the date it was observed and is hidden once it is stale.
- VAT is split from the TTC amount by rounding the VAT and deriving the HT, so HT + TVA = TTC always holds (`lib/vat.ts`).

**Delivery**
- Three modes: pickup, Mondial Relay point relais, home delivery. Prices come from weight bands in `lib/shipping.ts`, the single source used by both the cart preview and the server-side checkout.
- Relay eligibility follows Mondial Relay's published weight and size limits.
- A size classifier assigns S/M/L/XL from supplier data, dimensions parsed from the name, or a keyword dictionary, with manual overrides (`lib/sizeClassifier.ts`).
- The delivery promise shown before purchase is computed once (`lib/delivery.ts`) and stored on the order.
- The Mondial Relay and Cocolis API routes are placeholders that return static data.

**Checkout and orders**
- Stripe Checkout. The session is built server-side from database prices and real product weights; the client only sends product ids, delivery mode and customer details.
- The Stripe webhook verifies the signature and is the only place a product is marked sold, using a conditional update so two buyers cannot both get the same unit.
- Confirmation, email verification and new-arrival emails are sent with Resend.

**Accounts**
- Auth.js (NextAuth v5) with Google and email/password (bcrypt), JWT sessions, email verification required before checkout.
- Order history and favorites for signed-in customers.

**Back office**
- Password-protected admin pages for XLSX upload, photos, orders and stats. The password is checked server-side from a request header.
- Order status management per delivery mode and a CSV export of the VAT book for a date range.

**Content and compliance**
- MDX buying guides (`content/guides/`) with Amazon affiliate links, an affiliate disclosure page and click tracking.
- Legal pages (mentions légales, CGV, withdrawal form, privacy, cookies) read company data from `lib/company.ts`.
- `scripts/check-legal.mjs` runs before every build and fails it if a placeholder or an outdated tax statement would reach a public page.
- Google Analytics 4 loads only after explicit consent.

**Maintenance scripts** (`scripts/`): repricing, condition and size classification passes, each a dry run by default that writes a report and only saves with `--apply`.

## Stack

Next.js 15 (App Router), React 18, TypeScript, Tailwind CSS, Framer Motion, MongoDB with Mongoose, Stripe, Auth.js, Resend, Uploadthing, Zustand, Zod, MDX. Tests with Vitest and Playwright (including axe accessibility checks).

## Running locally

```bash
npm install
cp .env.local.template .env.local   # then fill in the values
npm run dev
```

```bash
npm run test:unit   # Vitest
npm run test:e2e    # Playwright
```

`npm run build` runs the legal check first; set `LEGAL_ALLOW_INCOMPLETE=1` to downgrade its failures to warnings during development.
