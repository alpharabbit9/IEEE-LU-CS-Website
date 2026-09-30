# IEEE CS LU SB Chapter Website — Database Schema Specification

> **Purpose:** This document is the database source of truth for Claude Code.
>
> **Read this before creating or modifying database models, migrations, queries, server actions, API routes, RLS policies, or admin workflows.**
>
> **Primary stack assumption:** PostgreSQL + Supabase, with Next.js on the application layer and Supabase Auth/Storage where appropriate.
>
> **Scale target:** approximately 1,000 users initially, with multi-year historical retention. The design prioritizes data integrity, maintainability, auditability, and archival correctness rather than premature scaling complexity.

---

## 1. Product Context

This website is the permanent digital home and historical record of the **IEEE Computer Society Leading University Student Branch Chapter**.

It has three responsibilities:

1. **Public face** — chapter information, activities, events, members, partners, etc.
2. **Archive** — preserve events, achievements, publications, photos, projects, leadership history, and milestones.
3. **Member system** — a member profile is generated from real records such as positions, event participation, achievements, certificates, publications, projects, and posts.

The database must therefore be designed as a **historical relational system**, not merely as a CMS.

### Core principle

Historical data must remain meaningful after committee turnover.

Do not design around only the current committee. A member can have multiple positions, multiple positions simultaneously, multiple sessions, many event participations, and years of contributions.

---

# 2. Non-Negotiable Design Rules

These rules apply to all database and application code.

## 2.1 PostgreSQL is the source of truth

Use relational PostgreSQL tables and foreign keys for structured relationships.

Do not turn the database into a collection of large JSON documents.

Use JSONB only where the structure is genuinely flexible, such as notification payloads or site configuration.

---

## 2.2 Authentication and member profile are different concepts

Authentication identity belongs to Supabase Auth.

A member profile belongs to the application's `members` table.

Recommended relationship:

```text
auth.users
    |
    | 1:1 / 0:1
    v
members
```

Do not store passwords in the application database when Supabase Auth is used.

---

## 2.3 Role and membership status are different

NEVER combine these concepts.

### Role

Describes what a user is allowed to do:

- Member
- Department Admin
- Executive Admin
- Super Admin

### Membership status

Describes the person's relationship with the chapter:

- Applicant
- Active
- Alumni
- Inactive

A person can therefore be:

```text
membership_status = ALUMNI
role = EXECUTIVE_ADMIN
```

Do not create logic such as:

```text
if membership_status == "admin"
```

That is incorrect.

---

## 2.4 Never store current position directly on members

DO NOT create:

```text
members.position
members.current_position
```

Positions are historical records.

Correct:

```text
members
   |
   v
positions
   |
   +-- position_title
   +-- team
   +-- session
   +-- start_date
   +-- end_date
```

A member may:

- hold multiple positions over time
- hold more than one position simultaneously
- hold positions across multiple sessions

---

## 2.5 Registration is NOT participation

An event registration means:

> The person registered for the event.

A participation means:

> An admin confirmed that the person actually participated.

Therefore these must remain separate:

```text
event_registrations
event_participations
```

Do not convert every registration automatically into participation.

---

## 2.6 Journey is derived data

The member journey must NOT be stored as a text field or manually maintained array.

It is generated from:

```text
positions
event_participations
achievements
certificates
```

Potentially other contribution records can be added later.

The application should query/compose these records into a chronological journey.

---

## 2.7 Historical records should not be destroyed

Normal admin deletion is a soft delete/archive operation for historical/content entities.

Hard deletion is a Super Admin-level exceptional operation.

Never casually delete:

- old positions
- old events
- old participations
- old certificates
- old achievements
- old publications
- old projects
- old applications
- historical partners
- history entries

The goal is long-term chapter archival.

---

## 2.8 Files do not belong in PostgreSQL

Do not store image/PDF binaries in database columns.

Use Supabase Storage or another object-storage provider.

Database stores references such as:

```text
storage_path
file_url
```

Use storage for:

- member photos
- event banners
- gallery images
- achievement proof files
- publication PDFs
- certificate files
- history images

---

## 2.9 Server-side authorization is mandatory

Frontend UI visibility is NOT access control.

Every mutation must be checked server-side.

Supabase RLS should be used where appropriate, especially for member-owned data.

---

## 2.10 Audit destructive and publishing actions

Every important administrative action should produce an audit record containing:

- actor
- action
- entity
- entity ID
- before state
- after state
- timestamp

---

# 3. Logical Database Domains

Organize the schema conceptually into these domains.

```text
IDENTITY & ACCESS
    users/auth
    roles
    permissions
    user_roles
    role_permissions
    content_domains
    admin_scopes

MEMBERS
    members
    member_links
    membership_statuses
    membership_status_history

ORGANIZATION
    sessions
    teams
    position_titles
    positions

EVENTS
    events
    event_categories
    event_registration_fields
    event_registrations
    event_participation_roles
    event_participations
    event_speakers

CONTENT
    posts
    post_types
    tags
    post_tags
    projects
    project_categories
    project_members
    achievement_categories
    achievements
    publication_types
    publications
    publication_authors

MODERATION
    submissions
    submission_status_history

CERTIFICATES
    certificates

ARCHIVE
    gallery_albums
    gallery_items
    partners
    history_entries

PLATFORM
    applications
    notifications
    audit_logs
    site_settings
    navigation_items
    contact_messages
```

---

# 4. Naming Conventions

Use consistent PostgreSQL naming.

- table names: `snake_case`, plural
- column names: `snake_case`
- primary key: `id`
- foreign keys: `<table_singular>_id`
- timestamps: `created_at`, `updated_at`
- deletion timestamp: `deleted_at`
- URL slug: `slug`
- booleans: `is_*` or `has_*`
- dates: `*_date`
- timestamps: `*_at`

Examples:

```text
member_id
event_id
session_id
created_at
updated_at
deleted_at
is_active
published_at
```

Avoid inconsistent names such as:

```text
created
createdDate
creation_date
createdTimestamp
```

---

# 5. Primary Key Strategy

Use UUIDs for application entities.

Recommended:

```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

Do not expose sequential database IDs for public resources such as:

- certificates
- members
- events
- projects

Slugs and verification codes are public identifiers where needed.

---

# 6. Timestamp Strategy

Use:

```text
TIMESTAMPTZ
```

for timestamps.

Store all server/database timestamps in UTC.

Convert to local time only at the UI/presentation layer.

Events specifically use `TIMESTAMPTZ` because they contain real-world date/time information.

Use:

```text
DATE
```

when only a calendar date matters, such as:

```text
joining_date
alumni_since
achievement_date
```

---

# 7. Identity & Access Schema

## 7.1 `members`

This is the main profile table.

```text
members
-----------------------------------------------
id                         UUID PK
user_id                    UUID UNIQUE NULL
slug                       VARCHAR UNIQUE NOT NULL
full_name                  VARCHAR NOT NULL
preferred_name             VARCHAR NULL
photo_url                  TEXT NULL
bio                        TEXT NULL

university                 VARCHAR NULL
department                 VARCHAR NULL
batch                      VARCHAR NULL

joining_date               DATE NULL
membership_status_id       UUID FK
alumni_since               DATE NULL

ieee_membership_status     VARCHAR NULL
ieee_cs_membership_status  VARCHAR NULL

privacy_settings           JSONB NOT NULL DEFAULT '{}'

created_at                 TIMESTAMPTZ NOT NULL
updated_at                 TIMESTAMPTZ NOT NULL
deleted_at                 TIMESTAMPTZ NULL
```

### Constraints

```text
UNIQUE(slug)
UNIQUE(user_id)
```

`user_id` can be nullable because:

- an applicant does not necessarily have an account
- historical/alumni records may exist before account creation
- member records may be imported before login is provisioned

### Slug rule

Member slugs are unique and immutable once issued.

Example:

```text
john-doe
john-doe-2
```

Changing a display name must NOT automatically change the slug.

---

## 7.2 `membership_statuses`

Use a lookup table rather than scattering string literals.

```text
membership_statuses
-------------------------
id
code UNIQUE
name
description
display_order
is_active
```

Seed:

```text
APPLICANT
ACTIVE
ALUMNI
INACTIVE
```

---

## 7.3 `membership_status_history`

Keep status transitions.

```text
membership_status_history
--------------------------------
id
member_id
from_status_id NULL
to_status_id
changed_by
changed_at
reason
```

This allows historical questions such as:

- When did the member become Alumni?
- Who changed the status?
- What was the previous status?

Do not rely only on `alumni_since`.

---

## 7.4 `member_links`

```text
member_links
--------------------------------
id
member_id
type
url
display_order
created_at
updated_at
```

Examples:

```text
github
linkedin
portfolio
researchgate
facebook
x
other
```

---

# 8. RBAC

## 8.1 `roles`

```text
roles
-------------------------
id
code UNIQUE
name
description
```

Seed:

```text
MEMBER
DEPARTMENT_ADMIN
EXECUTIVE_ADMIN
SUPER_ADMIN
```

Visitor does not need a DB role because unauthenticated access is represented by absence of an authenticated user.

---

## 8.2 `permissions`

```text
permissions
-------------------------
id
code UNIQUE
description
```

Examples:

```text
member.read
member.update
member.manage

event.read
event.create
event.update
event.delete
event.publish
event.participation.manage

post.create
post.update
post.review
post.publish

project.create
project.review
project.publish

achievement.create
achievement.review
achievement.publish

publication.create
publication.review
publication.publish

certificate.issue
certificate.revoke

application.read
application.review

admin.manage
settings.manage
audit.read
```

Do not hardcode authorization logic around role names when a permission-based check is more appropriate.

---

## 8.3 `user_roles`

```text
user_roles
--------------------------------
user_id
role_id
assigned_by
created_at

PRIMARY KEY(user_id, role_id)
```

A user can have multiple roles if needed.

---

## 8.4 `role_permissions`

```text
role_permissions
--------------------------------
role_id
permission_id

PRIMARY KEY(role_id, permission_id)
```

---

## 8.5 `content_domains`

```text
content_domains
--------------------------------
id
code UNIQUE
name
description
```

Example:

```text
EVENTS
GALLERY
POSTS
PROJECTS
ACHIEVEMENTS
PUBLICATIONS
SPONSORS
COLLABORATIONS
HISTORY
```

---

## 8.6 `admin_scopes`

Department Admin permissions must be scoped through data.

```text
admin_scopes
--------------------------------
user_id
domain_id
created_at

PRIMARY KEY(user_id, domain_id)
```

Do not hardcode:

```text
if user.email == "x" then gallery admin
```

---

# 9. Organization Schema

## 9.1 `sessions`

Sessions are first-class entities.

```text
sessions
--------------------------------
id
name
start_year
end_year
start_date
end_date
is_current
created_at
updated_at
```

Examples:

```text
2024-2025
2025-2026
2026-2027
```

A position belongs to a session.

---

## 9.2 `teams`

```text
teams
--------------------------------
id
name
slug UNIQUE
description
display_order
is_active
created_at
updated_at
```

---

## 9.3 `position_titles`

```text
position_titles
--------------------------------
id
name
slug UNIQUE
display_order
level
is_active
created_at
updated_at
```

Examples:

```text
Chairperson
Vice Chairperson
General Secretary
Treasurer
Technical Secretary
Media Secretary
Research Secretary
Executive Member
```

Do not hardcode display ordering in application code.

---

## 9.4 `positions`

```text
positions
--------------------------------
id
member_id
position_title_id
team_id NULL
session_id

start_date
end_date NULL

description NULL
responsibilities NULL

created_at
updated_at
```

`end_date IS NULL` means the position is currently active.

Do NOT store:

```text
is_current
```

because it is derived from the date.

Never delete a historical position just because it ended.

---

# 10. Event Schema

## 10.1 `event_categories`

```text
event_categories
--------------------------------
id
name
slug UNIQUE
description
display_order
is_active
created_at
updated_at
```

Examples:

```text
Workshop
Seminar
Competition
Hackathon
Conference
Webinar
Training
Community Activity
Social Event
Orientation
Research Event
```

Categories are DB records, not hardcoded enums.

---

## 10.2 `events`

```text
events
--------------------------------
id
slug UNIQUE
title
summary
description

category_id

banner_url

start_at
end_at

venue
is_online
online_url

capacity NULL

registration_enabled
registration_open_at NULL
registration_close_at NULL

show_participants

report_content NULL
report_file_url NULL

published_at NULL

created_by
updated_by

created_at
updated_at
deleted_at
```

### Event status

Do NOT make manually editable `UPCOMING`, `ONGOING`, `COMPLETED` status the source of truth.

Derive it from dates:

```text
now < start_at
    => UPCOMING

start_at <= now <= end_at
    => ONGOING

now > end_at
    => COMPLETED
```

Expose this through application logic or a database view.

### Registration status

Registration state is different from event lifecycle.

Registration can be:

```text
Coming Soon
Open
Closed
Full
```

This should be derived/validated from:

- registration enabled
- open/close dates
- capacity

Do not allow the UI to create contradictory states.

---

## 10.3 `event_registration_fields`

Each event can configure which registration fields are enabled/required.

```text
event_registration_fields
--------------------------------
id
event_id
field_key
is_enabled
is_required
display_order
```

Possible keys:

```text
name
email
student_id
department
batch
phone
```

This makes future fields possible without schema changes.

---

## 10.4 `event_registrations`

```text
event_registrations
--------------------------------
id
event_id
member_id NULL

name
email
student_id NULL
department NULL
batch NULL
phone NULL

status

registered_at
created_at
updated_at
```

The registration must preserve the submitted snapshot of information.

Do not rely solely on the current member profile because the member's:

- department
- batch
- name
- phone

may change later.

---

## 10.5 `event_participation_roles`

```text
event_participation_roles
--------------------------------
id
code UNIQUE
name
description
display_order
is_active
```

Seed:

```text
PARTICIPANT
VOLUNTEER
ORGANIZER
SPEAKER
MENTOR
JUDGE
```

---

## 10.6 `event_participations`

```text
event_participations
--------------------------------
id
event_id
member_id NULL

external_name NULL
external_email NULL

role_id

confirmed_by
confirmed_at

created_at
updated_at
```

A participant may be:

- a chapter member
- an external attendee

For an external person:

```text
member_id = NULL
external_name = "..."
external_email = "..."
```

For a member:

```text
member_id = actual member
```

Do not create fake member records for external attendees.

### Duplicate prevention

Use a uniqueness strategy that prevents duplicate member participation for the same event/role.

---

## 10.7 `event_speakers`

```text
event_speakers
--------------------------------
id
event_id
member_id NULL

name
designation
photo_url NULL
profile_url NULL
display_order

created_at
```

If the speaker is a member, link `member_id`.

If external, keep the external speaker fields.

---

# 11. Projects / Innovations

## 11.1 `project_categories`

```text
project_categories
--------------------------------
id
name
slug UNIQUE
description
display_order
is_active
```

Seed:

```text
AI/ML
Software
Hardware
Research
Startup
Other
```

---

## 11.2 `projects`

```text
projects
--------------------------------
id
slug UNIQUE
title
summary
description

category_id
cover_url

repo_url NULL
demo_url NULL

started_at NULL
completed_at NULL

status

created_by
reviewed_by NULL
published_at NULL

created_at
updated_at
deleted_at
```

---

## 11.3 `project_members`

Projects support both members and external collaborators.

```text
project_members
--------------------------------
project_id
member_id NULL
external_name NULL
role
display_order
```

Rules:

```text
member_id != NULL
OR
external_name != NULL
```

Do not create fake members for external collaborators.

---

# 12. Achievements

## 12.1 `achievement_categories`

```text
achievement_categories
--------------------------------
id
name
slug UNIQUE
description
display_order
is_active
```

Seed:

```text
COMPETITION_WIN
HACKATHON
AWARD
SCHOLARSHIP
PUBLICATION
CERTIFICATION
CONFERENCE_PRESENTATION
RESEARCH
CHAPTER_AWARD
EXTERNAL_ACHIEVEMENT
```

---

## 12.2 `achievements`

```text
achievements
--------------------------------
id
member_id

title
category_id

rank NULL
issuer NULL
event_name NULL

achievement_date
description

proof_url NULL
proof_file_path NULL

project_id NULL

status
reviewed_by NULL
published_at NULL

created_at
updated_at
deleted_at
```

Proof files are private.

Never expose the private storage path through public API responses.

---

# 13. Publications

## 13.1 `publication_types`

```text
publication_types
--------------------------------
id
code UNIQUE
name
description
```

Seed:

```text
JOURNAL
CONFERENCE
PREPRINT
POSTER
THESIS
```

---

## 13.2 `publications`

```text
publications
--------------------------------
id
slug UNIQUE
title
abstract

publication_type_id

venue
publication_date

doi NULL
xplore_url NULL
pdf_url NULL

pdf_redistribution_confirmed

status

created_by
reviewed_by NULL
published_at NULL

created_at
updated_at
deleted_at
```

Never expose a publication PDF unless redistribution permission has been confirmed.

---

## 13.3 `publication_authors`

```text
publication_authors
--------------------------------
publication_id
member_id NULL
external_name NULL
author_order
```

For a chapter member:

```text
member_id = member
```

For external co-author:

```text
member_id = NULL
external_name = "..."
```

`author_order` is mandatory because author order matters.

---

# 14. Posts

## 14.1 `post_types`

```text
post_types
--------------------------------
id
code UNIQUE
name
description
```

Examples:

```text
TECHNICAL_ARTICLE
PROJECT_SHOWCASE
RESEARCH_SUMMARY
EXPERIENCE
EVENT_REPORT
OPINION
LEARNING_RESOURCE
```

---

## 14.2 `posts`

```text
posts
--------------------------------
id
slug UNIQUE
author_member_id

type_id

title
excerpt
cover_url
body

reading_time_minutes

status

submitted_at
reviewed_at
published_at

reviewed_by NULL

created_at
updated_at
deleted_at
```

The body is rich text.

Use a controlled rich-text representation suitable for your editor.

Do not expose unsanitized HTML.

---

## 14.3 `tags`

```text
tags
--------------------------------
id
name
slug UNIQUE
created_at
```

---

## 14.4 `post_tags`

```text
post_tags
--------------------------------
post_id
tag_id

PRIMARY KEY(post_id, tag_id)
```

Do NOT store tags as comma-separated text.

Bad:

```text
tags = "AI, ML, Research"
```

Good:

```text
posts
   |
post_tags
   |
tags
```

---

# 15. Unified Submission / Moderation System

Posts, projects, achievements, and publications share one workflow.

Workflow:

```text
DRAFT
  ↓
SUBMITTED
  ↓
UNDER_REVIEW
  ↓
APPROVED / PUBLISHED

or

RETURNED
  ↓
Member edits
  ↓
RESUBMITTED

or

REJECTED

or

ARCHIVED
```

---

## 15.1 `submissions`

```text
submissions
--------------------------------
id

content_type
content_id

author_member_id

status

submitted_at
reviewed_at
reviewed_by NULL

feedback NULL

created_at
updated_at
```

`content_type` identifies the content domain:

```text
POST
PROJECT
ACHIEVEMENT
PUBLICATION
```

`content_id` points to the corresponding entity.

### Important

Because PostgreSQL cannot enforce a normal FK across multiple possible tables, application/service-layer validation is required for the polymorphic content reference.

Do not pretend this is a normal FK.

---

## 15.2 `submission_status_history`

```text
submission_status_history
--------------------------------
id
submission_id

from_status NULL
to_status

changed_by
feedback NULL

changed_at
```

This gives a complete moderation trail.

---

# 16. Certificates

## `certificates`

```text
certificates
--------------------------------
id
member_id

title
certificate_type

event_id NULL
achievement_id NULL
participation_id NULL

issue_date

issued_by

verification_code UNIQUE

file_path NULL

revoked_at NULL
revoked_by NULL
revocation_reason NULL

created_at
updated_at
```

Certificate types:

```text
PARTICIPATION
VOLUNTEER
ORGANIZER
SPEAKER
ACHIEVEMENT
APPRECIATION
```

### Certificate source

A certificate is issued against:

```text
event_participation
```

or:

```text
achievement
```

The relationship should be validated so a certificate does not reference unrelated or invalid source records.

### Verification code

Use a cryptographically secure random, non-sequential value.

Never use:

```text
CERT-000001
CERT-000002
```

Good examples are random opaque tokens.

The verification code is public-facing, so it must not expose database IDs.

### Revocation

Never delete a revoked certificate.

Verification must return:

```text
VALID
```

or:

```text
REVOKED
```

rather than 404.

---

# 17. Gallery / Media

## 17.1 `gallery_albums`

```text
gallery_albums
--------------------------------
id
slug UNIQUE
title
description NULL
year

event_id NULL
cover_url NULL

created_by

created_at
updated_at
deleted_at
```

An album can be:

- event-linked
- standalone

---

## 17.2 `gallery_items`

```text
gallery_items
--------------------------------
id
album_id

type
storage_path
url NULL

caption NULL
alt_text NULL

taken_at NULL
display_order

created_at
updated_at
deleted_at
```

Types:

```text
IMAGE
VIDEO_EMBED
```

For self-hosted images, use storage paths.

Videos should normally be external embeds unless the project requirements later change.

---

# 18. Partners

Sponsors and collaborations use the same underlying partner entity.

## `partners`

```text
partners
--------------------------------
id
name
slug UNIQUE

type

logo_url
website NULL
description NULL

start_date NULL
end_date NULL

is_featured
display_order

created_at
updated_at
deleted_at
```

Types:

```text
INDUSTRY
COMMUNITY
ACADEMIC
TECHNICAL
```

Past partners are archived, not destroyed.

---

# 19. Partner ↔ Event relationship

The PRD says partners may be associated with events.

Therefore add:

```text
event_partners
--------------------------------
event_id
partner_id

PRIMARY KEY(event_id, partner_id)
```

This table is important and should not be omitted.

---

# 20. History

## `history_entries`

```text
history_entries
--------------------------------
id
title
description

entry_date
year

image_url NULL

event_id NULL
project_id NULL
achievement_id NULL
publication_id NULL
session_id NULL

display_order

created_at
updated_at
deleted_at
```

History entries should reference existing records instead of duplicating their content.

Use nullable foreign keys for known entity relationships.

Avoid uncontrolled polymorphic:

```text
related_type
related_id
```

when a real FK can be used.

---

# 21. Join Applications

## `applications`

```text
applications
--------------------------------
id

name
email
phone
student_id
department
batch

ieee_membership_number NULL

interests
motivation

status

reviewed_by NULL
reviewed_at NULL
review_notes NULL

created_member_id NULL

created_at
updated_at
```

Statuses:

```text
SUBMITTED
UNDER_REVIEW
APPROVED
REJECTED
```

Rejected applications remain in the database.

When approved:

```text
application
    ↓
member
    ↓
auth account/invitation
```

---

# 22. Notifications

## `notifications`

```text
notifications
--------------------------------
id
user_id

type
title
message

data JSONB

read_at NULL
created_at
```

`data` can contain contextual IDs.

Example:

```json
{
  "submission_id": "...",
  "content_type": "POST"
}
```

JSONB is appropriate here because notification payloads may vary by notification type.

---

# 23. Audit Logs

## `audit_logs`

```text
audit_logs
--------------------------------
id

actor_user_id

action
entity_type
entity_id

before_data JSONB NULL
after_data JSONB NULL

ip_address NULL
user_agent NULL

created_at
```

Examples:

```text
CREATE
UPDATE
DELETE
RESTORE
PUBLISH
UNPUBLISH
APPROVE
REJECT
RETURN
ISSUE
REVOKE
ROLE_ASSIGN
ROLE_REMOVE
STATUS_CHANGE
```

At minimum, audit:

- destructive actions
- publish/unpublish actions
- moderation decisions
- role changes
- member status changes
- position changes
- certificate issuance/revocation
- settings changes

---

# 24. Site Settings

## `site_settings`

```text
site_settings
--------------------------------
id
key UNIQUE
value JSONB

description NULL
updated_by NULL

created_at
updated_at
```

Examples:

```text
chapter.name
chapter.email
chapter.address
chapter.university
chapter.location
chapter.facebook
chapter.linkedin
chapter.github
footer.copyright
homepage.statistics
homepage.sections
```

Use JSONB for flexible configuration.

Do not store secrets in public site settings.

---

# 25. Navigation

## `navigation_items`

```text
navigation_items
--------------------------------
id

location
label
url

parent_id NULL

display_order
is_visible

required_permission NULL

created_at
updated_at
```

Locations:

```text
HEADER
FOOTER
```

Navigation should be data-driven but authorization must still be enforced server-side.

---

# 26. Contact Messages

## `contact_messages`

```text
contact_messages
--------------------------------
id

name
email
subject
message

status

handled_by NULL
handled_at NULL

created_at
```

Statuses:

```text
NEW
READ
RESOLVED
SPAM
```

Contact submissions must have spam/rate-limit protection at the application layer.

---

# 27. Database Views

Use views for frequently derived concepts.

## 27.1 `current_positions`

Expose:

```text
member
position
position_title
team
session
```

where:

```text
end_date IS NULL
```

---

## 27.2 `event_status_view`

Calculate:

```text
UPCOMING
ONGOING
COMPLETED
```

from:

```text
start_at
end_at
```

Do not maintain a manually editable status field for this lifecycle.

---

## 27.3 `member_journey_view`

Conceptually combine:

```text
positions
event_participations
achievements
certificates
```

into a normalized chronological event structure.

Example conceptual output:

```text
date
type
title
description
reference_id
```

The UI then builds the vertical journey from the view/query.

Do not store the rendered journey.

---

# 28. Homepage Statistics

The homepage must not execute many independent COUNT queries on every request.

The PRD requires statistics to be cached/recomputed. Use a cached/materialized strategy.

Potential statistics:

```text
member_count
event_count
workshop_count
project_count
publication_count
achievement_count
```

A materialized view or cached server-side result can be used.

Refresh on write or on a schedule.

Do not prematurely introduce a separate analytics infrastructure.

---

# 29. Indexing Strategy

Indexes should support actual query patterns.

## Members

```text
UNIQUE(slug)
INDEX(membership_status_id)
INDEX(department)
INDEX(batch)
INDEX(created_at)
```

For name search, consider PostgreSQL `pg_trgm`:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX members_full_name_trgm_idx
ON members USING gin (full_name gin_trgm_ops);
```

At ~1,000 members, PostgreSQL search is sufficient. Do not introduce Elasticsearch/OpenSearch.

---

## Events

```text
UNIQUE(slug)
INDEX(category_id)
INDEX(start_at)
INDEX(end_at)
INDEX(created_at)
```

---

## Registrations

```text
INDEX(event_id)
INDEX(member_id)
INDEX(email)
INDEX(created_at)
```

---

## Participations

```text
INDEX(event_id)
INDEX(member_id)
INDEX(role_id)
```

Add duplicate-prevention uniqueness.

---

## Positions

```text
INDEX(member_id)
INDEX(session_id)
INDEX(position_title_id)
INDEX(team_id)
INDEX(start_date)
INDEX(end_date)
```

---

## Posts

```text
UNIQUE(slug)
INDEX(author_member_id)
INDEX(type_id)
INDEX(status)
INDEX(published_at)
```

---

## Projects

```text
UNIQUE(slug)
INDEX(category_id)
INDEX(status)
INDEX(published_at)
```

---

## Achievements

```text
INDEX(member_id)
INDEX(category_id)
INDEX(achievement_date)
INDEX(status)
```

---

## Publications

```text
UNIQUE(slug)
INDEX(publication_type_id)
INDEX(publication_date)
INDEX(status)
```

---

## Certificates

```text
UNIQUE(verification_code)
INDEX(member_id)
INDEX(event_id)
INDEX(participation_id)
INDEX(achievement_id)
```

---

## Applications

```text
INDEX(status)
INDEX(created_at)
```

---

## Notifications

```text
INDEX(user_id)
INDEX(read_at)
INDEX(created_at)
```

---

## Audit logs

```text
INDEX(actor_user_id)
INDEX(entity_type, entity_id)
INDEX(created_at)
```

---

# 30. Soft Delete Rules

Use `deleted_at` for historical/content entities where restore is useful.

Typical candidates:

```text
members
events
posts
projects
achievements
publications
gallery_albums
gallery_items
partners
history_entries
```

When `deleted_at IS NOT NULL`:

- exclude from normal public queries
- exclude from normal admin lists unless explicitly requested
- preserve relationships/history
- allow restore according to admin policy

Do not add soft deletion mechanically to every junction/lookup table.

---

# 31. Foreign Key Delete Rules

Be conservative.

Historical records should not disappear because a parent is deleted.

Preferred patterns:

### Required parent relationship

Use:

```text
ON DELETE RESTRICT
```

when deleting the parent should be blocked.

### Optional relationship

Use:

```text
ON DELETE SET NULL
```

when the relationship is optional and the record can survive.

### Junction table

Use:

```text
ON DELETE CASCADE
```

only for pure relationship rows where deleting the parent genuinely should remove the relationship.

Example:

```text
post_tags
project_members
publication_authors
event_partners
role_permissions
user_roles
```

Do not use `CASCADE` casually on historical entities.

---

# 32. Data Integrity Rules

Enforce important rules at DB level where possible.

Examples:

```text
events.end_at >= events.start_at

sessions.end_date >= sessions.start_date

positions.end_date >= positions.start_date

gallery_items.display_order >= 0

author_order > 0

capacity >= 0
```

Use CHECK constraints where practical.

Use unique constraints for natural uniqueness.

Do not rely only on frontend validation.

---

# 33. Privacy Architecture

Private fields must not be selected into public API responses.

Potentially private:

```text
email
phone
student_id
internal member ID
administrative notes
application data
certificate internal notes
achievement proof files
```

Members can control visibility of:

```text
email
phone
batch
bio
social_links
individual achievements
```

Do not do:

```text
SELECT *
FROM members;
```

for public profile endpoints.

Use explicit select lists or database views.

Bad:

```text
SELECT *
```

Good:

```text
SELECT
    id,
    slug,
    full_name,
    preferred_name,
    photo_url,
    bio,
    ...
```

Only include fields that the endpoint is allowed to expose.

---

# 34. Public vs Private Storage

Recommended buckets:

```text
public-assets
member-photos
event-media
gallery
certificates
private-proofs
publication-files
```

The exact bucket arrangement can differ, but private documents must not be publicly accessible.

### Private

```text
achievement proof
internal administrative documents
application attachments if later added
```

### Member-only

```text
member certificates
```

### Public

```text
published event banners
published gallery images
published member photos where permitted
```

Use signed URLs for protected files.

---

# 35. Certificate Verification Security

Certificate verification is public.

The endpoint:

```text
/verify/[code]
```

must:

1. Validate the code format.
2. Rate-limit requests.
3. Look up the certificate using the unique verification code.
4. Return a controlled public DTO.
5. Never return internal issuance notes.
6. Return `VALID` or `REVOKED`.
7. Never expose database IDs unnecessarily.

Public response should contain only:

```text
recipient name
certificate title
certificate type
event
issue date
issuing chapter
verification status
revocation date if revoked
```

---

# 36. Public API DTO Principle

Never return database rows blindly.

Use separate DTOs for:

```text
PublicMember
PrivateMember
AdminMember
PublicCertificate
AdminCertificate
PublicEvent
AdminEvent
```

The database schema is not the API contract.

This is especially important for privacy.

---

# 37. Admin Workflow Principle

Admin UI can be friendly and simplified.

Database remains normalized.

Never denormalize the core DB merely because the admin dashboard wants a convenient shape.

Example:

Admin wants:

```text
Event
 ├─ Registrations
 ├─ Participants
 ├─ Speakers
 └─ Gallery
```

That does NOT mean `events` should contain JSON arrays for all of these.

Keep:

```text
events
event_registrations
event_participations
event_speakers
gallery_albums
```

and join them.

---

# 38. Performance Strategy for ~1,000 Users

Do NOT introduce:

```text
database sharding
microservices
Elasticsearch
Kafka
complex event sourcing
multiple database clusters
```

unless future scale actually requires them.

PostgreSQL can comfortably handle this application.

The main large data source is media.

Therefore:

```text
PostgreSQL
    → structured data

Object Storage/CDN
    → images/PDFs/media
```

Keep these concerns separate.

---

# 39. Search Strategy

For:

```text
members
events
posts
projects
publications
```

start with PostgreSQL.

Use:

- B-tree indexes for exact/filter queries
- `pg_trgm` for name/title fuzzy search where needed
- full-text search if content search becomes necessary

Do not add an external search engine in v1 without a demonstrated requirement.

---

# 40. Pagination

Every admin list should support:

```text
search
filter
sort
pagination
```

Public archive lists should also be paginated.

For large/chronological lists, prefer cursor/keyset pagination.

Examples:

```text
/events
/members
/posts
/projects
/publications
/achievements
/gallery
```

Do not load entire tables into the browser.

---

# 41. Transactions

Use database transactions for multi-step operations that must be atomic.

Examples:

### Approving an application

```text
approve application
    +
create member
    +
create account/invitation state
    +
audit
```

### Bulk certificate issuance

```text
select participants
    +
create certificates
    +
audit
```

If part of a required atomic operation fails, rollback.

---

# 42. Bulk Operations

The admin dashboard needs bulk operations.

Examples:

```text
bulk participation confirmation
bulk certificate issuance
bulk gallery upload metadata
bulk member status update
```

Do not issue hundreds of independent client-side requests when a transactional server operation is appropriate.

---

# 43. Audit + Soft Delete Relationship

When an entity is deleted:

```text
UPDATE entity
SET deleted_at = NOW()
```

and write:

```text
audit_logs
action = DELETE
before_data = previous record
after_data = deleted state
```

When restored:

```text
action = RESTORE
```

Never silently delete without an audit entry for audited entities.

---

# 44. Data Migration / Committee Handover

The schema must support committee turnover without migration.

A new committee should:

1. Receive/administer existing accounts.
2. Create a new `session`.
3. Create/update positions.
4. End old positions with `end_date`.
5. Assign new positions.
6. Continue creating events/content.
7. Preserve all previous records.

Do NOT create separate databases for each committee/session.

Bad:

```text
database_2025
database_2026
database_2027
```

Correct:

```text
one database

sessions:
2025-2026
2026-2027
2027-2028
```

---

# 45. Historical URL Preservation

Public slugs should remain stable.

Member:

```text
/members/john-doe
```

Event:

```text
/events/ai-ml-workshop-2026
```

Do not automatically regenerate a slug because a title/name changes.

Old records should remain accessible unless there is a deliberate redirect strategy.

---

# 46. Event Slug Rule

Event slugs should include the year for recurring events.

Good:

```text
ai-ml-workshop-2026
ai-ml-workshop-2027
```

Bad:

```text
ai-ml-workshop
```

This prevents collisions across years.

---

# 47. Recommended Database-Level Views / Functions

Useful database abstractions:

```text
current_positions
event_status
member_journey
current_committee
public_members
public_certificates
homepage_statistics
```

Potential functions:

```text
get_member_journey(member_id)
get_event_status(event_id)
refresh_homepage_statistics()
```

Do not overuse stored procedures. Keep business logic in the application unless DB-side logic provides a clear integrity/performance benefit.

---

# 48. Prisma Guidance

If Prisma is used:

- keep model names singular in Prisma if that matches project conventions
- map them to plural snake_case PostgreSQL tables if desired
- use explicit relation names for multiple relationships between the same tables
- represent nullable relationships carefully
- use migrations as the source of schema evolution
- never manually alter production DB without recording the migration

Example convention:

```prisma
model Member {
  id        String   @id @default(uuid())
  slug      String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("members")
}
```

Use PostgreSQL-native features when appropriate.

Do not force every PostgreSQL feature into a simplistic ORM abstraction.

---

# 49. Supabase RLS Guidance

RLS should complement application authorization.

Typical member rule:

```text
member can read/update own permitted profile
```

but cannot update:

```text
positions
membership_status
certificates
participations
official achievements
```

Department Admin:

```text
can manage only domains assigned in admin_scopes
```

Executive Admin:

```text
chapter content/member management
```

Super Admin:

```text
full management
```

Never trust:

```text
role
```

sent by the frontend.

The authenticated identity must be resolved server-side.

---

# 50. Important Missing/Added Tables Compared to Initial PRD

The original PRD's proposed entities are the baseline, but the production design adds a few important supporting structures.

Important additions:

```text
roles
permissions
user_roles
role_permissions
content_domains
admin_scopes

membership_statuses
membership_status_history

sessions

event_registration_fields
event_participation_roles
event_partners

project_categories
achievement_categories
publication_types
post_types
tags
post_tags

submissions
submission_status_history

navigation_items

```

These are not unnecessary complexity. They support the explicit requirements for:

- dynamic permissions
- scoped Department Admins
- historical membership transitions
- first-class sessions
- configurable registration fields
- shared moderation
- filterable content
- admin-managed navigation

---

# 51. Recommended ERD

High-level relationship model:

```text
                         ┌──────────────┐
                         │ auth.users   │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │   members    │
                         └──────┬───────┘
                                │
        ┌───────────────┬───────┼───────────────┬──────────────┐
        │               │       │               │              │
        ▼               ▼       ▼               ▼              ▼
    positions      links   participations   achievements   certificates
        │                       │               │
        │                       ▼               ▼
        ▼                    events          projects
     sessions                   │               │
        │             ┌─────────┼───────┐       │
        ▼             ▼         ▼       ▼       ▼
 position_titles  registrations speakers gallery project_members
        │
        ▼
      teams
```

Content:

```text
members
   │
   ├── posts ─── post_tags ─── tags
   │
   ├── projects ─── project_members
   │
   ├── achievements
   │
   └── publications ─── publication_authors
```

Moderation:

```text
member
  │
  ▼
submission
  │
  ├── status
  └── status_history
       │
       ▼
  content entity
```

Administration:

```text
users
  │
  ├── user_roles ─── roles ─── role_permissions ─── permissions
  │
  └── admin_scopes ─── content_domains
```

---

# 52. Suggested Implementation Order

Do not create everything in one migration without testing.

Implement in phases matching the product build plan.

## Migration 001 — Foundations

```text
members
membership_statuses
membership_status_history

roles
permissions
user_roles
role_permissions
content_domains
admin_scopes
```

## Migration 002 — Organization

```text
sessions
teams
position_titles
positions
member_links
```

## Migration 003 — Events

```text
event_categories
events
event_registration_fields
event_registrations
event_participation_roles
event_participations
event_speakers
event_partners
```

## Migration 004 — Contributions

```text
post_types
posts
tags
post_tags

project_categories
projects
project_members

achievement_categories
achievements

publication_types
publications
publication_authors
```

## Migration 005 — Moderation

```text
submissions
submission_status_history
```

## Migration 006 — Certificates

```text
certificates
```

## Migration 007 — Archive

```text
gallery_albums
gallery_items
partners
history_entries
```

## Migration 008 — Platform

```text
applications
notifications
audit_logs
site_settings
navigation_items
contact_messages
```

Then:

```text
views
indexes
RLS
functions
triggers
seed data
```

---

# 53. Seed Data

The first seed migration should create the base:

### Roles

```text
MEMBER
DEPARTMENT_ADMIN
EXECUTIVE_ADMIN
SUPER_ADMIN
```

### Membership statuses

```text
APPLICANT
ACTIVE
ALUMNI
INACTIVE
```

### Event categories

```text
WORKSHOP
SEMINAR
COMPETITION
HACKATHON
CONFERENCE
WEBINAR
TRAINING
COMMUNITY_ACTIVITY
SOCIAL_EVENT
ORIENTATION
RESEARCH_EVENT
```

### Participation roles

```text
PARTICIPANT
VOLUNTEER
ORGANIZER
SPEAKER
MENTOR
JUDGE
```

### Project categories

```text
AI_ML
SOFTWARE
HARDWARE
RESEARCH
STARTUP
OTHER
```

### Publication types

```text
JOURNAL
CONFERENCE
PREPRINT
POSTER
THESIS
```

### Submission statuses

```text
DRAFT
SUBMITTED
UNDER_REVIEW
APPROVED
RETURNED
REJECTED
ARCHIVED
```

---

# 54. Things Claude Code Must NOT Do

When modifying the schema, DO NOT:

1. Add `position` directly to `members`.
2. Merge role and membership status.
3. Treat event registration as participation.
4. Store gallery binaries in PostgreSQL.
5. Store passwords if Supabase Auth is used.
6. Hardcode admin permissions by email.
7. Hardcode Department Admin scope.
8. Store tags as comma-separated strings.
9. Store journey as JSON/text.
10. Make event lifecycle status manually editable when it can be derived.
11. Use sequential certificate verification codes.
12. expose private fields in public DTOs.
13. hard-delete historical data through normal admin actions.
14. use `SELECT *` in public APIs.
15. create one database per committee session.
16. create fake member records for external collaborators.
17. duplicate the moderation system for every content type.
18. add a new table when a normalized existing relationship is sufficient.
19. denormalize core relationships merely to make a frontend component easier.
20. bypass migrations.

---

# 55. When Changing Existing Schema

Before modifying a table:

1. Search the repository for every reference to the table/model.
2. Identify API/server-action usage.
3. Identify RLS policies.
4. Identify foreign keys.
5. Identify indexes.
6. Identify seed data.
7. Identify UI assumptions.
8. Create a migration.
9. Update Prisma/schema definitions.
10. Update affected queries/types.
11. Test existing workflows.
12. Do not silently rename/remove production columns.

Schema changes must be backward-conscious.

---

# 56. Data Ownership Rules

### Member-controlled

Members can edit:

```text
preferred_name
photo
bio
social links
skills/interests if implemented
privacy settings
```

### Admin-controlled

Members cannot directly edit:

```text
membership status
positions
official achievements
certificates
event participation
IEEE membership status
administrative records
```

Corrections should go through a request/workflow rather than direct mutation.

---

# 57. Content Publishing Rules

Public content should normally require:

```text
approved/published
```

Do not expose:

```text
DRAFT
SUBMITTED
UNDER_REVIEW
RETURNED
REJECTED
```

content through public endpoints.

Archived content should only appear where archive/history functionality explicitly requires it.

---

# 58. Consistency Principle

If a fact can be derived from authoritative data, do not duplicate it unnecessarily.

Examples:

### Current position

Derived:

```text
positions.end_date IS NULL
```

Not:

```text
members.current_position
```

### Event status

Derived:

```text
start_at / end_at
```

Not manually maintained.

### Member journey

Derived:

```text
positions
participations
achievements
certificates
```

Not stored.

### Current committee

Derived:

```text
session + active positions
```

Not copied into a separate `current_team_members` table.

This prevents stale data.

---

# 59. Performance Philosophy

Optimize based on real query patterns.

At 1,000 users:

```text
PostgreSQL = enough
```

Focus on:

```text
correct indexes
proper pagination
efficient joins
server-side filtering
cached homepage statistics
object storage for media
RLS correctness
```

Do not optimize prematurely.

---

# 60. Final Architecture Principle

The system should behave like:

```text
                   CHAPTER
                      │
             ┌────────┴────────┐
             │                 │
          PEOPLE             CONTENT
             │                 │
        ┌────┼────┐       ┌────┼────┐
        │    │    │       │    │    │
      Roles Positions Events Posts Projects
        │    │    │       │    │    │
        │ Sessions│       │    │    │
        │         │       │    │    │
        └────┬────┘       └────┼────┘
             │                 │
             └────────┬────────┘
                      │
                 ARCHIVE / HISTORY
                      │
                      ▼
              LONG-TERM RECORD
```

The database should make the following question easy to answer:

> "What did this member contribute to the chapter during their years here?"

The answer should be assembled from actual records:

```text
Member
 ├── Membership history
 ├── Positions
 ├── Sessions
 ├── Event participation
 ├── Achievements
 ├── Certificates
 ├── Projects
 ├── Publications
 └── Posts
```

That is the core architectural purpose of this schema.

---

# 61. Definition of Done for Database Phase

The database phase is complete only when:

- [ ] Fresh database can be created from migrations.
- [ ] All FK relationships work.
- [ ] Seed data works.
- [ ] Super Admin can be created.
- [ ] RBAC works server-side.
- [ ] Department Admin scopes work from DB data.
- [ ] Member cannot edit protected fields.
- [ ] Positions are historical.
- [ ] Sessions work.
- [ ] Event registration and participation are separate.
- [ ] Event status is derived from dates.
- [ ] Shared submission workflow works.
- [ ] Certificate verification codes are non-guessable.
- [ ] Certificate revocation works.
- [ ] Public certificate response exposes only allowed metadata.
- [ ] Soft delete/restore works for applicable entities.
- [ ] Audit logs record required administrative actions.
- [ ] Private storage is protected.
- [ ] Public/private member data is separated at API level.
- [ ] Required indexes exist.
- [ ] Pagination works.
- [ ] Homepage statistics do not run excessive count queries per request.
- [ ] RLS policies are tested.
- [ ] Migrations run successfully from zero.
- [ ] Existing migrations can reproduce the same schema.
- [ ] No production table contains unnecessary duplicated historical data.

---

# 62. Claude Code Instruction

When working on this project, treat this document as the **database architecture contract**.

If an implementation request conflicts with this schema:

1. Prefer the relational/historical design.
2. Check whether the requested change violates data integrity.
3. Do not silently introduce denormalization.
4. Do not remove historical relationships.
5. Do not weaken authorization to make implementation easier.
6. Do not expose private fields.
7. Do not bypass migrations.
8. If a requirement genuinely requires changing this architecture, explain the tradeoff before making a destructive schema change.

The goal is not merely to make the current UI work.

The goal is to build a database that can preserve the chapter's history for many years while remaining maintainable by future student committees.
