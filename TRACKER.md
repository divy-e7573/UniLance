# TRACKER.md — Student–Freelancer Marketplace

> **For the AI agent:** This file is the single source of truth for project progress.
> Read `RULES.md` first, then this file, before every task.
> After finishing work, update this file as described in "How to update" below.

## How to update

- Mark an item `[x]` only when it is **fully implemented, runs without errors, and is tested** (or manually verified if no test exists yet).
- Use `[~]` for partially done work and add a short note after the item (e.g. `— backend done, UI pending`).
- Use `[!]` for blocked items and write the reason after it.
- Never delete items. If scope changes, strike through with `~~text~~` and add a note.
- Update the **Phase Status** table and **Current Focus** after every work session.
- Append one line to the **Change Log** at the bottom for every session (date, phase, what changed).
- Record any new decision in `RULES.md → Decisions Log`, not here.

Legend: `[ ]` not started · `[~]` in progress · `[x]` done · `[!]` blocked

## Phase Status

| Phase | Name | Status |
| --- | --- | --- |
| 0 | Project Setup & Foundations | Not started |
| 1 | Database Schema (Prisma) | Not started |
| 2 | Authentication & Accounts | Not started |
| 3 | Trial, Subscription State & Entitlements | Not started |
| 4 | Categories, Skills, Services & Work Experience | Not started |
| 5 | Student Profile, Requirements & Bookmarks | Not started |
| 6 | Marketplace Search & Discovery | Not started |
| 7 | Proposals & Orders | Not started |
| 8 | Chat (REST + Socket.IO) | Not started |
| 9 | File Management | Not started |
| 10 | Reviews & Ranking | Not started |
| 11 | Notifications | Not started |
| 12 | Subscription Payments | Not started |
| 13 | Dashboards & Frontend Polish | Not started |
| 14 | Admin Panel | Not started |
| 15 | Hardening, Testing & Deployment | Not started |

## Current Focus

- **Active phase:** Phase 0
- **Next task:** Initialize repository and folder structure
- **Blockers:** None

---

## Phase 0 — Project Setup & Foundations

- [ ] Create repo with monorepo or two-folder structure (`/frontend`, `/backend`) and `.gitignore`
- [ ] Initialize Next.js + React + Tailwind CSS in `/frontend`
- [ ] Initialize Node.js + Express (TypeScript) in `/backend` with layered structure (routes / controllers / services / data access / validation)
- [ ] Configure ESLint, Prettier, and TypeScript strict mode in both apps
- [ ] Set up PostgreSQL (local via Docker or managed) and Prisma connection
- [ ] Create `.env.example` for both apps (no real secrets committed)
- [ ] Add centralized config loader with Zod env validation
- [ ] Add global error handler, request logger, and standard API response/error format
- [ ] Add `/api/v1/health` endpoint
- [ ] Add README with setup and run instructions
- [ ] Configure test runner (Vitest or Jest) with one passing sample test

## Phase 1 — Database Schema (Prisma)

- [ ] Model `User` (common identity, globally unique normalized `username`, account type STUDENT/FREELANCER/ADMIN)
- [ ] Models `StudentProfile` and `FreelancerProfile`
- [ ] Model `OtpVerification` (short-lived, hashed code, attempts, expiry)
- [ ] Model `Session` / refresh token (if used by final auth design)
- [ ] Models `Category` (with optional parent) and `Skill`, plus join tables
- [ ] Model `Service` (category, price info, status, active/hidden flag)
- [ ] Model `WorkExperience`
- [ ] Model `StudentRequirement` (fixed price OR budget min/max, category, status)
- [ ] Model `Proposal` (source: REQUIREMENT or SERVICE, optional proposed price, status)
- [ ] Models `Order` and `OrderStatusHistory` (with snapshot fields for title, terms, agreed price)
- [ ] Model `OrderFile`
- [ ] Models `Conversation`, `ConversationMember`, `Message` (with read state)
- [ ] Model `Review` (tied to order and reviewer/reviewee)
- [ ] Model `Subscription` (status, trial and subscription dates, provider references)
- [ ] Model `Payment` (subscription payment records only)
- [ ] Model `Notification`
- [ ] Model `Bookmark`
- [ ] Add all UNIQUE constraints (username, one active proposal per freelancer per requirement, one accepted order per requirement, one review per reviewer per order)
- [ ] Add foreign keys and indexes (username, category, status, owner IDs, createdAt)
- [ ] Create initial migration and a seed script (categories, skills, demo data)
- [ ] Document the schema with an ER diagram in `/docs/database.md`

## Phase 2 — Authentication & Accounts

- [ ] Username normalization utility (lowercase, no spaces, restricted charset) with unit tests
- [ ] `GET /api/v1/auth/check-username` (rate-limited, indexed lookup)
- [ ] OTP request endpoint (phone for students and freelancers, email for students) with rate limiting
- [ ] OTP verification endpoint (short-lived, attempt-limited, hashed storage)
- [ ] Student registration (OTP → name/username/password → account + profile)
- [ ] Freelancer registration (phone OTP → name/username/password → account + profile + trial started, in one transaction)
- [ ] Password hashing with Argon2id
- [ ] Login with username + password, JWT in HTTP-only secure cookie
- [ ] Logout and session/refresh handling
- [ ] Auth middleware and role-based authorization middleware
- [ ] Brute-force and abuse protection on login and OTP endpoints
- [ ] Frontend: register (student), register (freelancer), login pages with debounced username check
- [ ] Frontend: route guards (UX only; backend remains authoritative)
- [ ] Tests: registration race on same username, wrong OTP, expired OTP, role access

## Phase 3 — Trial, Subscription State & Entitlements

- [ ] `Subscription` row created with `trialStartedAt` / `trialEndsAt` (server time, +7 days) at freelancer creation
- [ ] Entitlement service computing state: TRIAL / ACTIVE / EXPIRED from server timestamps
- [ ] Entitlement middleware guarding: create service, post work experience, "I'm Interested"
- [ ] Service limit enforcement (3 trial, 10 active) inside a transaction
- [ ] On expiry: retain existing services, hide/deactivate them from marketplace, restore on re-subscription
- [ ] `GET /api/v1/subscription/me` returning state, days left, limits
- [ ] Scheduled job or lazy check to flip expired states reliably
- [ ] Frontend: trial banner, "Subscribe to continue" prompt on restricted actions
- [ ] Tests: trial start, exact expiry boundary, student usage never affects trial, limit 3 vs 10, restoration

## Phase 4 — Categories, Skills, Services & Work Experience

- [ ] Public endpoints to list categories and skills
- [ ] Freelancer profile update endpoint (bio, skills, categories, avatar)
- [ ] Service CRUD (`POST/PATCH/DELETE /api/v1/services`) with Zod validation and ownership checks
- [ ] Service activate/deactivate behavior tied to entitlement
- [ ] Work Experience CRUD (separate from services)
- [ ] Public freelancer profile endpoint (profile + services + work experience + rating summary)
- [ ] Frontend: freelancer profile editor, "Post Service" and "Post Work Experience" forms
- [ ] Frontend: public freelancer profile page
- [ ] Tests: ownership, limits, validation, expired-state behavior

## Phase 5 — Student Profile, Requirements & Bookmarks

- [ ] Student profile update endpoint
- [ ] Requirement CRUD with title, description, category, skills, and fixed price or budget range
- [ ] Requirement status (OPEN, ASSIGNED, CLOSED) with owner-only management
- [ ] Bookmark add/remove/list (freelancers; services optional per Decisions Log)
- [ ] Frontend: "Post Requirement" form and "My Posts" page
- [ ] Frontend: bookmark button and bookmarks page
- [ ] Tests: price validation (fixed vs budget), ownership, duplicate bookmarks

## Phase 6 — Marketplace Search & Discovery

- [ ] `GET /api/v1/marketplace/services` with search, category, skills, price filters, pagination
- [ ] `GET /api/v1/marketplace/requirements` with search, category, budget filters, pagination
- [ ] Freelancer listing/discovery for students
- [ ] Search across title, description, skills, categories
- [ ] Default ranking by rating signals (formula isolated in one replaceable module)
- [ ] Exclude hidden/inactive services of expired freelancers
- [ ] Frontend: top search bar, category selection, filters, pagination/infinite scroll
- [ ] Add DB indexes for the main filter and sort fields
- [ ] Tests: filters, pagination, hidden-content exclusion

## Phase 7 — Proposals & Orders

- [ ] `POST /requirements/:id/proposals` ("I'm Interested" + optional price), entitlement-guarded
- [ ] Prevent duplicate active proposals by the same freelancer on a requirement
- [ ] Direct-hire flow from a service (creates a Proposal with source SERVICE; see Decisions Log)
- [ ] Student view of proposals per requirement
- [ ] `POST /proposals/:id/accept` as one atomic transaction (verify open → mark accepted → reject/close others → create Order → close requirement)
- [ ] Reject proposal endpoint
- [ ] Order creation with snapshot of title, terms, and agreed price
- [ ] Order status transitions with backend-validated transition matrix
- [ ] `PATCH /orders/:id/status` (freelancer-driven; student actions for revision request and completion confirmation per matrix)
- [ ] Append-only `OrderStatusHistory` on every change
- [ ] Order list and detail endpoints for both roles (multiple active orders supported)
- [ ] Frontend: proposals list, accept/reject UI, orders pages, status tracker UI
- [ ] Tests: concurrent acceptance race, invalid transitions, snapshot immutability after service edit

## Phase 8 — Chat (REST + Socket.IO)

- [ ] REST: create/get conversation (from profile, service, requirement, or order context)
- [ ] REST: paginated message history
- [ ] Socket.IO server with authenticated connections (cookie-based)
- [ ] Real-time send/receive with persistence in PostgreSQL
- [ ] Read/unread state and unread counts
- [ ] Typing and online indicators (optional, low priority)
- [ ] Participant-only access enforced on REST and socket events
- [ ] Frontend: inbox, conversation view, real-time updates, reconnect handling
- [ ] Tests: non-participant access denied, persistence after reload

## Phase 9 — File Management

- [ ] Choose storage provider (Cloudinary or S3-compatible) and record in Decisions Log
- [ ] Upload endpoint with auth, size and type restrictions
- [ ] Store metadata only in DB (filename, type, size, uploader, order/conversation link, storage key)
- [ ] Access-controlled download URLs (participants only)
- [ ] Attach files to orders (requirements, references, deliverables) and to chat
- [ ] Frontend: upload UI with progress, file list on order and chat
- [ ] Tests: type/size rejection, unauthorized access denied

## Phase 10 — Reviews & Ranking

- [ ] `POST /reviews` allowed only for participants of COMPLETED orders
- [ ] Enforce one review per reviewer per order
- [ ] Both directions supported (student ↔ freelancer)
- [ ] Aggregate rating fields (average, count) updated transactionally
- [ ] Ranking module consumes rating, review count, and completed orders
- [ ] Frontend: review form after completion and review lists on profiles
- [ ] Tests: ineligible reviewer, duplicate review, aggregate correctness

## Phase 11 — Notifications

- [ ] Notification service with a single `notify(userId, type, payload)` entry point
- [ ] Triggers: new proposal, proposal accepted/rejected, new message, order status change, files submitted, revision requested, order completed, trial near expiry, subscription activated/expired, new review
- [ ] `GET /notifications` (paginated) and mark-as-read endpoints
- [ ] Real-time delivery over Socket.IO
- [ ] Frontend: bell icon, unread count, notification list
- [ ] Tests: notification created for each trigger

## Phase 12 — Subscription Payments

- [ ] Choose payment provider and record in Decisions Log
- [ ] Create subscription checkout/order endpoint (monthly plan)
- [ ] Webhook endpoint with signature verification and idempotency
- [ ] Activate/renew/expire `Subscription` from verified webhook events only
- [ ] Store `Payment` records and provider references
- [ ] Restore hidden services on activation
- [ ] Frontend: subscription page, pricing, success/failure states
- [ ] Tests: webhook replay, invalid signature, state transitions

## Phase 13 — Dashboards & Frontend Polish

- [ ] Student dashboard: profile, my posts, bookmarks, messages, orders, status tracking, reviews, payments
- [ ] Freelancer dashboard: profile, work experience, services, opportunities, messages, orders, status updates, reviews, subscription, payments/earnings info
- [ ] Responsive layouts for desktop and mobile
- [ ] Loading, empty, and error states on every page
- [ ] Accessibility pass (labels, keyboard navigation, contrast)

## Phase 14 — Admin Panel

- [ ] Admin authentication and role guard
- [ ] User management (view, suspend/reactivate)
- [ ] Category and skill management
- [ ] Content moderation (services, work experience, requirements, reviews)
- [ ] View orders, subscriptions, and payment records
- [ ] Audit log of admin actions

## Phase 15 — Hardening, Testing & Deployment

- [ ] Security review against RULES.md checklist (authz on every route, rate limits, input validation)
- [ ] Full end-to-end test of the MVP acceptance criteria from the PRD
- [ ] Performance check: pagination and indexes on large seeded data
- [ ] Structured logging for auth, order, subscription, and payment events
- [ ] CI pipeline (lint, type-check, tests)
- [ ] Deploy frontend (Vercel) and backend + DB (Render/Railway) with production env vars
- [ ] Database backups and migration process documented
- [ ] Final README and `/docs` update

---

## Change Log

<!-- Append newest entries at the bottom. Format: YYYY-MM-DD | Phase X | what changed -->
