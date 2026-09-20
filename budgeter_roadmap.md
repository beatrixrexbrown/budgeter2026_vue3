# Budgeter 2026 - Project Roadmap

## Phase 1: Frontend Mock Pages & Local State (Vue 3)

- **Design Account Dashboard View**: Build a clean mobile-first view listing all user accounts and their current balances.
- **Build Bill Entry Form**: Create a simple form interface to log a paid bill (amount, category, date, account used).
- **Mock Persistence**: Wire these views up to local storage or local JSON structures so the entire user flow can be tested end-to-end without touching the database yet.

## Phase 2: Backend Database & Migration Verification

1. **Inspect Live Database**: Run artisan commands on the droplet to check existing migration tables (`accounts`, `bills`).
2. **Finalize Database Schema**: Write or verify Laravel migrations for `accounts` and `bills` tables to match the exact fields required by the frontend mocks.

## Phase 3: Core API Endpoint Development (Laravel 13)

1. **Accounts API**: Build controllers and routes for fetching all accounts and updating an account balance.
2. **Bills API**: Build controllers and routes for storing newly paid bills.
3. **API Documentation**: Update Scribe annotations to document the new endpoints cleanly.

## Phase 4: Frontend-to-Backend Integration

1. **Connect API Client**: Swap out local mock storage in the Vue 3 app for real Axios requests pointing to `http://dgloriaapi.co.uk:8081/api`.
2. **End-to-End Testing**: Test checking balances, updating balances, and logging bills from the live frontend to the live server.

## Phase 5: Production Polish & Hardening

1. **Turn Off Debug Mode**: Execute the production command to disable debug on the live environment (`APP_DEBUG=false`).
2. **Final Deployment Run**: Push updates to GitHub and run the automated production deployment script on the DigitalOcean droplet.
