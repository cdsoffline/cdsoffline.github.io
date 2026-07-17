# AGENTS.md — CDS Offline: verified change backlog

Produced 2026-07-17 from a multi-agent review (5 dimensions, every finding
adversarially re-verified with headless-Chrome measurements and computed WCAG
ratios). Line numbers refer to `index.html` as of commit `a244ea6` + working
tree. **No changes have been applied yet — this file is the to-do list.**

**Status 2026-07-17 (second pass):** items 1, 2, 3, 4, 8, 20, 25 are DONE and
verified by measurement (plus a new default palette `slate` — off-white
`#F2F1EC` / dark blue `#1D3557` / charcoal `#34383E`, AA-clean at 6.3–10.9:1,
added as a 6th swatch). Item 16 (repo rename) is BLOCKED on the user running
`gh repo rename cdsoffline --repo PlatinumPawn/cdsoffine --yes` — agents are
not permitted to rename repos. Everything else below is still open. The
masthead nav now collapses at 940px, not 760 — keep that in mind for any
future masthead work.

Read `CLAUDE.md` first for the hard constraints. They all still hold:
single self-contained `index.html`, no frameworks, no build step, GitHub
Pages, don't invent copy (facts stay `[ bracketed ]`). Note that `CLAUDE.md`
itself is stale in places — see "Docs drift" below.

---

## P0 — Real bugs (fix before showing the demo)

### 1. All three logo SVGs are missing `viewBox` — one attribute fixes three bugs
The `<use href="#outlet">` instances at lines **223** (masthead), **263**
(hero), **398** (join) have no `viewBox`, so with `height:auto` each SVG falls
back to the 150px default height. Verified consequences:
- **Masthead brand link is an invisible 151×150px click trap.** Its box
  extends ~43px *below* the sticky header (viewport y 62–105) at all widths
  ≥761px; `elementFromPoint` confirms left-edge content scrolling through that
  strip is unclickable and clicks jump to `#top` instead.
- **Hero mark renders at 49% of the disc width** instead of the intended 74%
  (line 102) — the disc looks under-filled.
- **Join mark letterboxes to 340×150** instead of 340×247 — the green right
  half reads as empty, with ~67px invisible letterbox each side.

**Fix (verified by re-measurement):** add `viewBox="0 0 132 96"` to each of
the three outer `<svg>` elements. Results measured after fix: brand 30×22
(anchor back to ~31px tall), hero 311×226, join 340×247. No CSS changes
needed. Afterwards, optionally re-judge `.join-mark` width from a fresh
screenshot — at true size it may deserve `min(32vw,400px)`.

### 2. Masthead is broken in the 761–886px band (iPad portrait widths)
Between the mobile collapse (760px) and ~886px the desktop bar cannot fit:
nav labels wrap to two lines (46px vs 28px, measured at 761/780/820/840/860),
"Ink ↓" wraps, and swatches squash to ovals (18.8px wide, no `flex-shrink:0`
on `.swatch`, line 81). This band covers iPad portrait CSS widths
(768/810/820/834). Evidence: `.nav` lines 73–74, `.inks` 79–83, breakpoint
line 175; screenshot showed "WHAT WE / DO", "E- / BOARD" wrapped.

**Fix:** move the nav-collapse media query from 760px to ~900px (measured
unwrap point is between 885 and 890px), or slim the bar below 900px (hide the
"Ink ↓" label, reduce `.nav` gap). Independently add `flex-shrink:0` to
`.swatch` so the circles can never ovalize.

### 3. `.ph` fade on whole photo blocks washes out the middle of the page
Lines 323/328/333/338/343 (`class="photo ph"`) and 357–360 (`class="pic ph"`)
put the 42% opacity fade (line 62) on the **entire ink block**, not just the
bracket label. A ~988px-tall region (sections 03–04 at 1440px) renders washed
out on every palette — forest polaroids fade to sage, red badges to mid-gray.
This is the single biggest reason the page reads "faded mockup" instead of
"printed flyer", and it is what the club will see in the demo.

**Fix:** move the fade to the label only:
`<div class="photo"><span class="ph">[ photo ]</span></div>` (same for
`.pic`). Blocks print at full ink strength; the bracket text stays faded.
Caveat: after the swap, paper-colored label text at 42% over full-strength
ink is very low contrast — restyle the label rather than keeping paper color
blindly. Img-swap workflow is unchanged.

### 4. Hero "Weekly meetups" chip is unreadable on the red palette (1.07:1)
`.tag.fill` (line 99) pairs `background:var(--ink-2)` with
`color:var(--text)`. Computed ratios: red **1.07:1** (a featureless black
rectangle — screenshot confirmed), midnight **1.56:1** (hard AA fail),
classic/forest/plum fine (5.9–8.6:1). Usage at line 256.

**Fix — use only this variant:** add a per-palette `--on-ink2` custom
property (`--paper` for red 14.45:1 and midnight 10.26:1; `--text` for the
three light palettes) and set `.tag.fill{color:var(--on-ink2)}`. A blanket
`color:var(--paper)` would break the three currently-good palettes (verified:
drops them to 1.8–2.6:1). The same variable also fixes badge initials (item
8).

---

## P1 — Contrast and accessibility (the palettes were never audited)

All ratios computed with the WCAG relative-luminance formula; the script the
reviewers used is reproducible in ~20 lines of python3.

### 5. `--ink-2` as text fails AA in 3 of 5 palettes, including the default
`.doing .n` (line 118) and `.stub h3` (line 127) color small text with
`--ink-2`: forest (the default) **1.84/1.98**, classic 2.60/2.81, plum
2.24/2.43 — all need 4.5:1. Red and midnight pass (their ink-2 is dark/light
enough). Do **not** blanket-switch to `--ink` — that regresses red (3.74:1).
**Fix:** add a per-palette text-safe accent variable (e.g. `--ink2-text`:
darkened for forest/classic/plum, equal to `--ink-2` for red/midnight) and
use it in these two rules only. Reserve `--ink-2` for backgrounds/graphics.

### 6. Red palette: all small ink-on-paper text fails AA at 3.74:1
Red's `--ink #E2342B` on paper is 3.74:1 — every small mono/UI text role
fails (kickers, eyebrow, CTAs, footer links; lines 92, 110, 131, 137, 156,
158, 164, 173). Large display text passes 3:1. Severity medium (red is
opt-in, forest is default) but the switcher is the club's voting tool, so red
could become default. **Fix:** palette-local split in the
`html[data-palette="red"]` block only — keep `#E2342B` for display/graphics,
add a darker red (#C1272B measures 4.96:1) for small-text roles. Do not
globally darken `--ink`; it also paints the join background and halftone
layers.

### 7. Join-section opacity fades sink text below AA in forest/plum
`.join .kicker{opacity:.8}` (line 156) → 3.53:1 forest / 4.40:1 plum;
`.join .sub{opacity:.92}` (line 158) → 4.14:1 forest. The default palette's
join blurb — real shipped copy — fails AA. Forest's full-opacity baseline is
4.57:1 with zero headroom, so any fade sinks it. **Fix:** delete both
opacity declarations; hierarchy is already carried by the mono kicker style
and smaller sub size.

### 8. Badge "initials" style is latent-failing (paper on ink-2)
`.badge .photo` (line 138) uses `--paper` on `--ink-2`: forest 1.84:1 (the
default = worst case), classic 2.60, plum 2.24. Latent until real initials
replace placeholders. **Fix:** rides on the `--on-ink2` variable from item 4.

### 9. Closed mobile nav is still keyboard-focusable off-screen (≤760px)
The nav hides via `transform:translateY(-130%)` only (line 176) — transforms
remove nothing from tab order or the accessibility tree. Verified: all 5
links focusable while fully off-screen (`position:fixed`, so the browser
can't scroll them into view — the focus ring vanishes for 5 tab stops).
Violates WCAG 2.4.3/2.4.7 and the CLAUDE.md quality floor ("visible keyboard
focus"). **Fix:** add `visibility:hidden` to the closed state,
`visibility:visible` on `.nav.open`, with `transition:transform .3s,
visibility .3s` (visibility flips at end-of-animation, slide preserved, zero
JS change). Note: under `prefers-reduced-motion` the flip is instant — fine,
just don't rely on the delay for correctness.

### 10. Menu button has no `aria-expanded`/`aria-controls`; Escape doesn't close
Line 238 has only `aria-label="Menu"`; the JS (422–425) toggles a class and
nothing else. Missing `aria-expanded` is a WCAG 4.1.2 failure;
Escape-to-close is APG best practice. **Fix:** ~4 lines — set
`aria-controls="nav"` + `aria-expanded` flipped in the click handler; one
keydown listener closes on Escape and restores focus to the button.

### 11. Form success/error is silent to screen readers, and success drops focus
No live region anywhere (lines 393, 395); the handler (436–444) hides the
form while the Submit button holds focus → focus falls to `<body>`.
**Fix (corrected during verification):** `role="status"` on `#note` handles
the error path with no JS change. For success, `role="status"` on `#ok` is
NOT reliable (unhiding a hidden live region is ignored by many screen
readers) — instead give `#ok` `tabindex="-1"` and call `ok.focus()` after
revealing it, which fixes announcement and focus loss together. Error path is
dead code until an endpoint is set.

### 12. Placeholder fade (.42) makes bracket labels near-invisible
`.ph` at opacity .42 (line 62) lands body placeholders at ~2.7:1 and the
big `.when.ph` at 1.81:1 (forest default) — below even the 3:1 large-text
bar. **Fix:** raise `.ph` to ~.6 for body placeholders (clears 4.5:1), but
`.when.ph` needs ~.75 to pass 3:1 — either raise `.ph` to ~.75 overall or
give `.when.ph` its own rule. Re-run the contrast check after.

### 13. Swatch buttons: title-only names, cramped mobile hit targets
Lines 233–237 rely on `title` alone (works, but inconsistently surfaced;
invisible to touch/keyboard users). Desktop spacing passes 2.5.8; mobile
(18px + 5px gap = 23px center spacing, lines 181–182) fails the 24px
minimum. **Fix:** add `aria-label` duplicating each title (5 attributes);
on mobile grow the hit area (padding/`::before inset:-4px`) AND bump `.inks`
gap to ≥6px — verified this still fits the pass-2 masthead budget (≥324px
viewports).

---

## P2 — Go-live path for the signup form

**Answer to "can we just use a plugin so I don't code it": yes.** The handler
(lines 427–444) already POSTs `FormData` to a `FORM_ENDPOINT` constant;
demo mode is just `FORM_ENDPOINT = ""`. Options below were researched and
fact-checked against official pricing/docs, July 2026.

| Backend | Free tier | Keeps riso-styled form? | Where signups land |
|---|---|---|---|
| **Google Forms** | Unlimited, forever | Yes via unofficial `formResponse` POST; No via official iframe | Linked Google Sheet, whole e-board |
| **Web3Forms** | 250/mo, 30-day dashboard | **Yes — works with existing handler + one hidden `access_key` input** | Email (route to shared club Gmail) |
| Tally | Unlimited (fair use), Sheets sync | No — embed/iframe only | Tally dashboard + Sheet |
| Formspree | 50/mo (not 25 — CLAUDE.md is stale), 30-day history | Yes, endpoint drop-in | Dashboard + ≤2 emails |
| Forminit (ex-Getform) | 100/mo | Yes | Email/dashboard |
| Basin | 50/mo, 0 collaborators | Yes | Dashboard (single owner) |

**Recommendation:** Google Forms backed by a Sheet — the only option with no
cap, no retention limit, and a permanent roster in the Google ecosystem the
e-board already shares. Two wiring modes: (a) keep the styled form via the
unofficial `docs.google.com/forms/d/e/FORM_ID/formResponse` POST — rename
inputs to the form's `entry.NNNN` IDs and switch the fetch to
`mode:'no-cors'` treating submit as success (Google sends no CORS headers, so
the current `res.ok` check can never pass); entry IDs silently change if the
Form is edited, so test after any edit. (b) If that feels too brittle:
official iframe embed (bulletproof, loses the riso styling). Clean runner-up
if the styled form is non-negotiable and unofficial hacks are not:
**Web3Forms** (250/mo, existing handler works unmodified).

### 14. The in-file Tally instructions are wrong and must be fixed regardless
The comment at lines 428–429 (and `CLAUDE.md` line ~128) says to set
`FORM_ENDPOINT` to `https://tally.so/r/XXXX`. Tally has **no** endpoint for
external HTML form POSTs (confirmed against its official developer docs —
embed/webhooks/management API only). Following the instructions produces a
CORS-blocked fetch → every submitter sees "Network error. Try again." — a
dead signup form at go-live. **Fix:** reword the comment to name only
Formspree-style AJAX endpoints for `FORM_ENDPOINT`, and state that
Tally/Google Forms require the embed route.

### 15. Go-live hardening notes (do when wiring the endpoint, not before)
- Disable the submit button before the fetch, re-enable on error paths
  (currently double-submittable mid-flight, lines 436–444).
- Spam: a client-side honeypot only stops naive bots; if Formspree, use its
  server-filtered `_gotcha` field; Google Form embed needs neither.
- `#note` hardcodes "Demo mode..." (line 393) — will show to real visitors
  when live. Set it from JS based on `FORM_ENDPOINT` truthiness; when set,
  leave empty unless the user supplies wording (no invented copy).

---

## P3 — `<head>`, deploy, and print

### 16. Repo slug is misspelled: the live URL is `/cdsoffine/` (missing "l")
Verified: `git remote -v` → `PlatinumPawn/cdsoffine.git`; the site is live
(HTTP 200) at `https://platinumpawn.github.io/cdsoffine/`. GitHub Pages does
**not** redirect after a repo rename, so fix it before the URL circulates or
gets baked into og:url/canonical/QR codes/flyers. **Fix:** rename the repo to
`cdsoffline` in GitHub Settings, then `git remote set-url origin ...`. Do
this before adding any URL metadata.

### 17. Head is missing description, OG/Twitter tags, favicon, theme-color
The entire `<head>` is charset/viewport/title/fonts (lines 3–9). A pasted
link in iMessage/GroupMe shows a title-only card. Addable now with zero
invented copy: `og:title`, `og:description` (reuse the lede at line 254
verbatim), `og:type`, `twitter:card`, `<meta name="description">` (reuse
lede/mission), title → `CDS Offline · Boston University` (matches footer
separator). Favicon: inline SVG data-URI of the outlet mark works for
Chrome/Firefox/Edge — note the mark is stroke-based, so hardcode the stroke
color, and Safari needs a PNG fallback. `theme-color: #F0EEE4`. Blocked until
later: `og:url`/canonical (needs the post-rename URL) and `og:image` (needs a
real ~1200×630 raster committed to the repo — scrapers ignore SVG/data URIs;
the repo currently has no raster logo at all, so exporting one also recovers
the asset lost in log.md pass 4). `apple-touch-icon` likewise needs a real
180×180 PNG.

### 18. Decide: demo is publicly indexable right now
The placeholder-filled demo is live and has no robots meta. Either add
`<meta name="robots" content="noindex">` until launch, or consciously accept
early indexing. One line either way.

### 19. Printing is broken (ironic for a flyer aesthetic)
No `@media print` block exists. Verified in print PDFs: Letter width (~734px)
triggers the ≤760px media query, so the `position:fixed` mobile nav paints
over content on every printed page; knockout (paper-on-ink) text vanishes
with backgrounds off; midnight prints near-blank. **Fix (~10–15 lines):**
`@media print` — force the light default palette variables, override
knockouts to `color:var(--text)`/transparent background, hide
`body::before/::after`, `.inks`, `.menu-btn`, **and `.nav`**.

### 20. `.DS_Store` is one `git add .` away from being published
`.gitignore` contains only `log.md`. Append `.DS_Store`.

---

## P4 — Polish (low severity, all verified real)

- **21. Empty `<a id="top">` + no scroll-padding** (lines 243, 32): all six
  in-page links land content ~62px under the sticky header's shadow zone and
  "back to top" stops 62px short. Fix: delete the empty anchor (browsers
  scroll `#top` to y=0 natively when no such element exists) and add
  `scroll-padding-top:~70px` to the `html` rule.
- **22. E-board grid orphans the 5th badge** below 940px: 3/2 at the 3-col
  tier, jarring 2/2/1 at ≤520px (line 195). Fix: last-child span rule or
  single column below 520 — but note a spanning badge's `aspect-ratio:1`
  photo (line 138) balloons to ~452px, so pair the grid rule with a photo
  size adjustment, or prefer the single-column option.
- **23. Board column jump at 520/521px**: cards invert 227px → 146px from 1px
  of width (lines 194–195), and the `.top` strip has ~6px slack at 521px.
  Mainly affects resized desktop windows. Fix: move the 2-up breakpoint to
  ~640px, or `repeat(auto-fit,minmax(160px,1fr))` for the mid-range.
- **24. Photo slots aren't `<img>`-ready**: no `object-fit` anywhere; a
  swapped-in img distorts/inherits wrong size and kills the badge halftone
  `::after`. Fix: add `.photo img,.pic img{width:100%;height:100%;object-fit:cover}`
  and reword the comments at lines 320/356 to "put an `<img>` inside this div
  and delete the `[ photo ]` text and `.ph` class".
- **25. Swatch `aria-pressed` init is hardcoded** (line 234 `forest`): desyncs
  if the default palette is changed per CLAUDE.md's documented procedure.
  Fix: derive initial state from `root.getAttribute('data-palette')` in the
  existing script. (Palette non-persistence across reloads is intentional —
  voting tool — leave it.)
- **26. CSS cruft**: `.menu-btn` declared twice in the same 760px media query
  (lines 179 and 183) — merge. Do NOT delete `font-weight:inherit` from
  `.ph` (line 62) — it is load-bearing (de-bolds `h3.ph` at line 302).

---

## Docs drift — update `CLAUDE.md` (5 stale counts)

1. Default palette is `forest`, not `classic`.
2. E-board has **5** badge cards, not 4.
3. "All copy is blanked" is no longer true — hero/about/join/footer copy was
   filled in log.md pass 1–2 (headings incl. "Meet the board.", line 318).
4. Working rule 1 ("never invent copy") was explicitly overridden by the user
   (log.md pass 1): tone copy may be drafted; **facts** (dates, names,
   addresses, emails, captions) stay bracketed.
5. The Tally `FORM_ENDPOINT` advice is wrong (see item 14); also Formspree's
   free cap is 50/mo now, not ~25.

---

## Content still needed from the user (facts — never invent)

- About activities `d/` and `e/` (or approval to trim the list to 3).
- Next meeting: day/date/time, location, address, sub-heading, prep list.
- E-board: 5 names + photos (roles are set).
- Past events: 4 photos + captions.
- Footer email (link currently `href="#"`).
- For metadata: final URL (after repo rename) and a raster logo export.

## Review provenance

37 findings confirmed, 0 rejected, across visual / responsive / a11y / code /
meta dimensions; each was independently re-verified (measurements reproduced,
ratios recomputed, fixes tested on scratch copies before being recommended
here). Structural verdict: the zine system, section rhythm, and responsive
skeleton are sound — no horizontal overflow at any width from 500–1440px, and
the pass-2/3/4 fixes in log.md all held. The perceived UI weakness traces
mostly to items 1–4 above, which are small, verified fixes.
