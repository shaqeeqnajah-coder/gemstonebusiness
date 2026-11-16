# Gemstone Business — Vercel edition

Start with [DATABASE-SETUP.md](DATABASE-SETUP.md) for the new database and guarded import instructions.

This package adapts the existing gemstone app to Vercel Functions and a remote Turso **libSQL** database. The current hosted site is not changed by this package. Deployment and a live remote database test still require your accounts.

## Deploy without a terminal

1. Extract this ZIP on your computer.
2. Create a private GitHub repository. Upload the extracted contents so `package.json`, `vercel.json`, `api`, `scripts`, `static`, `worker` and `tests` are at its root. Upload the files, not the ZIP.
3. In Turso, create a **libSQL** database (this app uses SQLite triggers). Copy its database URL and create a database auth token. Use a separate empty database for preview/testing.
4. In Vercel, choose **Add New → Project**, connect GitHub and import this repository.
5. Choose **Other** as the framework. The included configuration sets build command `npm run build`, output directory `public`, and Node.js `24.x`.
6. Add these environment variables to the production project:
   - `TURSO_DATABASE_URL`: the remote libSQL database URL.
   - `TURSO_AUTH_TOKEN`: the database auth token.
   - `OPENAI_API_KEY`: optional, only needed for the AI chatbot. Enter it privately in Vercel, never in chat or GitHub. AI API usage is billed separately.
7. Click **Deploy**. Open the resulting URL and register an account. Database tables are created automatically on the first API request.
8. Verify the checklist below before using it for a demonstration.

Never put tokens in `static`, browser JavaScript, screenshots, or GitHub. Set preview environment variables to a separate test database so preview deployments cannot change production data. If credentials change, redeploy.

## Data and feature boundaries

- A fresh database starts empty. Existing accounts, orders and photos on the previous hosted site are **not automatically transferred**. Keep that site available until a deliberate export/import migration is completed.
- Listings and certificate photos are currently stored as compressed image data in the database. This preserves the existing implementation and uses database storage quota; a large marketplace should move images to object storage and paginate results.
- Certificate scanning reads QR/barcode information and links to lab verification resources. It does not provide universal authenticated access to every laboratory.
- The chatbot needs a separately configured, funded API key. Do not describe a photo-based reply as a laboratory identification.
- Vercel Hobby is for personal/non-commercial use. Check current Vercel and Turso quotas before deploying; commercial marketplace operation needs an appropriate plan.

## Acceptance checklist after deployment

- Register seller and buyer accounts; verify login/logout and persistence after refresh.
- On a real mobile phone, upload a gemstone photo from the gallery and camera; save it and reopen it.
- Place an order as the buyer; verify seller sales, remaining stock and account balances.
- Record a partial payment as the seller; verify both accounts' balances.
- Attach and reopen a certificate photo; try QR/barcode scanning on the phone.
- Delete your own listing; verify it disappears while past orders remain.
- Confirm another user cannot modify or delete your listings.
- Try AI chat after configuring API billing; verify useful error handling without a key.
- Redeploy and confirm saved records remain.

## Developer checks

Requires Node 24. Run `npm ci`, `npm test`, `npm run test:vercel`, and `npm run build`.

The original business-flow tests use Node SQLite. The Vercel suite runs the same business flows against the real libSQL client with a local test database and separately checks persisted sessions, header stripping, origin protection and missing configuration. These tests do not replace a live Vercel/Turso integration test.

`npm start` runs the local SQLite edition on localhost. `npm run dev` is a QA fixture server, not a production server. The Vercel entry point is `api/index.js`; its database adapter is `scripts/libsql-adapter.mjs`. Never expose the QA server publicly.

Official references:
- https://vercel.com/docs/functions/runtimes/node-js
- https://vercel.com/docs/plans/hobby
- https://docs.turso.tech/sdk/ts/reference
