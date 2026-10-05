# MOMENTRA Professional Starter

Responsive studio dashboard and deployable backend starter for the Momentra wedding-photo delivery service.

## What works in preview mode

- Responsive studio dashboard, event creation, branding/settings screens, upload and guest-flow previews.
- Events are saved to this browser only. Chosen files stay on this device and are not transmitted.
- Guest face matching and payment checkout are deliberately disabled until their secure providers are configured.

## What is implemented for connected mode

- Supabase email/password authentication.
- Studio profile and event tables protected with Row Level Security (RLS).
- Private Supabase Storage bucket for original JPEG, PNG, and WebP uploads, with per-studio path policies.
- Authenticated Stripe Checkout Edge Function for a studio subscription or one-time event payment.
- Signed Stripe webhook receiver that records subscription/payment status server-side.
- Short-lived signed URLs for studio-side gallery previews.

The deployable site is a static single-page app so it can be hosted on GitHub Pages. It never contains a Supabase service-role key or Stripe secret. Service credentials must be added as Supabase Function secrets.

## Setup

1. Create a Supabase project. In **SQL Editor**, run `supabase/schema.sql`.
2. In Supabase **Authentication → URL Configuration**, set the site URL to the published website and add its exact URL to the redirect allow list.
3. Copy the project's **Project URL** and **Publishable key** into `config.js`. The publishable key is designed for browser use; keep RLS enabled. Never use a secret/service-role key in this file.
4. In **Storage**, confirm that `momentra-originals` is private. The SQL migration creates the bucket and restrictive upload/read/delete policies.
5. Create Stripe products and prices for a recurring studio plan and a one-time event pass. Set these Supabase Function secrets:
   - `STRIPE_API_KEY`
   - `STRIPE_WEBHOOK_SECRET`
   - `STRIPE_STUDIO_PRICE_ID`
   - `STRIPE_EVENT_PRICE_ID`
   - `APP_BASE_URL` (full published URL, including `/momentra-demo/` if using the project site)
   - `ALLOWED_APP_ORIGINS` (scheme + host only, e.g. `https://username.github.io`)
6. Deploy `create-checkout` and `stripe-webhook` Edge Functions. Configure a Stripe webhook for `checkout.session.completed`, `customer.subscription.updated`, and `customer.subscription.deleted` pointing at the Supabase `stripe-webhook` endpoint.
7. Publish the root `index.html` and `config.js` on GitHub Pages. Add `supabase/` source files to the repository to keep the backend deployment source with the site.

## Important remaining production integrations

This package does **not** yet implement biometric face indexing/search, automatic natural white-balance processing, RAW decoding, guest photo matching, downloadable guest archives, email delivery, or a staff admin console. Do not present those as active services or upload real guest selfies. They need a separately deployed image-processing worker and a chosen face-search provider, plus consent, retention/deletion, abuse controls, and provider/data-region review. The UI makes this limitation explicit.

Before inviting photographers, complete those integrations, add data deletion/export requests, configure backup/monitoring, and have local privacy/payment terms reviewed. GitHub Pages hosts the demo UI only; Supabase provides the backend once configured. Free service quotas and payment processing fees are controlled by those providers and may change.
