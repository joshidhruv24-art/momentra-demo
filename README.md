# MOMENTRA — Wedding Photo Delivery

A mobile-friendly studio app with public monthly/quarterly/yearly pricing, Supabase studio login, private photo storage, natural color-balanced derivatives, and Razorpay subscription checkout.

## Current browser experience

- Signed-out visitors land on the pricing page: ₹999 monthly, ₹2,699 quarterly, ₹9,999 yearly.
- Selecting a price asks the studio to log in, creates the chosen Razorpay subscription on the server, then opens Razorpay Checkout. The browser never receives the Razorpay secret.
- Payment verification and subscription state updates run through Supabase Edge Functions.
- Uploads keep the original image and create a separate 3000px natural color-balanced JPEG copy. White balance and exposure adjustments are deliberately gentle; they run in the photographer's browser.
- Guest face matching is implemented with in-browser face descriptors and event-scoped server search. Guest selfie pixels stay in the browser. The guest must consent before the descriptor is sent to Momentra.
- Face descriptor indexing is off by default per event. Studios must enable it and confirm they have informed guests. Owners can delete an event's face index without deleting its photos.

## External setup

The existing Supabase project already has the studio tables, private `momentra-originals` bucket, Razorpay plans, and payment functions configured. To enable the new face-match controls in a project that already exists:

1. Run `supabase/migrations/20261006_face_search.sql` once in Supabase SQL Editor.
2. Deploy `index-photo-faces`, `guest-face-search`, and `delete-event-face-index` from `supabase/functions/`.
3. Keep `guest-face-search` public at the Edge Function gateway (`verify_jwt = false`); it accepts only an event share token and an explicit one-time consent, and checks event expiry and indexing state. The other two functions require a studio login and verify ownership themselves.
4. Ensure the project has the existing `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` Edge Function environment values. Never place the service role key in `config.js`.
5. Publish this `index.html` and `config.js` together at the site's root so the pricing page and new processing flow are active.

The face recognition model is downloaded by the browser from the face-api.js upstream weights URL. Wedding photos and selfie images are processed in the browser; only event photo descriptors and a guest's one-time query descriptor are sent to the Momentra Supabase project. Face descriptors are biometric data: retain them only for an event with appropriate notice/permission, and delete them when no longer needed. Deleting an event cascades its descriptors; the dashboard also includes an event-level face-index deletion action.

## Payment configuration

- Monthly: ₹999 — `plan_TkLDqGzLUm7N9x`
- Quarterly: ₹2,699 per 3 months — `plan_TkLEUumVzaVKZh`
- Yearly: ₹9,999 per year — `plan_TkLEwHXFGiayb9`

Razorpay Checkout shows card, UPI and netbanking only; Pay Later, eMandate, EMI, Cardless EMI and wallets are hidden. Keep the account in test mode until Razorpay approves the public service/pricing site.

## Still needed before real customer use

- Run the SQL migration and deploy the three face-search functions before turning on event indexing.
- Wait for the current GitHub Pages deployment to succeed; until then, the public URL can still show the older preview.
- Complete Razorpay's website review and verify a test subscription lifecycle before taking real payments.
- Add signed guest downloads/favorites/sharing, RAW processing, expiry cleanup jobs, monitoring, and published privacy/retention terms before launch.
