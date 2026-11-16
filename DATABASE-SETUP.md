# Set up your new database

This ZIP includes the database structure and connection code. It does not include a provisioned cloud database, secret credentials, or a copy of your old records.

## Without a terminal

1. Create a Turso account at https://turso.tech and create an empty **libSQL** database named, for example, `gemstone-business`. Select libSQL, not the different Turso engine: this app uses SQLite triggers.
2. Get that database's URL and auth token from Turso. Keep the token private.
3. In your Vercel project settings, add `TURSO_DATABASE_URL` and `TURSO_AUTH_TOKEN` as environment variables for Production.
4. Upload all extracted project files to GitHub, including the `api`, `scripts`, `database`, `static` and `worker` folders. `api/index.js` must exist alongside the root `package.json` and `vercel.json`. Do not upload only the ZIP.
5. Redeploy. Opening the app and registering creates the tables automatically. No SQL copy/paste is necessary. `database/schema.sql` is included as a reference or for manual setup in a SQL console.
6. Test registration, login, a listing/photo, an order and a payment. Redeploy again and check that records remain.

If you intend to restore old records, import them into the empty database **before registering new accounts**. Use a separate database for tests/previews. Do not use a local `.sqlite` file as Vercel production storage.

## Optional terminal setup

Install Node.js 24, run `npm ci` in this folder, copy `.env.example` to `.env.local`, and privately fill in the Turso URL and token. Run `npm run db:setup`. This creates missing tables and preserves existing records.

## Restore existing records

The package includes a guarded importer for a **complete SQLite backup** with this app's schema. It is not a tool for exporting the old hosted database. The partial rows/images shown by a database viewer are not a complete backup and must not be used.

1. Obtain a full backup from the old database, including full photos, password hashes, orders, delivery contacts and payments. The ZIP contains no such backup. Keep a separate untouched copy.
2. Pause writes to the old app while making the final backup and performing the transfer so no new orders are missed.
3. Create a new, empty destination libSQL database. Set its credentials in `.env.local`; never commit that file or the backup to GitHub.
4. Run `npm run db:import -- /path/to/full-backup.sqlite`.
5. The importer checks the backup, refuses a populated destination, imports in one transaction, verifies row counts and restores the stock/order triggers. It preserves existing stock rather than subtracting historic orders twice. It rolls back on failure.
6. Existing password hashes are retained. Sessions are excluded, so users sign in again. Accounts created only through the former ChatGPT sign-in do not have a password; they need a separate account-recovery process before they can sign in on Vercel.
7. Compare old/new inventory, photos, order totals, received payments and balances before directing users to the Vercel site. Keep the old database until verification is complete.

The importer targets this small project's schema. Large databases may exceed remote transaction duration limits and require a provider-assisted migration. It does not merge two populated databases.

## Verification performed for this package

Local automated tests cover repeated setup, photo/password/payment preservation, historical stock preservation, refusal to overwrite records, transaction rollback, restored stock triggers and login/session handling. Live Turso setup, full old-data export and Vercel deployment still require your accounts and have not been performed by this ZIP.

Official references: https://docs.turso.tech/sdk/ts/reference and https://vercel.com/docs/environment-variables
