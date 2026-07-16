# Deploy tsquaresolutions on CloudPanel (Node.js + MariaDB)

Everything the Node.js runtime needs is bundled inside `.output/`. No
`node_modules` install is required on the server — Nitro packaged all
dependencies (including `mysql2`) into the server bundle.

## 1. Create the MariaDB database in CloudPanel

CloudPanel → **Databases → Add Database**

- Database Name: `tsquare-solutions`
- User Name: `tsquare`
- Password: `Tsquare@123`

The app auto-creates its table on the first request, so no SQL import is
required. (If you prefer, you can import `db/schema.sql` via phpMyAdmin.)

## 2. Create a Node.js site

CloudPanel → **Sites → + Add Site → Create a Node.js Site**

- Domain: `tsquaresolutions.metaedgecreatives.com`
- Node.js version: **20** or newer
- App Port: **3000**
- Site User: (whatever CloudPanel suggests, e.g. `tsquaresolutions`)

CloudPanel will create `/home/<site-user>/htdocs/tsquaresolutions.metaedgecreatives.com/`.

## 3. Upload the zip

1. Upload `tsquaresolutions-nodejs.zip` into
   `/home/<site-user>/htdocs/tsquaresolutions.metaedgecreatives.com/`.
2. Unzip it there. You should end up with:
   ```
   htdocs/tsquaresolutions.metaedgecreatives.com/
     ├── .output/
     ├── .env
     ├── package.json
     ├── db/schema.sql
     └── DEPLOY.md
   ```

## 4. Configure environment variables

Edit `.env` in the site root and confirm the values (they are already
pre-filled with the DB you provided):

```
PORT=3000
NODE_ENV=production
DB_HOST=localhost
DB_PORT=3306
DB_NAME=tsquare-solutions
DB_USER=tsquare
DB_PASSWORD=Tsquare@123
```

## 5. Start command in CloudPanel

In the Node.js site settings, set the **App Startup File** to:

```
.output/server/index.mjs
```

or set the **Custom Start Command** to:

```
node .output/server/index.mjs
```

CloudPanel wraps this with PM2 automatically. Click **Restart** and the
site should be live at:

<https://tsquaresolutions.metaedgecreatives.com>

## 6. Enable HTTPS

CloudPanel → your site → **SSL/TLS → Actions → New Let's Encrypt Certificate**.

## Troubleshooting

- **502 Bad Gateway** — the Node app didn't start. Check
  CloudPanel → your site → **Logs**. Usually a wrong start file path or
  the port already in use.
- **DB connection error** — verify the values in `.env` match the
  database created in step 1, then restart the site.
- **Data doesn't persist** — check the site logs for MariaDB errors and
  confirm the `app_state` table exists in the `tsquare-solutions`
  database (it is created on first request).

## What is stored where

All application data (leads, customers, bookings, quotes, invoices,
inspections, services, staff, reviews, settings, service areas) is stored
as a single JSON document in the `app_state` table in MariaDB. Login is
still a mock login (accepts any credentials) as requested.
