# Product Requirements Document
## IEEE Computer Society Leading University Student Branch Chapter — Official Website

| Field | Value |
|---|---|
| Product name | IEEE CS LU SB Chapter Website |
| Document version | v0.1 (draft for review) |
| Status | Requirements defined, stack and scope decisions pending |
| Owner | Chapter Web/Technical Team |
| Last updated | 2026-09-14 |

---

## 0. How to read this document

- Every requirement has an ID (e.g. `EVT-04`). Use these IDs in tickets, commits, and test cases.
- **MUST** = required for the release it sits in. **SHOULD** = expected but can slip. **MAY** = optional / future.
- Section 13 lists open questions. Nothing in this document is final until those are answered.
- Section 12 is the build plan. Do not start a phase until the previous phase passes its verification gate.

---

## 1. Product summary

### 1.1 One-line description
A permanent digital home and historical record of the IEEE Computer Society Leading University Student Branch Chapter, its activities, and its members.

### 1.2 Problem
Student club websites are usually static brochures. Activity records live in Facebook posts, Google Drive folders, and the memory of whoever was in charge that year. When an executive committee graduates, the chapter's history effectively disappears, and members have nothing durable to show for years of work.

### 1.3 Product vision
The website is three things at once:

1. **A public face** — what the chapter is, what it does, how to join.
2. **An archive** — events, achievements, publications, photos, and leadership history that survive committee turnover.
3. **A member system** — every member has a profile that automatically accumulates their real contribution record: events attended, roles held, certificates earned, work produced.

### 1.4 What success looks like
- A visitor can understand the chapter and find any past event in under 60 seconds.
- A member's profile is generated from real records, not hand-written text.
- A committee handover requires no data migration. The new committee logs in and continues.
- Five years from now, the 2026 committee's work is still browsable.

---

## 2. Goals and non-goals

### 2.1 Goals
| ID | Goal |
|---|---|
| G-01 | Public information platform for the chapter |
| G-02 | Complete event archive with participation records |
| G-03 | Member directory and individual profile/portfolio system |
| G-04 | Achievement, certificate, and publication archive |
| G-05 | Visual chapter history timeline |
| G-06 | Member-contributed content with admin moderation |
| G-07 | Role-based admin dashboard for non-technical committee members |
| G-08 | Alumni archive so past members and their contributions remain visible |
| G-09 | Public certificate verification |

### 2.2 Non-goals (v1)
| ID | Non-goal | Note |
|---|---|---|
| NG-01 | Payment collection for event registration | Revisit if paid events happen |
| NG-02 | Real-time chat / forum / DMs | Community feel comes from posts, not chat |
| NG-03 | Mobile apps | Responsive web only |
| NG-04 | Multi-chapter / multi-tenant support | Built for one chapter |
| NG-05 | Automated IEEE membership API sync | Membership status is admin-entered |
| NG-06 | Email newsletter engine | Use an external tool, link from site |

---

## 3. Users and roles

### 3.1 Two separate concepts
This is a core design decision. Do not merge them.

- **Role** = what you can do in the system (permissions).
- **Membership status** = your relationship with the chapter (Applicant, Active, Alumni, Inactive).

A person can be an Alumni with an Admin role, or an Active member with no special role. Alumni is **not** a permission level.

### 3.2 Roles

| ID | Role | Description |
|---|---|---|
| R-01 | Visitor | Not logged in. Public content only. |
| R-02 | Member | Logged-in chapter member. Own profile + submissions. |
| R-03 | Department Admin | Manages one or more content domains (e.g. Media manages Gallery). Scoped by domain. |
| R-04 | Executive Admin | Manages chapter content and members. Cannot manage admins or site settings. |
| R-05 | Super Admin | Everything, including roles, permissions, and site settings. |

### 3.3 Membership statuses

| ID | Status | Meaning |
|---|---|---|
| S-01 | Applicant | Submitted join form, not yet approved |
| S-02 | Active | Current chapter member |
| S-03 | Alumni | Graduated or left; profile archived and still public |
| S-04 | Inactive | Temporarily not participating; profile hidden from directory, kept in DB |

### 3.4 Permission matrix

Legend: **F** = full CRUD, **O** = own records only, **S** = submit for approval, **R** = read, **A** = approve/publish, **–** = no access.

| Capability | Visitor | Member | Dept. Admin | Exec Admin | Super Admin |
|---|---|---|---|---|---|
| View published public content | R | R | R | R | R |
| Own profile (edit permitted fields) | – | O | O | O | O |
| Own privacy settings | – | O | O | O | O |
| Posts | R | S + O | F + A *(if scoped)* | F + A | F + A |
| Innovations/Projects | R | S + O | F + A *(if scoped)* | F + A | F + A |
| Achievements | R | S + O | F + A *(if scoped)* | F + A | F + A |
| Events | R | R | F *(if scoped)* | F | F |
| Event participation records | – | R (own) | F *(if scoped)* | F | F |
| Gallery | R | R | F *(if scoped)* | F | F |
| Publications | R | S + O | F + A *(if scoped)* | F + A | F + A |
| Certificates (issue/revoke) | – | R (own) | – | F | F |
| Members (create/edit/status) | – | – | – | F | F |
| Positions (assign/end) | – | – | – | F | F |
| Sponsors & Collaborations | R | R | F *(if scoped)* | F | F |
| Chapter history entries | R | R | F *(if scoped)* | F | F |
| Join applications | – | – | – | F | F |
| Admin accounts & role assignment | – | – | – | – | F |
| Website settings | – | – | – | – | F |
| Audit log | – | – | – | R | F |

`ROLE-01` (MUST) Permissions are checked server-side on every request. Hiding a UI button is not access control.
`ROLE-02` (MUST) Department Admin scope is stored as a list of content domains per user, not hardcoded per person.
`ROLE-03` (MUST) Every destructive or publishing action writes to an audit log: who, what, when, before/after.

---

## 4. Information architecture

### 4.1 Main navigation

```
HOME   ABOUT   EVENTS   MEMBERS   GALLERY   MORE ▾        [Join Us] [Login]
```

**More** contains: Achievements · History · Innovations · Publications · Sponsors · Collaborations · Contact · Join Us

After login the right side becomes `[Profile ▾]` with: My Profile · My Certificates · My Submissions · Dashboard *(if admin)* · Logout

Mobile: logo + hamburger, full-screen drawer, same structure.

### 4.2 URL map

| Path | Page | Access |
|---|---|---|
| `/` | Home | Public |
| `/about` | About the chapter | Public |
| `/events` | Events list + filters | Public |
| `/events/[slug]` | Event detail | Public |
| `/events/[slug]/register` | Event registration | Public |
| `/events/[slug]/gallery` | Event gallery | Public |
| `/members` | Member directory | Public |
| `/members/[slug]` | Member public profile | Public |
| `/team` | Current executive committee | Public |
| `/team/[session]` | Past committee (e.g. `2025-2026`) | Public |
| `/alumni` | Alumni archive | Public |
| `/gallery` | Gallery by year/album | Public |
| `/achievements` | Achievements showcase | Public |
| `/innovations` | Projects & innovations | Public |
| `/publications` | Publication archive | Public |
| `/posts` | Member posts feed | Public |
| `/posts/[slug]` | Post detail | Public |
| `/history` | Chapter timeline | Public |
| `/sponsors` | Sponsors | Public |
| `/collaborations` | Partners & collaborations | Public |
| `/contact` | Contact | Public |
| `/join` | Join application form | Public |
| `/verify` | Certificate verification lookup | Public |
| `/verify/[code]` | Verification result | Public |
| `/login` · `/logout` | Auth | Public |
| `/me` | Own profile editor | Member |
| `/me/submissions` | Own submissions + status | Member |
| `/me/certificates` | Own certificates | Member |
| `/dashboard/**` | Admin dashboard | Admin roles |

`IA-01` (MUST) Member slugs are unique, generated from the name, and immutable once issued. On collision append a numeric suffix. Changing a display name does not change the slug.
`IA-02` (MUST) Event slugs include the year (e.g. `ai-ml-workshop-2026`) to keep them unique across recurring events.
`IA-03` (SHOULD) Archived and alumni pages stay at their original URLs permanently. Never 404 an old record.

---

## 5. Public pages

### 5.1 Home

| ID | Requirement |
|---|---|
| HOME-01 | Hero: chapter name, headline ("Building the Next Generation of Computing Leaders"), supporting text, `[Explore Chapter]` `[Join Us]` |
| HOME-02 | Hero visuals use real chapter photography plus subtle technical/network motifs. No generic stock imagery. Animation must be subtle and respect `prefers-reduced-motion`. |
| HOME-03 | About preview with `Learn More →` |
| HOME-04 | Chapter impact statistics (Members, Events, Workshops, Projects, Publications) computed from the database, not hardcoded |
| HOME-05 | Recent/previous events as cards (banner, title, `View Event`) |
| HOME-06 | Our Team preview: current committee photo, name, position; each links to the member profile |
| HOME-07 | Recent member posts (approved only) |
| HOME-08 | Member innovations showcase, each linked to its author's profile |
| HOME-09 | Member achievements showcase (e.g. "🏆 1st Place — National Hackathon 2026 · John Doe · Digital Delta") |
| HOME-10 | Sponsors & collaborations, grouped as Industry / Community / Academic / Technical partners |
| HOME-11 | Closing call to action: `[Join Us]` `[Explore Events]` |
| HOME-12 | Footer per section 5.2 |
| HOME-13 | Statistics are cached and recomputed on a schedule or on write. The homepage must not run 10 count queries per visit. |
| HOME-14 | Every homepage section degrades gracefully when empty. A new chapter with no publications must not render a broken section. |

### 5.2 Footer

Four columns plus a base row.

- **Chapter**: About · History · Team · Members
- **Explore**: Events · Activities · Publications · Achievements · Gallery
- **Community**: Join Us · Sponsors · Collaborations
- **Contact**: Email · Social media · Location
- **Base row**: IEEE CS logo, chapter logo, copyright, Privacy Policy, Terms

`FOOT-01` (MUST) Footer links are managed from site settings, not hardcoded.
`FOOT-02` (MUST) IEEE and IEEE CS marks follow official IEEE branding rules. See open question Q-11.

### 5.3 About

| ID | Requirement |
|---|---|
| ABT-01 | Chapter introduction, IEEE CS affiliation, Leading University connection |
| ABT-02 | Mission, Vision, Objectives |
| ABT-03 | "What We Do" cards: Technical Workshops, Seminars, Competitions, Hackathons, Research, Publications, Community Activities, Networking, Professional Development |
| ABT-04 | Chapter impact statistics (same source as `HOME-04`) |
| ABT-05 | Content editable from the admin dashboard as rich text blocks |

### 5.4 Contact

| ID | Requirement |
|---|---|
| CON-01 | Official email, social links, location, university details |
| CON-02 | Contact form with spam protection; submissions stored in DB and emailed to the chapter address |
| CON-03 | Map embed of the university location (SHOULD) |

---

## 6. Events system

### 6.1 Events list

| ID | Requirement |
|---|---|
| EVT-01 | Status filter: All · Upcoming · Ongoing · Completed |
| EVT-02 | Additional filters: Year, Category |
| EVT-03 | Categories: Workshop, Seminar, Competition, Hackathon, Conference, Webinar, Training, Community Activity, Social Event, Orientation, Research Event. Categories are DB rows, editable by admins, not a hardcoded enum. |
| EVT-04 | Event card shows banner, title, date, status badge, `View Event` |
| EVT-05 | Status (Upcoming/Ongoing/Completed) is derived from start/end datetime, not manually set |
| EVT-06 | List is paginated or infinite-scrolled; filters reflect in the URL so results are shareable |

### 6.2 Event detail

| ID | Requirement |
|---|---|
| EVT-07 | Banner, title, full description, date, time, location/venue or online link |
| EVT-08 | Speakers/guests with photo, name, designation, optional profile link |
| EVT-09 | Event gallery (links to the album for that event) |
| EVT-10 | Post-event report (rich text, optional PDF attachment) |
| EVT-11 | Registration block showing status: Open · Closed · Full · Coming Soon |
| EVT-12 | Participant list shown only if the event is marked to show it; otherwise participant count only |

### 6.3 Registration

| ID | Requirement |
|---|---|
| EVT-13 | Registration fields: Name, Email, Student/Member ID, Department, Batch, Phone (each toggleable per event) |
| EVT-14 | Per-event capacity limit; status auto-flips to Full when reached |
| EVT-15 | Logged-in members get the form pre-filled from their profile |
| EVT-16 | Confirmation email on successful registration |
| EVT-17 | Admins can export registrations as CSV |
| EVT-18 | Registration ≠ participation. A registration only becomes a participation record when an admin confirms attendance. |

### 6.4 Participation records

This is the link between events and member profiles.

| ID | Requirement |
|---|---|
| EVT-19 | Participation roles: Participant, Volunteer, Organizer, Speaker, Mentor, Judge |
| EVT-20 | Admins attach members to an event with a role, individually or in bulk |
| EVT-21 | A confirmed participation record automatically appears on the member's profile and journey timeline |
| EVT-22 | A participation record is the eligibility source for issuing a certificate |
| EVT-23 | Non-member attendees can be recorded by name/email without a member account |

---

## 7. Member system

### 7.1 Directory

| ID | Requirement |
|---|---|
| MEM-01 | `/members` lists Active members with photo, name, current position, department/batch |
| MEM-02 | Search by name; filter by batch, department, position, team |
| MEM-03 | Inactive members are excluded; Alumni appear in `/alumni`, not the main directory |

### 7.2 Profile page (`/members/[slug]`)

| ID | Requirement |
|---|---|
| MEM-04 | Header: photo, full name, "IEEE CS Member", current position + session, university, department |
| MEM-05 | Basic info: preferred name, bio, batch, chapter joining date, IEEE membership status, IEEE CS membership status |
| MEM-06 | Social links: GitHub, LinkedIn, portfolio, others |
| MEM-07 | Member Journey: vertical timeline built from position records + participation records + achievements, ordered by date, ending at "Present" (or "Alumni since YYYY") |
| MEM-08 | Achievements section |
| MEM-09 | Certificates section (public metadata only; see `PRIV-03`) |
| MEM-10 | Event participation list with role per event |
| MEM-11 | Innovations/projects section |
| MEM-12 | Publications section |
| MEM-13 | Posts authored by the member |
| MEM-14 | Every section hides itself when empty rather than showing a placeholder |

### 7.3 Positions — data model rule

**Do not store `member.position = "Vice Chairperson"`.** Positions are historical records. This is what makes the journey timeline, past-committee pages, and alumni archive possible.

Each position record has: member, position title, team/department, session (e.g. 2026–2027), start date, end date (nullable), description, responsibilities, `is_current` derived from the end date.

| ID | Requirement |
|---|---|
| MEM-15 | A member may hold multiple positions over time, and more than one at once |
| MEM-16 | Position titles are DB rows with a display order, so committee pages sort correctly (Chair → Vice Chair → General Secretary → …) |
| MEM-17 | Only Executive Admin and above can create, edit, or end a position. Members can never self-assign. |
| MEM-18 | Ending a position never deletes it. The record stays for history. |

### 7.4 Own profile editing (`/me`)

| ID | Requirement |
|---|---|
| MEM-19 | Members may edit: preferred name, photo, bio, social links, skills/interests, and their privacy toggles |
| MEM-20 | Members may **not** edit: positions, official achievements, certificates, participation records, membership status, IEEE membership status |
| MEM-21 | Members submit corrections for admin-controlled fields through a request, not a direct edit |

### 7.5 Privacy

| ID | Requirement |
|---|---|
| PRIV-01 | Public by default: name, photo, position, journey, achievements, publications, projects, contributions, professional/social links |
| PRIV-02 | Private by default: email, phone, internal member ID, administrative notes, application data |
| PRIV-03 | Certificate public view shows title, type, event, issue date, and verification code. It does not expose internal issuance notes. |
| PRIV-04 | Members control visibility of: email, phone, batch, bio, social links, and individual achievements |
| PRIV-05 | Privacy is enforced at the API layer. A private field must never be sent to the client and hidden with CSS. |
| PRIV-06 | Members can request account deactivation; profile becomes non-public but records are retained for chapter history |

### 7.6 Alumni

| ID | Requirement |
|---|---|
| ALU-01 | Admin transitions a member from Active to Alumni (single action, with a date) |
| ALU-02 | The profile URL stays the same and stays public |
| ALU-03 | The profile shows an "Alumni" badge and the years active |
| ALU-04 | The alumni card shows photo, positions held, and sessions |
| ALU-05 | `/alumni` is browsable by session and by position |
| ALU-06 | An alumnus retains login and can still edit permitted profile fields unless deactivated |

---

## 8. Member-contributed content

### 8.1 Shared submission workflow

```
Member → Create draft → Submit → Admin review → Approved → Published
                                      ↓
                              Returned with feedback → Member edits → Resubmit
```

Applies to: Posts, Innovations/Projects, Achievements, Publications.

| ID | Requirement |
|---|---|
| SUB-01 | States: Draft · Submitted · Under Review · Approved/Published · Returned · Rejected · Archived |
| SUB-02 | Returned and Rejected items carry a required admin feedback note visible to the author |
| SUB-03 | `/me/submissions` shows every submission with its current state and feedback |
| SUB-04 | Author is notified on every state change |
| SUB-05 | Admins can edit before publishing; edits are logged |
| SUB-06 | Approved content shows author name, photo, and profile link |
| SUB-07 | The same review queue component serves all four content types. Do not build four separate moderation systems. |

### 8.2 Posts

| ID | Requirement |
|---|---|
| POST-01 | Post types: technical article, project showcase, research summary, experience, event report, opinion, learning resource |
| POST-02 | Rich text editor with images and code blocks |
| POST-03 | Cover image, tags, reading time |
| POST-04 | Post detail page with author card and related posts |
| POST-05 | Comments are out of scope for v1 unless Q-06 is resolved otherwise |

### 8.3 Innovations / Projects

| ID | Requirement |
|---|---|
| INN-01 | Fields: title, cover, summary, description, category (AI/ML, Software, Hardware, Research, Startup, Other), tech stack, team members, repo link, live demo link, date |
| INN-02 | Team members who are chapter members link to their profiles; external collaborators are plain names |
| INN-03 | `/innovations` filterable by category, year, and member |

### 8.4 Achievements

| ID | Requirement |
|---|---|
| ACH-01 | Categories: competition win, hackathon, award, scholarship, publication, certification, conference presentation, research, chapter award, external achievement |
| ACH-02 | Fields: title, category, position/rank, issuing body, event/competition name, date, description, proof link or file, related project |
| ACH-03 | `/achievements` filterable by year, category, member, and type |
| ACH-04 | Proof files are stored privately; only admins view them during review |

---

## 9. Publications

| ID | Requirement |
|---|---|
| PUB-01 | Fields: title, authors, abstract, publication type (journal / conference / preprint / poster / thesis), venue name, date, DOI, IEEE Xplore link, PDF (if licensing permits) |
| PUB-02 | Author entries link to member profiles when the author is a chapter member; external co-authors are plain text |
| PUB-03 | Filter by year, type, and author |
| PUB-04 | PDF upload is gated behind an admin confirmation that redistribution is permitted |
| PUB-05 | Citation string (IEEE format) generated and copyable |

---

## 10. Archive features

### 10.1 History (`/history`)

| ID | Requirement |
|---|---|
| HIS-01 | Visual vertical timeline by year, from chapter establishment to present |
| HIS-02 | Each year node can contain: executive committee of that session, major events, awards, publications, projects, milestones |
| HIS-03 | Year entries link to the relevant archived records rather than duplicating them |
| HIS-04 | Admins add milestone entries with title, date, description, and image |
| HIS-05 | Timeline is keyboard-navigable and readable without JavaScript animation |

### 10.2 Gallery (`/gallery`)

| ID | Requirement |
|---|---|
| GAL-01 | Organized by Year → Album. Albums usually map to an event but can be standalone. |
| GAL-02 | Photos and videos (video via embed link, not self-hosted, unless Q-08 says otherwise) |
| GAL-03 | Lightbox viewer with keyboard navigation |
| GAL-04 | Images served in modern formats with responsive sizes and lazy loading |
| GAL-05 | Bulk upload with drag-and-drop for Media/Department Admins |
| GAL-06 | Each event detail page links to its album, and each album links back to its event |

### 10.3 Sponsors and collaborations

| ID | Requirement |
|---|---|
| SPO-01 | Partner types: Industry, Community, Academic, Technical |
| SPO-02 | Fields: name, logo, type, website, description, relationship period, associated events |
| SPO-03 | Homepage shows a curated subset; `/sponsors` and `/collaborations` show all |
| SPO-04 | Past partners are archived, not deleted |

---

## 11. Certificates and verification

| ID | Requirement |
|---|---|
| CER-01 | A certificate is issued against a participation record or an achievement, by Executive Admin or above |
| CER-02 | Fields: recipient, title, type (Participation / Volunteer / Organizer / Speaker / Achievement / Appreciation), related event, issue date, issuer, unique verification code, certificate file or generated render |
| CER-03 | Each certificate gets a unique, non-guessable verification code (not sequential) |
| CER-04 | `/verify/[code]` publicly shows: valid/invalid, recipient name, certificate title, type, event, issue date, issuing chapter |
| CER-05 | `/verify` provides a code entry form |
| CER-06 | Certificates carry a QR code or printed URL pointing to their verification page |
| CER-07 | Admins can revoke a certificate; verification then returns "Revoked" with the revocation date, never a 404 |
| CER-08 | Members download their own certificates from `/me/certificates` |
| CER-09 | Bulk issuance: select an event, select participants by role, issue in one action |
| CER-10 | Verification pages are rate-limited to prevent code enumeration |

---

## 12. Admin dashboard

```
Dashboard
├── Overview          (stats, pending queue, recent activity)
├── Members           (list, create, edit, status, merge)
├── Executive Committee (positions by session)
├── Events            (CRUD, registrations, participation, export)
├── Posts             (review queue)
├── Achievements      (review queue)
├── Certificates      (issue, bulk issue, revoke)
├── Publications
├── Projects / Innovations
├── Gallery           (albums, bulk upload)
├── Sponsors
├── Collaborations
├── History
├── Applications      (join requests)
├── Notifications
└── Website Settings  (roles, permissions, nav, footer, stats config)
```

| ID | Requirement |
|---|---|
| ADM-01 | The dashboard must be usable by non-technical committee members. No raw JSON, no database jargon. |
| ADM-02 | Overview shows counts of pending submissions, pending applications, upcoming events, and recent activity |
| ADM-03 | Every list view has search, filter, sort, and pagination |
| ADM-04 | Navigation items render based on the logged-in user's permissions |
| ADM-05 | Deletes are soft deletes with a restore window; hard delete is Super Admin only |
| ADM-06 | Audit log is viewable by Executive Admin and above |
| ADM-07 | Website Settings (Super Admin only): roles and permissions, admin accounts, nav/footer links, homepage section toggles, contact details, social links, statistics configuration |

### 12.1 Join applications

| ID | Requirement |
|---|---|
| APP-01 | `/join` form collects: name, email, phone, student ID, department, batch, IEEE membership number (optional), interests, motivation |
| APP-02 | Application states: Submitted · Under Review · Approved · Rejected |
| APP-03 | On approval, a member account is created and an invite/set-password email is sent |
| APP-04 | Applicant is notified of the outcome |
| APP-05 | Rejected applications are retained for the records |

---

## 13. Data model

Proposed entities and key fields. Relationship rules matter more than exact column types.

| Entity | Key fields | Notes |
|---|---|---|
| `users` | id, email, password/auth ref, role, status, created_at | Auth identity |
| `members` | id, user_id, slug, full_name, preferred_name, photo, bio, university, department, batch, joining_date, ieee_membership_status, ieee_cs_membership_status, membership_status, alumni_since | Profile |
| `member_links` | member_id, type, url | GitHub, LinkedIn, portfolio, etc. |
| `member_privacy` | member_id, field, is_public | Per-field toggles |
| `positions` | id, member_id, title_id, team_id, session, start_date, end_date, description, responsibilities | **Historical records** |
| `position_titles` | id, name, display_order, level | Sorting for committee pages |
| `teams` | id, name, description | Technical, Creative, Research, Media, Events |
| `events` | id, slug, title, banner, description, category_id, start_at, end_at, venue, is_online, registration_status, capacity, show_participants, report | |
| `event_categories` | id, name, slug | Admin-editable |
| `event_speakers` | event_id, name, designation, photo, member_id (nullable) | |
| `event_registrations` | id, event_id, member_id (nullable), name, email, student_id, department, batch, phone, created_at | |
| `event_participations` | id, event_id, member_id, role, confirmed_by, confirmed_at | Drives profiles |
| `certificates` | id, member_id, title, type, event_id, achievement_id, issue_date, issued_by, verification_code, file_url, revoked_at | |
| `achievements` | id, member_id, title, category, rank, issuer, event_name, date, description, proof_url, project_id, status | Goes through review |
| `projects` | id, title, summary, description, category, cover, tech_stack, repo_url, demo_url, date, status | Innovations |
| `project_members` | project_id, member_id (nullable), external_name, role | |
| `publications` | id, title, abstract, type, venue, date, doi, xplore_url, pdf_url, status | |
| `publication_authors` | publication_id, member_id (nullable), external_name, author_order | |
| `posts` | id, slug, author_member_id, type, title, cover, body, tags, status, published_at, reviewed_by, feedback | |
| `gallery_albums` | id, title, year, event_id (nullable), cover | |
| `gallery_items` | id, album_id, type, url, caption, taken_at | |
| `partners` | id, name, logo, type, website, description, start_date, end_date | Sponsors + collaborations |
| `history_entries` | id, year, title, date, description, image, related_type, related_id | Timeline |
| `applications` | id, name, email, phone, student_id, department, batch, ieee_number, interests, motivation, status, reviewed_by | |
| `notifications` | id, user_id, type, payload, read_at | |
| `audit_log` | id, actor_user_id, action, entity_type, entity_id, before, after, created_at | |
| `settings` | key, value | Site-wide config |
| `contact_messages` | id, name, email, subject, message, created_at | |

`DATA-01` (MUST) Nothing that represents chapter history is ever hard-deleted by a normal admin action.
`DATA-02` (MUST) Records that can reference either a member or an external person use a nullable `member_id` plus a text fallback.
`DATA-03` (MUST) The journey timeline is a query across `positions`, `event_participations`, `achievements`, and `certificates`. It is never a stored text field.
`DATA-04` (SHOULD) Sessions (e.g. `2026-2027`) are a first-class value so committee pages and alumni archives can group by them.

---

## 14. Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-01 | Responsive across mobile, tablet, desktop. Mobile is the primary traffic assumption. |
| NFR-02 | WCAG 2.1 AA: contrast, keyboard navigation, focus states, alt text, `prefers-reduced-motion` |
| NFR-03 | Public pages: Largest Contentful Paint under 2.5s on a mid-range phone on 4G |
| NFR-04 | All public pages server-rendered or statically generated for SEO; per-page meta tags and Open Graph images |
| NFR-05 | Passwords hashed; sessions secure and HTTP-only; CSRF protection on all mutations |
| NFR-06 | File uploads validated by type and size; images stripped of EXIF location data |
| NFR-07 | Rate limiting on login, registration, contact form, and certificate verification |
| NFR-08 | Automated daily database backups with a documented restore procedure |
| NFR-09 | Dark mode (SHOULD) |
| NFR-10 | Design system with shared tokens and components; no ad-hoc per-page styling |
| NFR-11 | Sitemap.xml and robots.txt; structured data for events and articles |

---

## 15. Build plan with verification gates

Each phase ships working software. Do not begin a phase until the previous gate passes.

### Phase 0 — Foundations
Scope: repo setup, design system and tokens, base layout (header/footer/nav), database schema, auth, role and permission framework, audit log, admin dashboard shell.
**Gate:** A Super Admin can log in, see an empty dashboard, and create a second admin with a restricted role. A Member account cannot reach any `/dashboard` route. Schema migrations run clean from zero.

### Phase 1 — Public site
Scope: Home, About, Contact, Join form + applications queue, sponsors/collaborations, site settings, footer/nav management.
**Gate:** Every homepage section renders correctly with real content and with zero content. A non-technical committee member edits the About page and a footer link without developer help. Lighthouse performance and accessibility both ≥ 90 on Home.

### Phase 2 — Events
Scope: Event CRUD, categories, list with filters, detail page, registration, registration export, participation records.
**Gate:** A full event lifecycle works end to end: create → publish → accept registrations → close → confirm participants → mark completed. Status transitions are derived from dates, not manual. Registration CSV exports correctly.

### Phase 3 — Members, positions, profiles
Scope: Member CRUD, slugs, directory, public profile, `/me` editing, privacy toggles, positions and position titles, `/team`, `/team/[session]`, journey timeline, alumni transition, `/alumni`.
**Gate:** A member with three positions across two sessions and five event participations renders a correct chronological journey with no manual data entry. Private fields are absent from the API response, verified by inspecting the network payload. An alumni transition keeps the URL live.

### Phase 4 — Certificates and verification
Scope: Certificate issuance, bulk issuance, revocation, `/me/certificates`, `/verify` and `/verify/[code]`, QR codes.
**Gate:** Bulk-issue certificates to 20 participants in one action; scan a QR from a printed certificate and land on a valid verification page; revoke one and confirm the page reports Revoked rather than 404. Verification is rate-limited.

### Phase 5 — Member content
Scope: Shared submission workflow, review queue, posts, innovations, achievements, publications, notifications, `/me/submissions`.
**Gate:** One reviewer queue handles all four content types. A returned submission carries feedback to the author, who edits and resubmits successfully. Approved content appears on the homepage and on the author's profile automatically.

### Phase 6 — Archive and polish
Scope: Gallery, History timeline, achievements/innovations/publications public pages with filters, SEO, dark mode, backups, admin documentation.
**Gate:** Full content dry run with real chapter data from at least two past sessions. Handover documentation exists and a committee member who has never seen the codebase can perform every routine content task from it.

---

## 16. Open questions

| ID | Question | Why it matters |
|---|---|---|
| Q-01 | What is the tech stack and hosting? | Everything downstream depends on this. Next.js + Supabase is a natural fit given the team's existing skills, but it needs a decision. |
| Q-02 | Who owns the domain, hosting account, and database after this committee graduates? | The whole "permanent archive" goal fails if the account is tied to one graduating student. |
| Q-03 | How do members log in? Email/password, Google, or IEEE-verified email only? | Affects auth build and onboarding friction. |
| Q-04 | Can anyone register an account, or only admin-approved applicants? | Current assumption: approval-only via `/join`. Confirm. |
| Q-05 | Approximate scale: how many members, events per year, and photos per event? | Drives storage and cost decisions. |
| Q-06 | Are comments or reactions wanted on posts? | Currently out of scope; adds moderation burden. |
| Q-07 | Are certificates uploaded as designed PDFs, or generated by the system from a template? | Generation is significantly more work but removes manual effort forever. |
| Q-08 | Where do gallery images and videos live? Object storage, Google Drive, or YouTube embeds? | Cost and performance. |
| Q-09 | How far back does the chapter history go, and does that archive data exist in a usable form? | Determines a backfill effort. |
| Q-10 | Which roles map to which department admin scopes in your actual committee structure? | Needed before Phase 0 permission seeding. |
| Q-11 | What are the IEEE branding and web presence requirements the chapter must comply with? | IEEE has rules for chapter sites, logos, and naming. Better to find out before design, not after. |
| Q-12 | Bengali/English bilingual, or English only? | Affects content model and all copy. |
| Q-13 | Do you need paid event registration at any point? | Currently NG-01. |
| Q-14 | What is the target launch date and what is the hard minimum scope for it? | Determines where to cut the phase plan. |

---

## 17. Glossary

| Term | Meaning |
|---|---|
| Chapter | IEEE Computer Society Leading University Student Branch Chapter |
| Session | An executive committee term, e.g. 2026–2027 |
| Position | A titled role held by a member for a defined period |
| Participation record | Confirmed involvement of a member in an event, with a role |
| Journey | The generated timeline of a member's chapter history |
| Submission | Member-created content awaiting admin review |
