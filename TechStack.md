# Tech Stack Specification

## IEEE Computer Society Leading University Student Branch Chapter Platform

**Purpose:** This document is the technical stack and implementation contract for Claude Code. It defines the technologies, architectural boundaries, package choices, and engineering rules for the platform.

---

# 1. Technology Philosophy

Build the application as a **production-oriented modular monolith**.

The initial target is approximately **1,000 users**, so the architecture must prioritize:

- maintainability
- security
- strong relational data integrity
- clear module boundaries
- good developer experience
- performance
- accessibility
- responsive UI
- long-term historical archival
- straightforward deployment

Do **not** introduce infrastructure merely because it is commonly used in large systems.

### Initial architecture

```text
                    ┌──────────────────────────┐
                    │        Next.js            │
                    │      Web Frontend         │
                    │                          │
                    │ Public Website            │
                    │ Member Dashboard          │
                    │ Admin Dashboard            │
                    └────────────┬─────────────┘
                                 │
                            HTTPS / REST
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │         NestJS            │
                    │    Modular Monolith       │
                    │                          │
                    │ Auth / RBAC               │
                    │ Members                   │
                    │ Sessions                  │
                    │ Events                    │
                    │ Projects                  │
                    │ Achievements              │
                    │ Publications              │
                    │ Certificates              │
                    │ Moderation                │
                    │ Applications              │
                    │ Notifications             │
                    │ Audit                     │
                    └────────────┬─────────────┘
                                 │
                              Prisma
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │     PostgreSQL / Neon    │
                    │      Source of Truth      │
                    └──────────────────────────┘
```

---

# 2. Final Technology Stack

## 2.1 Frontend

| Technology | Purpose | Required |
|---|---|---|
| Next.js | Full-stack React framework / frontend | YES |
| React | UI | YES |
| TypeScript | Type safety | YES |
| Tailwind CSS | Styling | YES |
| shadcn/ui | UI component foundation | YES |
| Framer Motion | UI animation / transitions | YES |
| Lenis | Smooth scrolling | YES |
| Three.js | 3D / interactive visual experiences | YES, selectively |
| React Three Fiber | React integration for Three.js | Recommended |
| Drei | React Three Fiber helpers | Recommended |
| Zustand | Lightweight client state | YES |
| Redux Toolkit | Complex/global state when required | Optional |
| TanStack Query | Server state / API caching | YES |
| React Hook Form | Forms | YES |
| Zod | Frontend schema validation | YES |
| next/image | Image optimization | YES |
| next/font | Font optimization | YES |

---

## 2.2 Backend

| Technology | Purpose | Required |
|---|---|---|
| Node.js | Backend runtime | YES |
| NestJS | Backend framework | YES |
| TypeScript | Type safety | YES |
| REST | API architecture | YES |
| Prisma | ORM / database access | YES |
| PostgreSQL | Relational database | YES |
| Neon | Managed PostgreSQL hosting | YES |
| Swagger / OpenAPI | API documentation | YES |
| class-validator | NestJS DTO validation | YES |
| class-transformer | DTO transformation | YES |
| @nestjs/config | Environment configuration | YES |
| @nestjs/throttler | Rate limiting | YES |
| Helmet | HTTP security headers | YES |

---

## 2.3 Authentication and Storage

| Technology | Purpose |
|---|---|
| Supabase Auth | Authentication / identity |
| Supabase Storage | Images, PDFs and other files |
| Neon PostgreSQL | Application data |
| Prisma | Database access |

Authentication and application membership are separate concepts.

```text
Supabase Auth
    =
Authentication identity

members table
    =
Application member profile
```

Do not store passwords in the application database.

---

## 2.4 External Services

| Service | Purpose | Initial Requirement |
|---|---|---|
| Resend | Transactional email | Recommended |
| Sentry | Production error monitoring | Recommended |
| Vercel | Next.js deployment | Recommended |
| Railway / Render / Fly.io | NestJS deployment | Choose one |
| GitHub | Source control / CI | YES |

---

# 3. Frontend Architecture

## 3.1 Next.js

Use the current stable Next.js version available when implementation begins.

Use:

- App Router
- Server Components by default
- Client Components only where interactivity requires them
- TypeScript
- route groups
- layouts
- loading states
- error boundaries
- metadata APIs
- `next/image`
- `next/font`

Suggested structure:

```text
src/
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── dashboard/
│   ├── admin/
│   ├── api/
│   ├── layout.tsx
│   └── page.tsx
│
├── components/
│   ├── ui/
│   ├── shared/
│   ├── navigation/
│   ├── sections/
│   └── charts/
│
├── features/
│   ├── members/
│   ├── events/
│   ├── projects/
│   ├── achievements/
│   ├── publications/
│   ├── certificates/
│   └── applications/
│
├── lib/
│   ├── api/
│   ├── auth/
│   ├── utils/
│   └── validations/
│
├── hooks/
├── stores/
├── types/
└── styles/
```

Keep domain-specific UI close to its feature.

Do not create one enormous `components/` directory containing all application logic.

---

# 4. UI System

## 4.1 Tailwind CSS

Tailwind is the primary styling system.

Rules:

- Prefer utility classes.
- Keep design tokens centralized.
- Avoid arbitrary values unless genuinely necessary.
- Do not create excessive one-off CSS.
- Build reusable visual primitives.
- Keep responsive behavior intentional.

The visual system should feel like one coherent product.

---

# 5. shadcn/ui

Use **shadcn/ui** as the base component system.

Use it for:

- Button
- Input
- Textarea
- Select
- Checkbox
- Radio Group
- Dialog
- Sheet
- Drawer
- Dropdown Menu
- Popover
- Tooltip
- Tabs
- Accordion
- Card
- Table
- Badge
- Alert
- Toast
- Command
- Calendar
- Form

Components can be customized to match the chapter's visual identity.

Do not blindly use default shadcn styling everywhere.

The component library is a foundation, not the final design.

---

# 6. Animation

## 6.1 Framer Motion

Use Framer Motion for meaningful interface motion.

Good use cases:

- page transitions
- section reveal
- card hover interactions
- modal transitions
- navigation transitions
- list animations
- dashboard state changes
- scroll-triggered reveals
- micro-interactions

Avoid:

- excessive animation
- animation on every element
- long blocking animations
- unnecessary movement
- animations that reduce accessibility

Animations must respect:

```text
prefers-reduced-motion
```

The UI should remain usable when motion is reduced.

---

# 7. Smooth Scrolling

## Lenis

Use Lenis for smooth scrolling on the public marketing/landing experience where appropriate.

Do not force Lenis onto every page if it causes problems with:

- accessibility
- keyboard navigation
- nested scrolling
- modals
- admin tables
- mobile behavior

Dashboard/admin interfaces should prioritize usability over decorative scrolling.

---

# 8. Three.js

Three.js should be used selectively.

Recommended stack:

```text
Three.js
    +
React Three Fiber
    +
Drei
```

Use Three.js for:

- hero visualizations
- interactive chapter identity
- subtle 3D backgrounds
- special landing-page experiences
- visual storytelling

Do NOT use Three.js for:

- normal cards
- standard dashboard UI
- ordinary icons
- simple animations
- anything CSS can handle efficiently

Performance rules:

- lazy load heavy 3D scenes
- avoid unnecessary high-poly models
- reduce draw calls
- use optimized textures
- avoid blocking initial page rendering
- provide fallback visuals
- consider reduced-motion preferences

Three.js is an enhancement, not the foundation of the UI.

---

# 9. State Management

Use the simplest state solution that solves the problem.

## 9.1 Zustand

**Zustand is the default client-state solution.**

Use it for:

- UI preferences
- sidebar state
- modal state
- filters when shared across components
- temporary multi-step UI state
- client-only workflow state
- lightweight global state

Example conceptual structure:

```text
stores/
├── ui.store.ts
├── auth.store.ts
├── event-filter.store.ts
└── admin.store.ts
```

Do not put server data into Zustand merely because it is convenient.

---

# 10. Redux Toolkit

Redux Toolkit is optional.

Use Redux only when application state becomes sufficiently complex that centralized reducers/actions/devtools provide meaningful value.

Do not install Redux simply because the project is "large."

For this application:

```text
Simple global UI state
        ↓
Zustand

Server/API state
        ↓
TanStack Query

Complex cross-domain client state
        ↓
Redux Toolkit, only if justified
```

Avoid using Zustand and Redux for the same state.

There should be one clear owner for each piece of state.

---

# 11. Server State

Use **TanStack Query** for client-side server state.

Responsibilities:

- fetching
- caching
- refetching
- mutations
- invalidation
- loading states
- error states
- optimistic updates where appropriate

Example:

```text
Next.js Client Component
        ↓
TanStack Query
        ↓
NestJS REST API
        ↓
Prisma
        ↓
Neon PostgreSQL
```

Do not duplicate API data into Zustand/Redux unless there is a specific architectural reason.

---

# 12. Forms

Use:

```text
React Hook Form
        +
Zod
```

For:

- event creation
- event editing
- applications
- member profile editing
- project submissions
- achievement submissions
- publication submissions
- admin forms
- contact forms

Client-side validation improves UX.

Backend validation is mandatory for security and data integrity.

---

# 13. Backend Architecture

## 13.1 NestJS

NestJS is the backend framework.

It runs on Node.js.

Use a **modular monolith**, not microservices.

Suggested structure:

```text
src/
├── app.module.ts
│
├── auth/
├── members/
├── membership/
├── roles/
├── permissions/
├── sessions/
├── teams/
├── positions/
│
├── events/
├── event-registrations/
├── event-participations/
│
├── projects/
├── achievements/
├── publications/
├── posts/
│
├── submissions/
├── certificates/
├── applications/
│
├── gallery/
├── partners/
├── history/
│
├── notifications/
├── audit/
├── contact/
├── site-settings/
│
└── common/
    ├── decorators/
    ├── guards/
    ├── interceptors/
    ├── filters/
    ├── pipes/
    ├── dto/
    └── utils/
```

Modules should have clear boundaries.

---

# 14. NestJS Request Pipeline

The backend should conceptually follow:

```text
HTTP Request
     ↓
Middleware
     ↓
Security Headers
     ↓
Authentication
     ↓
Authorization / RBAC
     ↓
Validation
     ↓
Controller
     ↓
Service
     ↓
Repository / Prisma
     ↓
PostgreSQL
```

Do not place business logic inside controllers.

Controllers should remain thin.

---

# 15. Controller Responsibility

Controllers should:

- receive requests
- validate DTOs
- invoke services
- return response DTOs

Controllers should NOT:

- contain large business rules
- contain raw Prisma queries
- perform complex authorization logic directly
- duplicate service logic
- return internal database objects blindly

Example:

```text
POST /events

EventsController
      ↓
EventsService
      ↓
Prisma
```

---

# 16. Service Responsibility

Services contain business rules.

Example:

```text
EventsService
├── createEvent()
├── updateEvent()
├── publishEvent()
├── archiveEvent()
├── registerAttendee()
├── confirmParticipation()
└── getEventDetails()
```

Business rules belong here.

---

# 17. Prisma

Prisma is the ORM.

Responsibilities:

- typed database queries
- migrations
- relations
- transactions
- schema management
- database client generation

Use:

```text
Prisma
    ↓
Neon PostgreSQL
```

Do not access the database directly from frontend code.

Do not expose the Prisma client to the browser.

---

# 18. PostgreSQL / Neon

PostgreSQL is the source of truth.

Neon is the managed PostgreSQL platform.

The distinction is:

```text
PostgreSQL
= database engine

Neon
= managed PostgreSQL infrastructure
```

Use relational modeling for core entities.

Prefer:

```text
Foreign Keys
Unique Constraints
Indexes
Check Constraints
Transactions
Normalized Tables
```

Use JSONB only when data is genuinely flexible.

Do not use JSONB to avoid designing relational tables.

---

# 19. Database Rules

Follow the schema.md architecture contract.

Important rules:

- UUID primary keys.
- `snake_case` database naming.
- `TIMESTAMPTZ` for timestamps.
- UTC timestamps.
- Explicit foreign keys.
- Explicit indexes.
- Unique constraints where required.
- Conservative delete behavior.
- Soft deletion/archive for historical records where appropriate.
- Junction tables for many-to-many relationships.
- No comma-separated IDs.
- No comma-separated tags.
- No passwords in application tables.
- No media binaries in PostgreSQL.
- No hardcoded admin emails.
- No position field directly on members.
- Do not merge membership status with role.
- Do not merge registration with participation.

---

# 20. Authentication

Use Supabase Auth.

The application should distinguish:

```text
Authentication
    ↓
"Who is this user?"

Authorization
    ↓
"What is this user allowed to do?"
```

Application identity:

```text
Supabase Auth User
        ↓
members.user_id
        ↓
Application Member
```

Do not treat the authentication user record as the complete application profile.

---

# 21. RBAC

Use role-based access control.

Roles include the schema-defined concepts:

```text
MEMBER
DEPARTMENT_ADMIN
EXECUTIVE_ADMIN
SUPER_ADMIN
```

Permissions should be granular.

Examples:

```text
member.read
member.update
event.create
event.update
event.publish
event.archive
certificate.issue
certificate.revoke
submission.review
audit.read
```

Authorization should be enforced on the backend.

Frontend permission checks are for UX only.

Never rely on hiding a button as security.

---

# 22. Department Admin Scoping

Department Admin is not automatically allowed to access everything.

The backend must evaluate:

```text
User
 ↓
Role
 ↓
Permission
 ↓
Admin Scope
 ↓
Requested Resource
```

Example:

```text
Department Admin
        ↓
events.update
        ↓
Does this event/resource fall inside the admin's scope?
        ↓
Allow / Deny
```

Do not hardcode department scopes into frontend code.

---

# 23. API Design

Use REST.

Version the API:

```text
/api/v1/
```

Examples:

```text
GET    /api/v1/events
GET    /api/v1/events/:id
POST   /api/v1/events
PATCH  /api/v1/events/:id
DELETE /api/v1/events/:id

GET    /api/v1/members
GET    /api/v1/members/:id

GET    /api/v1/projects
POST   /api/v1/projects

GET    /api/v1/certificates/:verificationCode
```

Use appropriate HTTP methods and status codes.

---

# 24. API DTOs

Never return raw Prisma/database records directly from public APIs.

Use:

```text
Request DTO
Response DTO
```

Example:

```text
Database Entity
      ↓
Service
      ↓
Response DTO
      ↓
API
```

This prevents accidental exposure of:

- private fields
- internal IDs where inappropriate
- audit metadata
- moderation internals
- storage internals
- sensitive user information

Never use `SELECT *` as a public API strategy.

---

# 25. API Documentation

Use Swagger/OpenAPI.

Expose development API documentation such as:

```text
/api/docs
```

Document:

- endpoints
- request DTOs
- response DTOs
- authentication
- status codes
- parameters
- validation errors

Swagger is part of the development contract, not merely decoration.

---

# 26. Validation

NestJS should use DTO validation.

Recommended:

```text
class-validator
class-transformer
```

Use a global validation pipe.

Conceptually:

```text
Request
   ↓
DTO transformation
   ↓
Validation
   ↓
Controller
```

Reject invalid input before business logic executes.

---

# 27. Security

Initial backend security stack:

```text
Helmet
CORS
Rate Limiting
DTO Validation
Authentication
RBAC
Scope Checks
Secure Headers
Environment Variables
```

Never commit:

```text
DATABASE_URL
SUPABASE_SERVICE_ROLE_KEY
API keys
JWT secrets
private credentials
```

Use environment variables/secrets.

---

# 28. Rate Limiting

Use NestJS throttling.

Protect public/high-risk endpoints such as:

```text
Authentication-related endpoints
Contact form
Application submission
Certificate verification
Public APIs
Search endpoints
```

Initial deployment can use in-memory throttling if the backend runs as a single instance.

Do not introduce Redis solely for rate limiting at this scale.

If the application later becomes multi-instance, move shared rate-limit state to an appropriate shared store.

---

# 29. Redis Policy

**Redis is NOT part of the initial required stack.**

Do not add Redis unless an actual requirement appears.

Possible future uses:

```text
Redis
├── distributed caching
├── distributed rate limiting
├── background job infrastructure
├── BullMQ
├── distributed locks
└── high-volume ephemeral state
```

At approximately 1,000 users, PostgreSQL + proper indexes + application-level caching is sufficient for the initial architecture.

---

# 30. Background Jobs

Do not build a queue system until asynchronous work justifies it.

Potential future jobs:

```text
Send email
Generate certificates
Send notifications
Process uploaded files
Generate reports
Perform scheduled reminders
Recalculate expensive statistics
```

If background jobs become necessary:

```text
NestJS
   ↓
BullMQ
   ↓
Redis
   ↓
Worker
```

Introduce this only when required.

---

# 31. Email

Use a transactional email provider such as Resend.

Potential emails:

```text
Application received
Application approved
Application rejected
Event registration confirmation
Event reminder
Submission status change
Certificate issued
Contact acknowledgement
```

Do not run your own mail server.

Email sending should eventually be abstracted behind an application service:

```text
EmailService
     ↓
Resend
```

This prevents vendor-specific code from spreading through modules.

---

# 32. File Storage

Use Supabase Storage.

Database stores metadata:

```text
file_id
bucket
storage_path
mime_type
file_size
created_at
```

Storage contains:

```text
Member photos
Event banners
Project covers
Gallery images
Certificate PDFs
Publication PDFs
Achievement proofs
```

Never store large binary files directly in PostgreSQL.

Separate public and private storage.

Private proof documents must not be publicly accessible.

Use signed/protected URLs when necessary.

---

# 33. Error Monitoring

Use Sentry after the production application becomes active.

Track:

- backend exceptions
- frontend runtime errors
- failed requests
- important context
- release/deployment information

Do not send sensitive personal data unnecessarily.

---

# 34. Logging

Use structured application logging.

Useful fields:

```text
request_id
user_id
route
method
status_code
response_time
error_code
timestamp
```

Avoid logging:

```text
passwords
access tokens
private keys
sensitive personal data
```

Do not use unrestricted `console.log()` as the production observability strategy.

---

# 35. Caching Strategy

Do not immediately add Redis.

Use caching at the appropriate layer.

Potential cache targets:

```text
Public event listings
Published posts
Public member directory
Homepage statistics
Navigation configuration
Site settings
```

For expensive homepage statistics, use a cached/materialized approach rather than executing many independent aggregate queries on every request.

---

# 36. Search

Start with PostgreSQL search.

For approximately 1,000 users, do not introduce Elasticsearch/OpenSearch.

Possible tools:

```text
PostgreSQL indexes
ILIKE
pg_trgm
Full-text search where appropriate
```

For member name search, `pg_trgm` can support fuzzy matching.

Introduce a dedicated search engine only when the actual workload requires it.

---

# 37. State Ownership Rules

Every piece of state must have one clear owner.

### Server state

```text
TanStack Query
```

### Lightweight client/UI state

```text
Zustand
```

### Complex client state

```text
Redux Toolkit only if justified
```

### Form state

```text
React Hook Form
```

### Persistent application data

```text
PostgreSQL
```

Do not duplicate the same source of truth across multiple stores without a specific reason.

---

# 38. Frontend ↔ Backend Boundary

The frontend must communicate with the backend through defined APIs.

```text
Next.js
    ↓
API Client
    ↓
NestJS
```

Create a centralized API client.

Example conceptual structure:

```text
lib/
└── api/
    ├── client.ts
    ├── members.ts
    ├── events.ts
    ├── projects.ts
    ├── achievements.ts
    ├── publications.ts
    └── certificates.ts
```

Do not scatter raw URLs throughout React components.

---

# 39. Type Safety

Use TypeScript across frontend and backend.

The goal is:

```text
Frontend TypeScript
        ↓
API contract
        ↓
Backend TypeScript
        ↓
Prisma types
```

Where practical, generate or share API types from OpenAPI rather than manually duplicating large interfaces.

Do not blindly share backend database types with the frontend.

Database models and public API DTOs are different abstractions.

---

# 40. Accessibility

Accessibility is mandatory.

Follow:

- semantic HTML
- keyboard navigation
- focus management
- accessible labels
- sufficient contrast
- screen-reader support
- reduced motion support
- accessible dialogs
- accessible forms
- meaningful error messages

shadcn/ui components should still be tested and customized correctly.

Do not sacrifice accessibility for animation.

---

# 41. Performance

Frontend performance priorities:

```text
Fast initial load
Small client bundles
Server Components by default
Optimized images
Lazy-loaded 3D
Lazy-loaded heavy components
Minimal unnecessary JavaScript
Good caching
Responsive images
```

Three.js and animation libraries must not unnecessarily dominate the initial bundle.

---

# 42. SEO

The public website should use Next.js SEO capabilities.

Implement:

- metadata
- title templates
- descriptions
- Open Graph metadata
- Twitter/X metadata where relevant
- canonical URLs
- sitemap
- robots
- structured metadata where useful

Public pages should be crawlable unless intentionally private.

Dashboard/admin pages should not be indexed.

---

# 43. Deployment

Recommended deployment model:

```text
GitHub
   │
   ├───────────────────┐
   │                   │
   ▼                   ▼
Vercel              Railway/
                     Render/
                     Fly.io
   │                   │
Next.js              NestJS
   │                   │
   └─────────┬─────────┘
             │
             ▼
       Neon PostgreSQL
             │
             └── Supabase Auth
             └── Supabase Storage
```

Choose one backend host.

Do not build unnecessary Kubernetes infrastructure.

---

# 44. Environment Strategy

Use separate environments where practical:

```text
Development
Staging
Production
```

Environment variables should be explicitly documented.

Example:

```text
DATABASE_URL
DIRECT_DATABASE_URL
SUPABASE_URL
SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
API_BASE_URL
RESEND_API_KEY
SENTRY_DSN
```

Only expose variables prefixed with `NEXT_PUBLIC_` when they are genuinely safe for browser exposure.

Never expose the Supabase service-role key to the browser.

---

# 45. Git and CI

Use GitHub.

Recommended checks on pull requests:

```text
TypeScript
ESLint
Tests
Build
Prisma validation
```

Conceptually:

```text
Pull Request
     ↓
Lint
     ↓
Type Check
     ↓
Unit Tests
     ↓
Build
     ↓
Review
     ↓
Merge
```

Database migrations must be version-controlled.

---

# 46. Testing Stack

Use:

```text
Jest
Supertest
```

Test important business logic.

Priority:

```text
RBAC
Permission checks
Department scoping
Event lifecycle
Registration
Participation
Certificate verification
Certificate revocation
Submission workflow
Application workflow
Historical position logic
```

Do not only test CRUD.

Test the rules that protect the integrity of the organization.

---

# 47. Architecture Principles

## 47.1 Modular Monolith

The backend starts as one deployable NestJS application.

```text
NestJS
├── Auth
├── Members
├── Events
├── Projects
├── Achievements
├── Publications
├── Certificates
├── Moderation
└── Platform
```

Modules communicate through services/contracts.

Do not create microservices prematurely.

---

## 47.2 Database First

The relational database is the source of truth.

Business logic must respect the schema.

Do not create frontend-only pseudo-databases.

---

## 47.3 Historical Integrity

Historical records matter.

Do not destroy organizational history merely because a record is no longer active.

Use:

```text
status
archive
soft delete
effective dates
history tables
```

where appropriate.

---

# 48. What We Are NOT Using Initially

Do not add these without a demonstrated requirement:

```text
Redis
Kafka
RabbitMQ
Elasticsearch
OpenSearch
MongoDB
ClickHouse
Kubernetes
Microservices
GraphQL
Event Sourcing
CQRS
Service Mesh
Dedicated API Gateway
Dedicated Search Cluster
```

These technologies can be introduced later if real requirements justify them.

---

# 49. Package Philosophy

Do not install packages simply because they are popular.

Before adding a dependency, ask:

1. What problem does it solve?
2. Can the existing stack solve it?
3. Does it introduce another source of truth?
4. Does it increase bundle size?
5. Does it increase operational complexity?
6. Is it actively maintained?
7. Is it necessary for the current scale?

Prefer fewer well-understood dependencies.

---

# 50. Recommended Initial Dependency Groups

## Frontend

```text
next
react
react-dom
typescript

tailwindcss
shadcn/ui components

framer-motion
lenis

three
@react-three/fiber
@react-three/drei

zustand

@tanstack/react-query

react-hook-form
zod

lucide-react
```

Use only the shadcn/ui components actually required by the application.

---

## Backend

```text
@nestjs/common
@nestjs/core
@nestjs/config
@nestjs/swagger
@nestjs/throttler

class-validator
class-transformer

helmet

prisma
@prisma/client
```

Add authentication-related packages according to the final Supabase Auth integration approach.

---

# 51. Architecture Decision Summary

The initial production stack is:

```text
┌───────────────────────────────────────────┐
│                  FRONTEND                 │
├───────────────────────────────────────────┤
│ Next.js                                   │
│ React + TypeScript                        │
│ Tailwind CSS                              │
│ shadcn/ui                                 │
│ Framer Motion                             │
│ Lenis                                     │
│ Three.js + R3F + Drei                     │
│ Zustand                                   │
│ TanStack Query                             │
│ React Hook Form + Zod                     │
└──────────────────────┬────────────────────┘
                       │
                    REST API
                       │
┌──────────────────────▼────────────────────┐
│                  BACKEND                  │
├───────────────────────────────────────────┤
│ Node.js                                   │
│ NestJS                                    │
│ TypeScript                                │
│ REST                                      │
│ Prisma                                    │
│ Swagger / OpenAPI                         │
│ class-validator                           │
│ Helmet                                    │
│ Throttling                                │
└──────────────────────┬────────────────────┘
                       │
                    Prisma
                       │
┌──────────────────────▼────────────────────┐
│                 DATABASE                  │
├───────────────────────────────────────────┤
│ PostgreSQL                                │
│ Neon                                      │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│             PLATFORM SERVICES             │
├───────────────────────────────────────────┤
│ Supabase Auth                             │
│ Supabase Storage                          │
│ Resend                                    │
│ Sentry                                    │
│ GitHub                                    │
└───────────────────────────────────────────┘
```

---

# 52. Claude Code Implementation Rules

Claude Code must follow these rules while implementing the project:

### Rule 1
Do not replace NestJS with Express unless explicitly instructed.

### Rule 2
Do not replace PostgreSQL with MongoDB or another database.

### Rule 3
Do not add Redis unless a concrete requirement is identified.

### Rule 4
Do not create microservices.

### Rule 5
Do not put business logic in Next.js when that logic belongs to the NestJS backend.

### Rule 6
Do not access PostgreSQL directly from the browser.

### Rule 7
Do not expose Prisma to frontend code.

### Rule 8
Do not store passwords in the application database.

### Rule 9
Do not hardcode administrator emails.

### Rule 10
Do not rely on frontend permission checks for security.

### Rule 11
Every protected operation must be authorized server-side.

### Rule 12
Do not duplicate server state into Zustand or Redux unnecessarily.

### Rule 13
Do not use Redux if Zustand + TanStack Query already solves the requirement.

### Rule 14
Do not use Three.js for UI that can be implemented efficiently with HTML/CSS.

### Rule 15
Do not make every page use Lenis automatically.

### Rule 16
Animations must not compromise accessibility or performance.

### Rule 17
Do not store media binaries in PostgreSQL.

### Rule 18
Do not use `SELECT *` in public API implementations.

### Rule 19
Do not delete historical organizational data without an explicit requirement.

### Rule 20
Do not bypass Prisma migrations.

### Rule 21
Do not create a new dependency when an existing package already solves the problem.

### Rule 22
Do not introduce infrastructure for hypothetical future scale.

### Rule 23
Keep modules independent and clearly bounded.

### Rule 24
Use DTOs between the API and database layers.

### Rule 25
Keep controllers thin and services responsible for business logic.

### Rule 26
Validate input on both frontend and backend.

### Rule 27
Treat PostgreSQL as the source of truth.

### Rule 28
Use server-side authorization as the final security boundary.

### Rule 29
Keep public and private data models separate.

### Rule 30
Optimize only after identifying an actual performance bottleneck.

---

# 53. Definition of Done for the Stack

The implementation is technically complete when:

- Next.js frontend is working.
- NestJS backend is working.
- PostgreSQL/Neon connection is stable.
- Prisma migrations are reproducible.
- Supabase Auth integration is secure.
- Supabase Storage is configured.
- REST API is versioned.
- Swagger documentation exists.
- DTO validation works.
- RBAC works server-side.
- Department admin scoping works server-side.
- Public/private response boundaries are enforced.
- Rate limiting is enabled for appropriate endpoints.
- Error handling is centralized.
- Logging is structured.
- Tests cover critical business rules.
- Production environment variables are secure.
- CI checks lint/type/test/build.
- No unnecessary Redis/cache/search infrastructure exists.
- Three.js is lazy-loaded where appropriate.
- Animations respect reduced-motion preferences.
- Frontend state ownership is clearly defined.
- Database remains the source of truth.
- Historical records are preserved appropriately.

---

# 54. Final Technology Decision

For this project, use:

```text
Frontend:
Next.js + TypeScript
Tailwind CSS
shadcn/ui
Framer Motion
Lenis
Three.js + React Three Fiber + Drei
Zustand
TanStack Query
React Hook Form
Zod

Backend:
Node.js
NestJS
TypeScript
REST
Prisma
Swagger/OpenAPI
class-validator
class-transformer
Helmet
NestJS Throttler

Database:
PostgreSQL
Neon

Authentication:
Supabase Auth

Storage:
Supabase Storage

Email:
Resend

Monitoring:
Sentry

Deployment:
Vercel + Railway/Render/Fly.io

Source Control / CI:
GitHub

Initial Infrastructure:
NO REDIS
NO KAFKA
NO RABBITMQ
NO ELASTICSEARCH
NO MICROSERVICES
NO KUBERNETES
```

This stack is intentionally powerful enough for a serious production application while remaining appropriate for the expected scale and complexity of the IEEE Computer Society Leading University Student Branch Chapter platform.
