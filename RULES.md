# RULES.md — Rules for the AI Agent

> Read this file and `TRACKER.md` at the start of every session. These rules override convenience.
> If a task conflicts with a rule, stop and ask the human instead of guessing.

## 1. Project Summary

A two-sided marketplace for students and freelancers.
- Students find and hire freelancers, and post work requirements.
- Freelancers publish Services and Work Experience, browse requirements, and click "I'm Interested" with an optional price.
- Revenue: freelancer monthly subscription only (7-day free trial first). No order commission.
- Built-in chat, orders with status tracking, mutual reviews, file exchange, in-app notifications.
- Disputes are out of V1 scope.

## 2. Tech Stack (do not change without asking)

- Frontend: Next.js + React + Tailwind CSS
- Backend: Node.js + Express (TypeScript)
- Database: PostgreSQL + Prisma
- Auth: JWT in HTTP-only cookies; Argon2id for passwords
- Real-time: Socket.IO
- Validation: Zod (backend always; frontend for UX)
- Files: Cloudinary or S3-compatible storage (never binary in PostgreSQL)
- Redis only when needed (rate limiting, caching)
- Deploy: Vercel (frontend) + Render/Railway (backend, DB)

## 3. Workflow Rules

1. Before coding: read `RULES.md` and `TRACKER.md`, then state which phase and items you are working on.
2. Work on **one phase at a time**, in order. Do not start a later phase unless the human says so.
3. Make small, reviewable changes. Prefer several focused commits over one huge change.
4. After finishing: run lint, type-check, and tests; fix failures before reporting done.
5. Update `TRACKER.md` (checkboxes, Phase Status, Current Focus, Change Log) at the end of every session.
6. Log every new technical or product decision in the **Decisions Log** below.
7. Do not add features, libraries, or scope that are not in the tracker or docs. Suggest them instead.
8. If a requirement is unclear or missing, ask. Never invent business rules silently.
9. Never leave half-working code without marking it `[~]` or `[!]` in the tracker and noting what is missing.
10. Do not delete or rewrite existing working code unless the task requires it; explain why if you do.

## 4. Architecture Rules

- Layered backend: `routes → controllers → services → data access (Prisma)`. Business logic lives in **services**, never in route handlers.
- Version all APIs under `/api/v1/`.
- Consistent response and error format across all endpoints.
- Paginate every list endpoint (marketplace, messages, notifications, orders).
- Keep backend instances stateless where practical.
- The frontend never contains secrets and never makes privileged decisions (trial eligibility, limits, authorization).
- Isolate swappable logic in its own module: ranking formula, order transition matrix, entitlement policy.

## 5. Security Rules (non-negotiable)

- Enforce authorization in the backend on **every** protected route, using both account type and resource ownership. Knowing an ID must never grant access.
- Validate and sanitize all input with Zod on the backend.
- Never store plaintext passwords or OTPs; hash both. Never return hashes, OTP secrets, or tokens in API responses.
- Rate-limit: OTP requests, OTP verification, login, and username availability checks.
- Use HTTP-only, secure, same-site cookies; HTTPS in production.
- Time-sensitive logic (trial, subscription, OTP expiry) uses **server time only**.
- Payment provider secrets stay on the backend only. Verify webhook signatures and handle events idempotently.
- Validate file uploads (type and size); use access-controlled download URLs.
- Never log secrets, passwords, OTPs, tokens, or full payment details.
- Never commit `.env` files or real credentials.

## 6. Data & Transaction Rules

- `users.username` is globally unique across all account types, normalized (lowercase, no spaces, restricted charset). The database UNIQUE constraint is the final authority; do not rely on a pre-check alone.
- Use database transactions for multi-step operations:
  - Register account (verify OTP → create account/profile → start trial for freelancers)
  - Create service (check eligibility → count → enforce limit → create)
  - Accept proposal (verify open → mark accepted → close others → create order → close requirement)
  - Complete order (verify state → record → unlock reviews)
- Enforce invariants with constraints, not just code: one accepted proposal/order per requirement; no duplicate active proposal per freelancer per requirement; one review per reviewer per order.
- `OrderStatusHistory` is append-only.
- Orders store a **snapshot** (title, terms, agreed price). Later edits to a service or requirement must never change an existing order.
- Soft-hide rather than delete content when entitlement lapses.

## 7. Business Rules (from the PRD and System Design)

- BR-01: Student and Freelancer accounts are separate accounts.
- BR-02/03: The freelancer trial lasts 7 days from freelancer account creation and is never paused, reset, or extended by student-account usage.
- BR-04/05: After expiry, freelancers can still browse; creating services and "I'm Interested" require an active trial/subscription.
- Service limits: **3** during trial, **10** with an active subscription. Enforced server-side.
- Existing services are never deleted on expiry; they are hidden/inactive and restored on resubscription.
- Students can post requirements with either a fixed price or a budget/range.
- Freelancers can express interest with an optional proposed price.
- Orders are created only when a student accepts a proposal. A requirement has at most one accepted freelancer/order in V1.
- Students and freelancers can each have multiple active orders.
- Freelancers update order status; students view it.
- Both sides can review each other, only after a COMPLETED order. Freelancer ratings influence ranking.
- No order commission. The platform does **not** hold project funds. The payment gateway is used for **subscription payments**.
- No dispute system in V1.
- Order lifecycle (recommended): `CREATED → IN_PROGRESS → SUBMITTED → COMPLETED`, with `SUBMITTED → REVISION_REQUESTED → IN_PROGRESS`. Cancellation allowed from appropriate pre-completion states. Invalid transitions must be rejected by the backend.

## 8. Code Quality Rules

- TypeScript strict mode; no `any` unless justified in a comment.
- Small, single-purpose functions and files. Descriptive names. No dead code or leftover debug logs.
- Handle errors explicitly; return proper HTTP status codes and clear messages.
- Write tests for every business rule above, especially: username race, trial timing, service limits, proposal acceptance race, order transitions, authorization checks, review eligibility.
- Every page needs loading, empty, and error states.
- UI must be responsive (desktop and mobile) and keyboard-accessible.
- Comments explain *why*, not *what*.
- Follow the existing folder structure and naming conventions; do not reorganize without asking.

## 9. Git Rules

- Branch per phase or feature: `phase-N/short-description`.
- Conventional commit messages (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`).
- Never force-push to `main`. Never commit generated secrets or build artifacts.

## 10. Definition of Done (for any tracker item)

- Code implemented and follows these rules.
- Input validation and authorization in place.
- Tests written and passing; lint and type-check clean.
- Works end to end (backend + frontend, if applicable).
- `TRACKER.md` updated; decisions logged.

## 11. Decisions Log

Defaults below resolve conflicts found between the Main Idea, PRD, and System Design. **The human should confirm or change them.** Add new decisions at the bottom.

| # | Topic | Default decision (pending confirmation) |
| --- | --- | --- |
| D1 | Project payments | Follow System Design: non-custodial, no platform-held funds. Gateway handles subscription payments only. |
| D2 | Hiring directly from a service | Student's "Hire" on a service creates a Proposal (source = SERVICE); the freelancer accepts it, which creates the Order with a service snapshot. |
| D3 | Work Experience after expiry | Existing entries stay visible; creating new ones requires active trial/subscription. No count limit. |
| D4 | Expired freelancer visibility | Services are hidden from student marketplace; profile and work experience remain viewable. Restored on resubscription. |
| D5 | Bookmarks | Students can bookmark freelancers; bookmarking services is optional/secondary. |
| D6 | Chat timing | Chat can start from a profile, service, requirement, or order, before any proposal or order exists. |
| D7 | Student signup | Phone OTP or email OTP, then username + password. |
| D8 | Subscription billing | Monthly interval; price and provider TBD. |
| D9 | Reviews | One review per reviewer per COMPLETED order. |
| D10 | Ranking | Initial formula uses average rating, review count, completed orders; kept in one replaceable module. |

<!-- Append new decisions below: | D11 | Topic | Decision | -->

## 12. Open Questions (ask the human before implementing)

- Subscription price and payment provider
- Exact username character and length rules
- Order cancellation rules per state and who can cancel
- Maximum file size and allowed file types
- Final category/subcategory administration model
- Detailed admin capabilities
