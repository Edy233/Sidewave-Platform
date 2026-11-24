# Sidewave

Sidewave is a Node.js and MongoDB platform with a shared account for three experiences.

| Area | Features |
| --- | --- |
| Commons | Publish and remove your own posts, search for members, and manage access to your private profile with approved connections. |
| Stillroom | Upload and caption photos, explore the community gallery, and save favorites. Profile visibility applies to uploaded images and gallery listings. |
| Lowline | Stream an original three-track catalog, preview premium tracks, and unlock individual recordings with credits. |

The membership page offers Free, Plus ($9/month), and Studio ($19/month), plus three credit packs. Plus includes full-catalog streaming and an optional private listening history. Studio also offers an optional high-resolution photo upload setting.

## Run locally

From the extracted `sidewave` folder:

```sh
docker compose up --build
```

Open [http://localhost:3000](http://localhost:3000) and create an account. The default Compose stack binds Sidewave to the local machine and keeps MongoDB on a container-only network. Its named volume persists your accounts, posts, photos, and settings. `docker compose down` stops the stack and preserves those records.

For a direct Node.js install, use Node.js 22 or newer and a running MongoDB instance. Copy `.env.example` to `.env`, set `MONGODB_URI`, then run:

```sh
npm ci
npm start
```

The platform starts with its own MongoDB database, `sidewave`. Accounts and purchases from the earlier classroom project are separate.

## Deploy

Run the included Docker image on a Node-compatible host. Configure `MONGODB_URI` for a dedicated, authenticated MongoDB deployment and use TLS for the connection. Set `APP_ORIGIN` to the exact public HTTPS origin. Terminate HTTPS at the host's reverse proxy and route it to the app port. Keep `MONGODB_URI` and Stripe values in the host's secret settings. Database credentials belong to an account with access to the Sidewave database.

## Enable Stripe Checkout

1. Use a Stripe test account and an Atlas cluster or MongoDB replica set. Credit fulfillment uses a database transaction to record the Stripe session and add credits together.
2. In Stripe, create monthly USD prices for Plus ($9) and Studio ($19), and one-time USD prices for 25 credits ($3), 100 credits ($9), and 250 credits ($19). Set their environment variables as listed in `.env.example`.
3. Set the API key and signing secret through the deployment environment. Add `/api/stripe/webhook` to your HTTPS origin in Stripe and subscribe it to the following events:
   - `checkout.session.completed`
   - `checkout.session.async_payment_succeeded`
   - `checkout.session.expired`
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
4. Configure Stripe Customer Portal for subscription cancellation and switching between the two monthly prices.
5. Verify the flow in Stripe test mode by paying for a credit pack, returning to Membership, refreshing the balance, and unlocking a track. Check subscription activation and cancellation separately before enabling live billing.

Sidewave verifies the webhook signature, re-reads Checkout from Stripe, and awards credits once per paid session. Subscriber access follows Stripe's active subscription state and paid period. Credit refunds and disputes require manual reconciliation. See [Stripe Checkout fulfillment](https://docs.stripe.com/checkout/fulfillment), [subscription webhooks](https://docs.stripe.com/billing/subscriptions/webhooks), and [Customer Portal integration](https://docs.stripe.com/customer-management/integrate-customer-portal).

The software includes username and password sign-in. Email-based account recovery and public content moderation workflows are outside this initial release.

## Classroom fork exchange

[CLASSROOM.md](CLASSROOM.md) contains the instructor setup, team assignments, handoff form, and scoring rubric. Each team works from the same runnable Sidewave baseline, then reviews a peer's changes at a specific commit.

## Verify

```sh
npm run check
MONGODB_TEST_URI='mongodb://127.0.0.1:27017/?replicaSet=rs0' npm test
```

Use a test replica set. The suite creates disposable databases and exercises actual sharing, photo uploads, account boundaries, playback, and purchase fulfillment. Billing checks use signed test events and a local Stripe service fixture.

## Media

The three recordings were synthesized for this project. The Field Notes and hero photos are from Unsplash. [License and source photos](https://unsplash.com/license).

- [Lake cabin](https://images.unsplash.com/photo-1470770841072-f978cf4d019e)
- [Sunrise](https://images.unsplash.com/photo-1470252649378-9c29740c9fa8)
- [Mountain ridge](https://images.unsplash.com/photo-1464822759023-fed622ff2c3b)
- [Shoreline](https://images.unsplash.com/photo-1501785888041-af3ef285b470)
- [Night sky](https://images.unsplash.com/photo-1519681393784-d120267933ba)
