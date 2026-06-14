# CLAUDE.md — CDS Offline website

Context file for anyone (human or agent) working in this folder. Read this
before touching `index.html`. The site is **already built**; the job is to
fill it in and refine it, not to regenerate it from scratch.

---

## What this is

A one-page website for **CDS Offline**, a club at Boston University (in/around
the Faculty of Computing & Data Sciences). The club's whole point is community
*offline* — people meeting in person instead of through feeds and DMs. The site
is the club's public face: club description, next meeting, e-board, past events,
and an interest/email sign-up.

Immediate goal: a working demo to show the rest of the club. It can be rewritten
in another stack later — for now, ship the static version.

---

## Stack & hard constraints

- **Plain HTML + CSS + vanilla JS. One self-contained file: `index.html`.**
- **No framework. No React, Vue, Next, Astro, Tailwind, or build step.** Do NOT
  "upgrade" this to a framework, even if it seems cleaner. Reason: it's a
  one-page static site hosted on GitHub Pages; a framework adds a build
  toolchain and deploy config for zero benefit and makes it harder for non-dev
  club members to edit. React is a *maybe-later* if the site grows into a
  multi-page app — not now.
- **Hosted on GitHub Pages.** The file must stay named `index.html` and live at
  the repo root so Pages serves it at the base URL.
- Keep everything in the single file (inline `<style>` and `<script>`). Don't
  split into separate CSS/JS files unless explicitly asked.
- External dependencies are limited to Google Fonts (loaded via `<link>`). No
  npm, no bundler, no CDN JS libraries.

---

## Working rules (read these twice)

1. **Do not write copy. Leave blanks.** Every piece of text the user hasn't
   explicitly provided stays as a bracketed placeholder, e.g. `[ headline ]`,
   `[ Name ]`, `[ caption ]`, styled with the `.ph` class (faded). **Never
   invent** taglines, descriptions, names, dates, locations, event captions,
   or "filler" personality. No AI slop. If you don't have the exact words from
   the user, the slot stays blank.
2. **Wait to be told what goes where.** The user fills placeholders by giving
   explicit content ("hero headline: X", "treasurer is Y"). Only then replace
   the matching `[ bracket ]`. Don't guess or "improve" wording.
3. **Brutal honesty, no hedging.** If something won't work, say so plainly and
   say why. No "this could work" / "maybe try" filler. State the realistic
   tradeoff and the recommended path.
4. **Edit the existing file.** Modify `index.html` in place. Don't regenerate
   the whole thing or restructure unless asked.
5. **Preserve the design system** below. Don't drift toward generic defaults
   (cream + serif + one accent, etc.). The look is intentional.

---

## Design system (keep consistent)

Direction: a **risograph / zine / printed-flyer** aesthetic — the look of a club
event flyer stapled to a corkboard. Bold flat inks, halftone texture, monospace
labels, slight rotation on pinned items. The irony is the point: an anti-screen
club whose site looks printed, not digital.

**Type**
- Display / headlines: **Bricolage Grotesque** (700–800), big and uppercase.
- Body: **Work Sans** (400–600).
- Labels / meta / data: **Space Mono** (uppercase, tracked).

**Color — palettes via `data-palette` on `<html>`**
Colors are CSS custom properties (`--paper`, `--paper-2`, `--ink`, `--ink-2`,
`--text`, `--muted`, `--line`). Five named palettes are defined in the
`<style>` block:
- `classic` (federal blue + fluoro pink) — current default
- `forest` (green + sunflower)
- `plum` (purple + aqua)
- `red` (single-ink red)
- `midnight` (dark mode: warm coral + cyan)

To change the **default**, edit `<html data-palette="...">` at the top of the
file. To add a palette, copy a `html[data-palette="x"]{...}` block and add a
matching `.sw-x` swatch + button.

**Signature element:** the live **ink switcher** in the top-right nav. Clicking a
swatch re-inks the whole page instantly (sets `data-palette`). This is both the
memorable design moment AND the tool the club uses to vote on colors. Spend the
visual boldness here and on the halftone; keep everything else quiet and
disciplined.

**Quality floor:** responsive to mobile (nav collapses to a hamburger), visible
keyboard focus, `prefers-reduced-motion` respected. Don't regress these.

---

## Page structure

Single scroll. Sticky masthead nav links to each section by `id`:

| Nav label     | `id`     | Contains (currently blank placeholders)                          |
|---------------|----------|------------------------------------------------------------------|
| —             | `#top`   | Hero: banner line, headline (2 lines), one-line pitch, 3 tags    |
| What we do    | `#about` | Heading, 2 paragraphs, 5-item activity list                      |
| Next meeting  | `#next`  | "Ticket stub": day/date/time, location, address + a prep checklist |
| E-board       | `#board` | 4 ID-badge cards: name, role, photo (initials/`<img>`)           |
| Past events   | `#past`  | 4 taped-polaroid cards: photo + caption                          |
| Join          | `#join`  | Heading, blurb, sign-up form (name / email / year)               |
| —             | footer   | Club name (kept), tagline/colophon (blank), email (blank)        |

The only non-placeholder text is the user's real facts: **"CDS Offline"** and
**"Boston University."** Everything else is `[ bracketed ]`.

---

## The form (important constraint)

GitHub Pages is **static — there is no server**, so the form cannot store
submissions on its own. It must POST to a third-party handler. Current state:
**demo mode** — `FORM_ENDPOINT = ""` in the script, so submitting just shows a
success message and stores nothing.

To make it collect real emails, pick one:
- **Google Form (recommended for a club):** free, unlimited, lands in a Google
  Sheet the whole e-board can see. Replace the `.sheet` block with the Google
  Form `<iframe>` embed.
- **Tally:** free + unlimited; set `FORM_ENDPOINT` to the Tally URL, or embed.
- **Formspree:** works, but **do not rely on the free tier** — it's ~25
  submissions/month, which a public club form burns through fast.

Don't silently "fix" the form by pretending it works — it's genuinely inert
until an endpoint is connected.

---

## Hosting / deploy

Fastest (no git, browser only):
1. New GitHub repo → Public → upload `index.html` to root.
2. Settings → Pages → Deploy from a branch → `main` → `/(root)` → Save.
3. Live at `https://<user>.github.io/<repo>/` in ~1 min.

Local-folder workflow (better for ongoing edits): keep this folder, edit
`index.html`, double-click to preview (static HTML needs no local server), then
`git add/commit/push` to the repo — Pages auto-redeploys. Optional custom
domain (`cdsoffline.com`, ~$10/yr): add a CNAME to `<user>.github.io` and set it
in Settings → Pages.

---

## Current state

`index.html` is complete and functional: full layout, all 5 palettes + live
switcher, responsive nav, working demo form. All copy is blanked to `[ ... ]`
placeholders awaiting the user's exact wording. Next step is the user providing
content to slot into the blanks — section by section.