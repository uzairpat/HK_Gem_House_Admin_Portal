# Hong Kong Gem House — Website & Admin Portal

A unified web platform for **Hong Kong Gem House** that brings the public gemstone website and the company owner's administration workspace into one React application, backed by a shared API and PostgreSQL database.

**Repository:** [uzairpat/HK_Gem_House_Admin_Portal](https://github.com/uzairpat/HK_Gem_House_Admin_Portal)  
**Default branch:** `main`

---

## Project overview

The platform has two main interfaces:

- **Public website** — presents the Hong Kong Gem House brand, heritage, gemstone collections, quality and trust information, global presence and contact options.
- **Owner Admin Portal** — a protected workspace for managing gemstone inventory, customer enquiries, customer records and operational information.

Both interfaces use the same backend API and the same database within each environment. The database is the source of truth for managed gemstone and enquiry records.

## Main features

### Public website

- Responsive, single-page public website with the sections Home, Our Heritage, Our Gemstones, Quality & Trust, Global Presence and Contact.
- Header navigation with section-aware scrolling.
- Gemstone category navigation for Ruby, Blue Sapphire, Emerald and Rare Gemstones.
- Gemstone collection and detail routes.
- Customer registration, sign-in, email verification and account pages.
- Public contact/enquiry form.
- WhatsApp click-to-chat entry point.

### Owner Admin Portal

- Owner-only sign-in, password recovery foundation and mandatory authenticator-app MFA flow.
- Admin dashboard with operational summary cards and recent activity.
- Gemstone inventory management, including draft, publication and availability states.
- Public visibility and public price display controls.
- Enquiry inbox with status, priority, staff assignment and private notes.
- Customer directory.
- Staff directory records (staff directory entries do not grant portal access).
- Activity/audit history foundation.

### Shared backend

- Express API for public catalogue, customer authentication, enquiries and owner-only administration.
- PostgreSQL persistence for users, sessions, gemstones, enquiries and related administration records.
- CSRF protection for state-changing requests and protected owner routes.
- Database migration scripts and a one-time initial owner bootstrap command.
- Public API responses are designed to exclude private business information such as acquisition cost, internal notes and private certificate data.

---

## Technology stack

| Layer | Technology |
|---|---|
| Frontend | React, TypeScript, Vite |
| Routing | React Router |
| Backend | Node.js, Express, TypeScript |
| Database | PostgreSQL |
| Authentication | Server-side sessions, Argon2 password hashing, owner MFA |
| Validation / security | Zod, Helmet, CORS, CSRF protection |
| Source control | Git and GitHub |
| Frontend hosting configuration | Netlify (`netlify.toml`) |

The frontend and backend are separate applications inside this repository:

- `hong-kong-gem-house-website/` — React/Vite public website and Admin Portal.
- `backend/` — Express API and PostgreSQL migrations.
- `docs/` — project specifications, implementation plan and setup guides.

---

## Run locally

### Requirements

- Git
- Node.js **22.12 or newer**
- PostgreSQL
- VS Code or another code editor

### 1. Clone the repository

~~~bash
git clone https://github.com/uzairpat/HK_Gem_House_Admin_Portal.git
cd HK_Gem_House_Admin_Portal
~~~

### 2. Configure and start the backend

Open a VS Code terminal in the repository root:

~~~powershell
cd backend
npm ci
Copy-Item .env.example .env
~~~

Edit `backend/.env` with the connection string for a **dedicated local Hong Kong Gem House development database**, a random session secret of at least 32 characters, and a private MFA encryption key. The local setup example uses:

- Database: `hk_gem_house_admin_dev`
- API port: `4000`

Confirm the database identity and permissions before running migrations. Then, only against the intended development database:

~~~powershell
npm run migrate
~~~

For the first local owner account only, set `ADMIN_OWNER_EMAIL` and a temporary `ADMIN_OWNER_PASSWORD` (at least 16 characters) in `backend/.env`, then run the one-time bootstrap:

~~~powershell
npm run admin:bootstrap
~~~

Do not run the bootstrap again if an administrator already exists. Remove the temporary `ADMIN_OWNER_PASSWORD` value after successful bootstrap, but keep the MFA encryption key stable after MFA enrollment.

Start the API:

~~~powershell
npm run dev
~~~

Keep this terminal running. The API is available at `http://localhost:4000`.

### 3. Configure and start the frontend

Open a **second VS Code terminal** from the repository root:

~~~powershell
cd hong-kong-gem-house-website
npm ci
Copy-Item .env.example .env.local
~~~

Make sure `hong-kong-gem-house-website/.env.local` contains:

~~~dotenv
VITE_API_BASE_URL=http://localhost:4000
VITE_WHATSAPP_NUMBER=
~~~

Start Vite:

~~~powershell
npm run dev
~~~

Keep this second terminal running.

### 4. Open the application

| Destination | Local URL |
|---|---|
| Public website | http://localhost:5173/ |
| Owner Admin Portal | http://localhost:5173/admin/login |
| Backend health check | http://localhost:4000/api/health |

**Both terminals must remain running.** The frontend calls the API at `http://localhost:4000`.

For the complete setup sequence, PostgreSQL role/database preparation, environment-variable details, first owner creation, MFA and troubleshooting, see **[Local Setup & Admin Access](docs/LOCAL_SETUP_AND_ADMIN_ACCESS.md)**.

---

## Useful development commands

Run commands from the relevant application directory.

### Frontend — `hong-kong-gem-house-website/`

~~~bash
npm run dev
npm run typecheck
npm run build
npm run preview
~~~

### Backend — `backend/`

~~~bash
npm run dev
npm run typecheck
npm run build
npm start
npm run migrate
npm run admin:bootstrap
~~~

---

## Environment variables and security

- Local environment files are not intended to be committed. Use `.env.example` files as templates.
- Never commit database passwords, session secrets, MFA encryption keys, SMTP credentials, authenticator recovery codes or private certificate links.
- Use separate databases and credentials for local development, staging and production.
- Do not point local development at the production database or another application's database.
- The initial owner account is created through the one-time bootstrap process; there is no public admin registration endpoint.
- Owner routes require an authorised owner session and MFA. State-changing API requests require CSRF protection.

---

## Current implementation status

The repository contains a development baseline for the public website, owner authentication/MFA foundation, dashboard, gemstone inventory CRUD and publication controls, enquiry CRM foundation, customer directory, staff records and audit foundations.

The following areas are **not yet complete for production use** and should not be represented as finished features:

- Managed gemstone image/video uploads and private certificate upload/delivery.
- Full database-driven Ruby collection and gemstone detail-page content; some existing Ruby content remains hardcoded and needs API integration.
- Full end-to-end verification of all category collections against owner-created inventory.
- Complete quotations, orders, invoices, payment allocation, receipts, unapplied credits and refunds.
- Full structured website CMS editing and revision workflow.
- Complete GA4 integration, consent workflow and analytics/reporting screens.
- Full notification/email delivery and production operational workflows.
- Production hosting, production database cutover, domain configuration and release approval.

The current local database is a development/test database, not the live business database. Production deployment requires a separately configured, secured and approved backend API and PostgreSQL database.

---

## Hosting notes

The repository includes a root `netlify.toml` for the Vite frontend build and an SPA redirect rule at `hong-kong-gem-house-website/public/_redirects`.

Netlify hosts the **frontend**. The Express API must be deployed separately, and PostgreSQL must be available to that backend. Set the production frontend variable `VITE_API_BASE_URL` to the public HTTPS URL of the deployed API. Do not use `localhost` as the production API URL.

The site is not production-ready merely because the frontend build can be deployed. Configure CORS, secure cookies/session settings, HTTPS, database credentials, MFA encryption key, email delivery and a production database before enabling real business use.

---

## Project documentation

- [Local Setup & Admin Access](docs/LOCAL_SETUP_AND_ADMIN_ACCESS.md)
- [Unified Website & Admin Portal Implementation Plan](docs/HK_GEM_HOUSE_UNIFIED_PLATFORM_IMPLEMENTATION_PLAN.md)
- [Stone Inventory & Media Specification](docs/HK_GEM_HOUSE_STONE_INVENTORY_AND_MEDIA_SPEC.md)

## License

No open-source license has been added. Unless the repository owner adds a license, the source remains subject to the repository owner's rights and is not automatically licensed for reuse or redistribution.
