# HK Gem House — Local Setup & Admin Access

This guide explains how to run the public website and owner Admin Portal locally from this repository.

## 1. Requirements

- Git
- Node.js 22.12 or newer (Node.js 24 LTS is suitable)
- PostgreSQL running locally
- A separate development database for this project

Do not use the GrowAIs database, another project database, or a production database.

## 2. Clone the repository

Open PowerShell:

```powershell
git clone https://github.com/uzairpat/HK_Gem_House_Admin_Portal.git
cd HK_Gem_House_Admin_Portal
```

## 3. Create a dedicated local PostgreSQL role and database

Open pgAdmin Query Tool while connected to your local PostgreSQL server as a database administrator.

Create a login role and dedicated database. Replace the example password with a strong private password:

```sql
CREATE ROLE hkg_gem_house_app
  WITH LOGIN PASSWORD 'REPLACE_WITH_A_STRONG_LOCAL_PASSWORD';

CREATE DATABASE hk_gem_house_admin_dev
  OWNER hkg_gem_house_app;
```

If the role or database already exists, do not run the corresponding CREATE statement again. Verify that the existing database is dedicated to this project and that the application role owns it or has the permissions needed to run migrations.

When using a password containing reserved URL characters, URL-encode it in the DATABASE_URL value.

## 4. Configure and start the backend

Open a PowerShell terminal in the repository root:

```powershell
cd backend
npm ci
Copy-Item .env.example .env
```

Edit `backend/.env` and set these values:

```dotenv
NODE_ENV=development
PORT=4000
DATABASE_URL=postgresql://hkg_gem_house_app:URL_ENCODED_PASSWORD@localhost:5432/hk_gem_house_admin_dev
DATABASE_SSL=false
CORS_ORIGIN=http://localhost:5173,http://localhost:5174
SESSION_SECRET=PASTE_A_RANDOM_SECRET_AT_LEAST_32_CHARACTERS
APP_BASE_URL=http://localhost:5173
ADMIN_OWNER_EMAIL=owner@your-company-domain.com
ADMIN_OWNER_PASSWORD=SET_A_PRIVATE_TEMPORARY_PASSWORD_AT_LEAST_16_CHARACTERS
MFA_ENCRYPTION_KEY=PASTE_A_PRIVATE_BASE64_32_BYTE_KEY
```

Generate a session secret (run in PowerShell and paste the output into SESSION_SECRET):

```powershell
[Convert]::ToBase64String([Security.Cryptography.RandomNumberGenerator]::GetBytes(48))
```

Generate the MFA encryption key (run in PowerShell and paste the output into MFA_ENCRYPTION_KEY):

```powershell
[Convert]::ToBase64String([Security.Cryptography.RandomNumberGenerator]::GetBytes(32))
```

Keep both values private. The MFA encryption key must remain stable after MFA enrollment; store it securely and do not regenerate it for an already-enrolled account.

### Confirm the database before migrating

Check that DATABASE_URL ends with the dedicated database name `hk_gem_house_admin_dev`. Make sure the database is not a production or GrowAIs database.

Then apply the project's migrations:

```powershell
npm run migrate
```

The migration command records applied SQL files in `schema_migrations` and skips migrations already applied.

### Create the initial owner account (one time only)

Set `ADMIN_OWNER_EMAIL` and a private temporary `ADMIN_OWNER_PASSWORD` (minimum 16 characters) in `backend/.env`, then run:

```powershell
npm run admin:bootstrap
```

The initial owner bootstrap is one-time only. It will not create or replace an owner if an admin account already exists. Do not rerun it to reset a password.

After it succeeds, remove the `ADMIN_OWNER_PASSWORD` line or clear its value in `backend/.env`. Keep the email, SESSION_SECRET, DATABASE_URL and MFA_ENCRYPTION_KEY configured as needed.

Start the API:

```powershell
npm run dev
```

Keep this terminal open.

Check the API in another terminal or browser:

```powershell
Invoke-RestMethod http://localhost:4000/api/health
```

Expected health response includes `status: ok` and database connection `connected`.

## 5. Run both servers in VS Code

Open the cloned repository folder in VS Code:

`HK_Gem_House_Admin_Test`

You need **two separate VS Code integrated terminals**. Keep both terminals running while using the website.

### Terminal 1 — Backend API

In VS Code, select **Terminal → New Terminal**. In this terminal, move into the backend folder:

```powershell
cd backend
npm ci
```

Run `npm ci` only during initial setup or when dependencies change. After configuring `backend/.env` and applying approved local database migrations, start the API:

```powershell
npm run dev
```

Keep this terminal open. The API runs at `http://localhost:4000`.

### Terminal 2 — Frontend website and Admin Portal

In VS Code, select **Terminal → New Terminal** again. This second terminal starts at the repository root. Move into the frontend folder:

```powershell
cd hong-kong-gem-house-website
npm ci
Copy-Item .env.example .env.local
```

Run `npm ci` during initial setup or when dependencies change. Ensure `hong-kong-gem-house-website/.env.local` contains:

```dotenv
VITE_API_BASE_URL=http://localhost:4000
VITE_WHATSAPP_NUMBER=
```

The WhatsApp number is optional for local development. Start the frontend:

```powershell
npm run dev
```

Keep this terminal open. Vite normally runs at `http://localhost:5173`.

### Open the website

- Public website: http://localhost:5173/
- Owner Admin Portal: http://localhost:5173/admin/login
- Backend health check: http://localhost:4000/api/health

Both terminal windows must remain running. The frontend uses the backend at `http://localhost:4000`.

## 6. Admin sign-in details

- Admin email: use the company-approved owner email configured as `ADMIN_OWNER_EMAIL` before the one-time bootstrap.
- Admin password: supplied separately by the account owner. Do not add the real password to this guide or to GitHub.
- Login URL: http://localhost:5173/admin/login
- MFA: required. When the login asks for a verification code, contact the account owner to receive the current time-sensitive authenticator code through a separate secure channel. Do not store verification codes in this repository.

If this is the first sign-in, follow the authenticator-app enrollment prompt before completing MFA.

## 7. Admin account details and security

- Initial password: created privately by the person setting up the local database; there is no default password in the repository.
- MFA: required. Complete authenticator-app enrollment at first sign-in.
- The bootstrap creates only the initial owner. The staff directory does not create portal login accounts.

Do not put passwords, DATABASE_URL values, SESSION_SECRET, MFA_ENCRYPTION_KEY, SMTP credentials, authenticator recovery codes, or private certificate links in GitHub, README files, screenshots, or group messages. Share any temporary password directly and securely with the intended recipient, then rotate it after the demonstration if appropriate.

Each person's local database and initial owner setup are separate. A user created in one laptop's local PostgreSQL database does not automatically exist on another laptop.

## 8. Current implementation limits

The following are not yet complete for production use: managed image/video uploads, private certificate uploads, full quotations/orders/invoices/payment allocation workflows, full structured CMS editing, GA4 reporting, and production database cutover. The current local database is for development/testing only.

## 9. Troubleshooting

- If the frontend cannot reach the API, confirm the backend is running on port 4000 and `VITE_API_BASE_URL=http://localhost:4000`.
- If browser requests are blocked by CORS, confirm `CORS_ORIGIN` includes the exact Vite origin, normally `http://localhost:5173`.
- If the API reports a database error, verify PostgreSQL is running, the database name in `DATABASE_URL` is exactly `hk_gem_house_admin_dev`, the password is URL-encoded when required, and the application role has database/schema/table permissions.
- If bootstrap says an administrator already exists, do not rerun bootstrap. Check the owner account in the local database and use the supported password-recovery process.
- Do not run migrations or bootstrap against production without an approved backup and release plan.
