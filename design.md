# Design System
## IEEE Computer Society — Leading University SB Chapter Website

| Field | Value |
|---|---|
| Document version | v0.3 (draft) |
| Companion doc | ieee-cs-lusb-website-prd.md |
| Status | Colors and typography locked; homepage structure patterns added (§13) |

---

## 0. How to read this document

This is the single source of truth for visual tokens and component rules (see PRD `NFR-10`). No page should invent its own colors, spacing, or type sizes — everything traces back to a token defined here.

---

## 1. Brand foundation

**Revision note (v0.2):** the color and type system below now follows a reference board supplied directly for this project — an orange, a dark brown, a near-black, a gray, and white, paired with Neue Montreal — rather than the earlier logic of echoing the logo's own blue-plus-orange structure. The chapter logo's amber is still never reused verbatim anywhere in the token set below.

- The chapter logo itself is never re-colored or redrawn. It appears in its original form in the header and footer.
- IEEE and IEEE CS branding compliance (PRD `Q-11`) is still open. Confirm logo lockup, minimum clear space, and pairing-with-Master-Brand rules with the chapter advisor before final header/footer implementation. See the Computer Society's own brand guide and logo-use policy for the current rules.

---

## 2. Color tokens

Source palette (supplied directly): orange `#FF6D29` · dark brown `#453027` · near-black `#161316` · gray `#BABABA` · white `#FFFFFF`. There is only one brand hue now (orange) rather than the earlier orange-plus-blue pair, so a couple of status roles below intentionally share `primary-500` across screens that never appear side by side.

### 2.1 Brand

| Token | Value | Use |
|---|---|---|
| `primary-50` | `#FFE7D9` | Subtle tinted backgrounds (selected nav item, highlighted row) |
| `primary-500` | `#FF6D29` | Default — buttons, links, active states, "Upcoming" indicators. **Text sitting on this fill uses `text-on-primary` below, not `text-inverse`** — white fails contrast here (~2.8:1). |
| `primary-600` | `#E85A1C` | Hover |
| `primary-700` | `#C24515` | Active / pressed |

### 2.2 Semantic

| Token | Light | Dark | Use |
|---|---|---|---|
| `success-50` / `success-500` | `#E8F3EA` / `#3F7D4A` | `#1C2A1E` / `#7FC08C` | Open registration, Active member, Ongoing event |
| `warning-50` / `warning-500` | `#FBF3E1` / `#C79A2E` | `#2B2415` / `#E6C05C` | Full capacity, Coming soon, Applicant |
| `danger-50` / `danger-500` | `#FBEAEA` / `#B23A3A` | `#2B1A1A` / `#E38686` | Closed, revoked certificate, form errors |
| `neutral-500` | `#707070` | `#BABABA` | Completed event, Inactive, disabled. Darkened from the reference gray in light mode — see §10. |

### 2.3 Surfaces & text

| Token | Light | Dark | Use |
|---|---|---|---|
| `bg-page` | `#F7EAE2` (tender rose gold) | `#453027` (cocoa brown) | Page background |
| `surface-card` | `#FFFFFF` | `#5A4436` | Cards, panels, modals |
| `border` | `#E3D2C6` | `#6B5342` | Dividers, card borders, input outlines |
| `text-primary` | `#161316` | `#F7EAE2` | Body copy, headings |
| `text-secondary` | `#453027` | `#BABABA` | Supporting copy, labels |
| `text-muted` | `#707070` | `#948577` | Placeholders, captions, timestamps — see §10 for the dark-mode value's headroom |
| `text-inverse` | `#F7EAE2` | `#F7EAE2` | Text on near-black or cocoa-brown fills (dark nav bars, ink-colored chips) |
| `text-on-primary` | `#161316` | `#161316` | Text specifically on the `primary-*` orange fills, at any shade |

### 2.4 Status mapping (reuse the tokens above — don't hardcode new colors per feature)

| Domain | States → token |
|---|---|
| Event status (`EVT-01`, `EVT-05`) | Upcoming → `primary-500` · Ongoing → `success-500` · Completed → `neutral-500` |
| Registration status (`EVT-11`) | Open → `success-500` · Closed → `neutral-500` · Full → `warning-500` · Coming soon → `primary-500` |
| Membership status | Applicant → `warning-500` · Active → `success-500` · Alumni → `primary-500` · Inactive → `neutral-500` |
| Submission review | Pending → `warning-500` · Approved → `success-500` · Returned → `danger-500` |

---

## 3. Typography

**Neue Montreal**, one family throughout — Semibold/Medium for headings, Regular for body — rather than a heading/body pairing.

**Licensing:** Neue Montreal is a commercial typeface from Pangram Pangram; its free tier covers personal/portfolio use only, and a webfont license for the live chapter site starts at roughly $30. Either budget that small one-time cost, or self-host **General Sans** (Fontshare, free for personal and commercial use) as a near-identical neo-grotesque substitute — the scale below holds unchanged either way, it's only the `font-family` value that changes.

| Style | Size (desktop) | Size (mobile) | Weight | Line-height |
|---|---|---|---|---|
| Display | 48px / 3rem | 36px / 2.25rem | Semibold | 1.15 |
| H1 | 36px | 30px | Semibold | 1.2 |
| H2 | 28px | 24px | Semibold | 1.25 |
| H3 | 22px | 20px | Medium | 1.3 |
| H4 | 18px | 18px | Medium | 1.35 |
| Body large | 18px | 16px | Regular | 1.6 |
| Body | 16px | 16px | Regular | 1.6 |
| Small | 14px | 14px | Regular | 1.5 |
| Caption | 12px | 12px | Medium | 1.4 |

Neue Montreal has no Bengali glyphs. If Bengali ever comes into scope (PRD `Q-12`), pair it with **Noto Sans Bengali** for Bengali runs, matched by weight rather than family, since the two won't share a type system.

---

## 4. Spacing & layout

| Token | Value |
|---|---|
| `space-xs` | 4px |
| `space-sm` | 8px |
| `space-md` | 16px |
| `space-lg` | 24px |
| `space-xl` | 32px |
| `space-2xl` | 48px |
| `space-3xl` | 64px |
| `space-4xl` | 96px |

**Breakpoints** (mobile-first, per `NFR-01`): mobile `0–639px` · tablet `640–1023px` · desktop `1024px+`. Container max-width: `1200px`, with `space-md`/`space-lg` side gutters on mobile/desktop respectively.

---

## 5. Radius, borders, elevation

| Token | Value | Use |
|---|---|---|
| `radius-sm` | 6px | Buttons, inputs, badges |
| `radius-md` | 10px | Cards |
| `radius-lg` | 16px | Modals, hero panels |
| `radius-pill` | 999px | Status pills, tags |
| `border-width` | 1px | Default rule width |
| `shadow-sm` | `0 1px 2px rgba(36,28,20,0.06)` | Resting cards |
| `shadow-md` | `0 4px 12px rgba(36,28,20,0.10)` | Dropdowns, popovers |
| `shadow-lg` | `0 12px 32px rgba(36,28,20,0.16)` | Modals, dialogs |

In dark mode, prefer stepped surface color (`bg-page` → `surface-card` → a lighter third step for popovers) over shadows — shadows barely read against dark backgrounds.

---

## 6. Iconography

- Outline-style icon set (e.g. Lucide or Phosphor), 1.5px stroke.
- Inline sizes: 16px and 20px. Standalone/decorative: 24px max.
- One icon per event category (`EVT-03`) and one per participation role (`EVT-19`), reused consistently across cards, filters, and profile timelines.

---

## 7. Imagery & photography

- Real chapter photography only — no stock imagery (`HOME-02`).
- Crop ratios: hero banners `21:9`, event/post cards `16:9`, member photos `1:1`, gallery `masonry, uncropped`.
- Hero text over photos sits on a `rgba(23,17,11,0.45)` scrim (dark mode: `rgba(0,0,0,0.55)`) for legibility, never a hard color block.
- Empty states (`HOME-14`, `MEM-14`): an icon + one line of copy in `text-muted`, never a broken image or a placeholder photo.

---

## 8. Motion

| Token | Value |
|---|---|
| `duration-fast` | 120ms — hover, checkbox, toggle |
| `duration-base` | 200ms — dropdown, tab switch, accordion |
| `duration-slow` | 320ms — mobile drawer, page transition |
| `ease-out` | `cubic-bezier(0.16, 1, 0.3, 1)` — entrances |
| `ease-in-out` | default — toggles |

Respect `prefers-reduced-motion`: disable slide/scale transitions and fall back to instant or opacity-only changes (`NFR-02`, `HOME-02`).

Two additional patterns, borrowed from bracucc.org's homepage:

- **Stat count-up** — the `HOME-04` impact numbers animate from 0 to their real value once, over `duration-slow`, triggered on scroll into view. Under `prefers-reduced-motion`, show the final number immediately — don't just skip the animation and strand it at 0.
- **Marquee ticker** — a slow, continuous horizontal scroll of the `EVT-03` event categories (Workshop, Hackathon, Seminar, …) beneath the hero, content duplicated once for a seamless loop. This is exactly the kind of persistent motion `HOME-02` asks to respect `prefers-reduced-motion` for — swap it for a static wrapped row of the same tags rather than leaving it spinning.

---

## 9. Components (spec at a glance)

| Component | Notes |
|---|---|
| Button | Variants: primary (`primary-500` fill), secondary (`border` outline), ghost, destructive (`danger-500`). States: default, hover, active, disabled (40% opacity), loading. |
| Form field | `surface-card` fill, `border` outline, standard focus ring (§10), `danger-500` error state with inline message below the field. |
| Card (event/member/achievement/project/post) | `surface-card`, `radius-md`, `shadow-sm`, consistent header/body/footer slots. |
| Status badge/pill | `radius-pill`, background = semantic `-50` tint, text = semantic `-500`. |
| Nav (desktop bar + mobile drawer) | Per PRD `4.1`. Active link underlined in `primary-500`. |
| Member journey timeline | Vertical line in `border`, event dots colored by type (position/participation/achievement/certificate). |
| Stat counters (`HOME-04`) | Large `Display`/`H1` numeral in `text-primary`, label in `text-muted` below. |
| Table (admin, registrations) | `border`-separated rows, `bg-page` header row, sticky on scroll. |
| Pillar card (numbered) | Large numeral (01/02/03), title, one-line role, 2–3 line description, two bullet highlights, linked tag chips below. Groups `ABT-03`'s nine "What We Do" items into three pillars for the homepage "why join" section — see §13. |
| Feature card (photo + tags) | Photo, small eyebrow label, title, short description, a tag-chip row, trailing "Explore →" link. One card shape reused for `HOME-05` events, `HOME-08` innovations, and `HOME-10` sponsor groups — swap the content, not the shape. |
| Marquee | Single-line auto-scrolling tag strip (see §8 for the motion spec and its `prefers-reduced-motion` fallback). |
| Contact + FAQ split panel | Optional, for `/contact` (`CON-01`–`CON-03`): a two-column layout, contact form on one side, an FAQ accordion on the other. Not required by the PRD, but a clean pattern worth borrowing. |
| Empty / loading state | Centered icon + `text-muted` copy; loading uses a skeleton in `border` color, not a spinner, for cards/lists. |

---

## 10. Accessibility

- Contrast: body text ≥ 4.5:1, large text/UI components ≥ 3:1 against their background (`NFR-02`). All token pairs above have been chosen to clear this against their paired surface.
- The reference gray (`#BABABA`) only clears AA on the cocoa-brown dark background (~7.1:1). Against the light rose-gold/white surfaces it drops to ~1.9:1, so raw `#BABABA` is restricted to non-text UI there — dividers, disabled fills — while any muted *text* in light mode uses the darkened `neutral-500`/`text-muted` value (`#707070`, ~4.9:1) instead.
- The primary orange (`#FF6D29`) clears AA as standalone text/links on the dark cocoa background (~4.9:1) but not on light surfaces (~2.4:1) — keep it to fills, icons, and borders on light backgrounds, paired with `text-primary` ink for any label. It also fails as a background for *white* text at any shade used as a button fill (~2.8:1 at `primary-500`); use `text-on-primary` (dark ink) on every `primary-*` fill instead.
- `text-muted` in dark mode (`#948577`) sits around 3.9:1 against `bg-page` — fine for captions and timestamps, but avoid it for anything that functions as primary reading text.
- Focus ring: 2px solid `text-primary` (mode-aware — near-black in light mode, rose-gold in dark), 2px offset, visible on every interactive element — never removed, only restyled. `text-primary` is used here rather than the brand orange because no single orange shade clears 3:1 against both `bg-page` extremes at once; the mode's own ink color always does.
- All interactive elements reachable and operable by keyboard alone; skip-to-content link on every page.

---

## 11. Certificates & verification pages

Certificates and `/verify/[code]` (PRD `Q-07`, section 8) are printed/shared outside the site's normal chrome, so they get their own restrained palette rather than the full web token set: `text-primary` ink, a thin `primary-500` border or seal, and the chapter logo — no accent blue, no semantic colors. Keep this palette locked even if the web theme changes later, since printed certificates can't be retroactively updated.

---

## 12. Implementation notes

- Ship tokens as CSS custom properties (or a Tailwind theme extension, if `Q-01`'s Next.js lean is confirmed), one variable per token name above — never a raw hex value inline in component code.
- Dark mode: attribute-based (`data-theme="dark"`) or `prefers-color-scheme`, your call — but every token in sections 2–5 needs both values defined before Phase 0 closes, per `NFR-09`/`NFR-10`.

---

## 13. Homepage composition — structure reference: bracucc.org

Colors (§2) and type (§3) are unchanged. What follows is section order and card patterns, mapped from bracucc.org's homepage onto this chapter's own content (`HOME-01`–`HOME-14`), not onto BUCC's content.

1. **Hero** (`HOME-01`, `HOME-02`) — headline, subtext, `[Explore Chapter]` `[Join Us]`, real chapter photography with a subtle technical/network motif behind it. The `EVT-03` category marquee (§8, §9) sits directly beneath it, standing in for BUCC's scrolling activity-tag line.
2. **Impact stats** (`HOME-04`) — the stat row from the earlier mockup, now with the count-up animation from §8.
3. **Why join, in three pillars** — group `ABT-03`'s nine "What We Do" items into three numbered Pillar cards (§9), echoing BUCC's Mind/Career/Soul structure without borrowing its actual labels. Pick three groupings that fit this chapter — e.g. *Technical Growth*, *Career & Projects*, *Community*.
4. **Recent events** (`HOME-05`) — Feature cards (§9), status pill from §2.4. A light version of the `EVT-01` status tabs sits above the row.
5. **Innovations showcase** (`HOME-08`) — same Feature card shape; tags become tech stack instead of focus areas; "Explore Member Showcase →" links to `/innovations`.
6. **Achievements showcase** (`HOME-09`) — a tighter strip variant of the Feature card — icon and trophy line instead of a photo, no eyebrow/tag row needed.
7. **Team preview** (`HOME-06`) — photo, name, position, linked to the profile. The same simple grid BUCC uses for advisors and its governing body, collapsed to one row for the current exec.
8. **Sponsors & collaborations** (`HOME-10`) — grouped by type (Industry/Community/Academic/Technical) as logo rows, not full Feature cards — a sponsor doesn't need the eyebrow/description treatment a department does.
9. **Closing CTA** (`HOME-11`) — full-width banner, primary + ghost button pair, matching the tone of BUCC's "Ready to Upgrade Yourself?" close.
10. **Footer** (`HOME-12`, PRD §5.2) — logo and one-line description, social icon row, the four existing link columns, address/contact block, base row. BUCC's small credit line ("Crafted by BUCC R&D") is worth keeping as a nod to whoever builds this.

Every section still needs its `HOME-14` empty state before it ships. BUCC's own events section ships a plain "stay tuned" message instead of hiding the section — that's the right instinct to copy.

**Deliberately not carried over:** department-style content (this chapter has no BUCC-style department structure per the PRD) and the "Initializing 3D Space" WebGL interstitial — a nice flourish, but real engineering scope this PRD doesn't currently budget for. Worth a `MAY` in the backlog, not a v1 ask.
