# MOMENTRA Professional Starter

Responsive studio dashboard and deployable backend starter for the Momentra wedding-photo delivery service.

## What works in preview mode

- Responsive studio dashboard, event creation, branding/settings screens, upload and guest-flow previews.
- Current studio subscription prices: ₹999 monthly, ₹2,699 quarterly, ₹9,999 yearly. Create Razorpay plans using these exact amounts and billing periods.
- Events are saved to this browser only. Chosen files stay on this device and are not transmitted.
- Guest face matching and payment checkout are disabled until the secure providers are configured.

## What is implemented for connected mode

- Supabase email/password authentication.
- Studio profile and event tables protected with Row Level Security (RLS).
- Private Supabase Storage bucket for original JPEG, PNG, and WebP uploads, with per-studio path policies.
- Authenticated Razorpay Subscriptions checkout using one pre-created Razorpay plan for each period. Checkout is configured to show cards, netbanking and UPI only, and hide Pay Later, eMandate, EMI, Cardless EMI and wallets.
- Server-side HMAC verification of the Razorpay checkout callback and signed webhooks for subscription status.
- Short-lived signed URLs for studio-side gallery previews.

The deployable site is a static single-page app and can be hosted on GitHub Pages. It never contains a Supabase service-role key or Razorpay key secret. Provider credentials must be added as Supabase Function secrets.

## Setup

1. Create a Supabase project. In **SQL Editor**, run `supabase/schema.sql`. If you already ran an older version, run the upgrade statements at the end of the file.
2. In Supabase **Authentication → URL Configuration**, set the site URL to the published website and add its exact URL to the redirect allow list.
3. Copy the project's **Project URL** and **Publishable key** into `config.js`. The publishable key is designed for browser use; keep RLS enabled. Never use a secret/service-role key in this file.
4. In **Storage**, confirm that `momentra-originals` is private. The SQL creates the bucket and restrictive upload/read/delete policies.
5. The three plans have been created in the signed-in Razorpay account. Their IDs are `RAZORPAY_PLAN_MONTHLY=plan_TkLDqGzLUm7N9x` (₹999 every month), `RAZORPAY_PLAN_QUARTERLY=plan_TkLEUumVzaVKZh` (₹2,699 every 3 months), and `RAZORPAY_PLAN_YEARLY=plan_TkLEwHXFGiayb9` (₹9,999 every year). These periods and amounts are fixed on the Razorpay plans; create replacement plans if pricing changes later.

**Account activation blocker:** Razorpay Dashboard currently flags the registered website URL as needing clarification and says it is not live or accessible. Razorpay requires a publicly accessible website showing the offered service and its pricing before it can complete review. After publishing Momentra to a public URL, the account owner must update the website details and reply to the existing Razorpay support clarification; do not represent payments as live until Razorpay clears the review.
6. Add these secrets to Supabase Edge Functions (never to `config.js`):
   - `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` (use test credentials first)
   - `RAZORPAY_WEBHOOK_SECRET` (set when creating the webhook in Razorpay)
   - `RAZORPAY_PLAN_MONTHLY`, `RAZORPAY_PLAN_QUARTERLY`, `RAZORPAY_PLAN_YEARLY`
   - `ALLOWED_APP_ORIGINS` (scheme + host only, e.g. `https://username.github.io`)
7. Deploy `create-subscription`, `verify-subscription`, and `razorpay-webhook` Edge Functions. Configure Razorpay webhooks for subscription lifecycle events such as `subscription.activated`, `subscription.charged`, `subscription.halted`, `subscription.cancelled`, and `subscription.completed` to the Supabase `razorpay-webhook` endpoint, using the same webhook secret.
8. Publish the root `index.html` and `config.js` on GitHub Pages. Keep the `supabase/` source files with the site repository.
9. Test the complete flow with Razorpay test keys and test plans before switching to live keys. A payment is not considered production-ready until the Razorpay account is activated and webhook delivery is confirmed.

Razorpay subscriptions require a plan created before a subscription, and the period frequency is defined by the plan. Checkout displays only cards, netbanking and UPI; Pay Later, eMandate, EMI, Cardless EMI and wallets are hidden in the checkout configuration. Checkout details are verified server-side and subscription status is synchronized through signed webhooks. See [Razorpay Create Plan documentation](https://razorpay.com/docs/api/payments/subscriptions/create-plan/?preferred-country=IN) and [Razorpay integration and signature verification guide](https://razorpay.com/docs/server-integration/python/test-app/).

## Important remaining production integrations

This package does **not** yet implement biometric face indexing/search, automatic natural white-balance processing, RAW decoding, guest photo matching, downloadable guest archives, email delivery, or a staff admin console. Do not present those as active services or upload real guest selfies. They need a separately deployed image-processing worker and a chosen face-search provider, plus consent, retention/deletion, abuse controls, and provider/data-region review. The UI makes this limitation explicit.

Before inviting photographers, complete those integrations, add data deletion/export requests, configure backup/monitoring, and have local privacy/payment terms reviewed. GitHub Pages hosts the demo UI only; Supabase provides the backend once configured. Free service quotas and payment processing fees are controlled by those providers and may change.
