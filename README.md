# MOMENTRA — Wedding Photo Delivery

A mobile-friendly studio dashboard prototype for private wedding galleries, event uploads, Supabase authentication and Razorpay studio subscriptions.

## Current setup

- Supabase project **Momentra** is active in Mumbai.
- Database schema and private `momentra-originals` storage bucket are configured.
- The studio account, event and photo tables use Row Level Security.
- Razorpay subscription plans are set to ₹999 monthly, ₹2,699 quarterly and ₹9,999 yearly.
- Supabase Edge Functions `create-subscription`, `verify-subscription` and `razorpay-webhook` are deployed.
- `config.js` contains only the public Supabase URL and publishable key. Never add a Supabase secret/service-role key or Razorpay key secret to this file.

**Payments are not live yet.** Add the Razorpay API key ID, API key secret and webhook secret as Supabase Edge Function secrets. Razorpay has also requested a publicly accessible service/pricing website before account activation. Keep the account in test mode until Razorpay approves the site and the secrets/webhook are configured.

## Launch prices and plans

- Monthly: ₹999 — `plan_TkLDqGzLUm7N9x`
- Quarterly: ₹2,699 every 3 months — `plan_TkLEUumVzaVKZh`
- Yearly: ₹9,999 per year — `plan_TkLEwHXFGiayb9`

Checkout is configured to show cards, netbanking and UPI, and hide Pay Later, eMandate, EMI, Cardless EMI and wallets.

## Supabase function secrets still required

Add these in **Supabase → Edge Functions → Secrets**. Never publish these values in GitHub or the browser app.

- `RAZORPAY_KEY_ID`
- `RAZORPAY_KEY_SECRET`
- `RAZORPAY_WEBHOOK_SECRET` — use the same value when creating the Razorpay webhook
- `RAZORPAY_PLAN_MONTHLY=plan_TkLDqGzLUm7N9x`
- `RAZORPAY_PLAN_QUARTERLY=plan_TkLEUumVzaVKZh`
- `RAZORPAY_PLAN_YEARLY=plan_TkLEwHXFGiayb9`
- `ALLOWED_APP_ORIGINS=https://joshidhruv24-art.github.io`

The webhook endpoint is:

`https://mmvyohlhhbycmccinexl.supabase.co/functions/v1/razorpay-webhook`

When ready, subscribe to Razorpay subscription lifecycle events including `subscription.activated`, `subscription.charged`, `subscription.halted`, `subscription.cancelled` and `subscription.completed`. The endpoint verifies Razorpay's HMAC signature. Enable its public webhook invocation only after the webhook secret is configured.

## Supabase website sign-in

After the GitHub Pages deployment is available, set Supabase **Authentication → URL Configuration** Site URL and redirect URL to:

`https://joshidhruv24-art.github.io/momentra-demo/`

## Product limitations

The current code supports studio email/password authentication, event records, private JPG/PNG/WebP uploads, short-lived studio gallery previews and the Razorpay subscription integration code. It does **not** implement automatic white-balance correction, RAW processing, face indexing/search, guest-only photo results, guest downloads, or a staff admin console. Do not upload guest selfies or describe those capabilities as active until their processing services, consent, deletion and access controls have been built and reviewed.

Before accepting real customers, configure and verify test-mode payments and webhook delivery, complete Razorpay review, and add operational backups, monitoring, customer data deletion/export, and suitable privacy/payment terms.
