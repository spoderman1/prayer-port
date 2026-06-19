# Landing Page Build Spec
**Project:** Salah-Centered Desktop Companion (working name: Lantern / NoorPath — unresolved, see §0)
**This document is for:** Claude Code, as the implementation brief for the next build pass
**Supersedes:** the first-draft `index.html` static page. Read alongside `salah-companion-prd.md` and `figma-design-vision-action-plan.md`, which remain the source of design and product truth — this spec translates their constraints into a build plan, it doesn't override them.

---

## 0. Scope note — read this before building

Everything in this spec is landing-page scope (Validation Phase 2: fake-door test, email capture, founding pre-order). It is explicitly **not** the PRD's MVP build (Phase 3, Tauri app).

One section of this spec — the **plan-your-day modal (§5)** — is a deliberate scope amendment to the PRD, made by the founder during landing-page planning. PRD §4 item 4 specifies a single one-line intention *prompt* after a prayer completes as the P0 differentiator; it does not specify a drag-and-drop scheduling interface. The founder has decided the drag-and-drop planner should ship as a real, working v1 feature rather than a landing-page-only concept teaser. Flagging this explicitly so it's a recorded decision, not a silent scope drift — per PRD Risk #6 ("building before validating") and the general principle that scope changes should be visible, not accidental. If this spec is picked up out of context later, the drag-and-drop planner should be treated as confirmed product scope, not as a stray idea — but if anyone revisits it, it's worth re-checking against the PRD's "no feature should outrun the core promise" test.

Practically, this means: build the modal in §5 as production-quality interactive code (not a static mock), because it is previewing a feature the app will actually have.

---

## 1. What's changing from the first draft

| Element | First draft | This spec |
|---|---|---|
| Hero art | Inline placeholder SVG, static | Same placeholder SVG, but now also the anchor for the scroll-linked sun (§3) |
| Sun | Decorative, fixed inside hero SVG | Promoted to a persistent, page-level element that travels across the top of the entire page as the user scrolls (§3) |
| Sections | Problem → Loop → Intention → Founding | Same backbone, content unchanged, but every section now needs a defined "sun sky-band" hand-off point (§3.2) |
| Modal | None | New: "Plan your day" interactive demo — drag-and-drop prayer scheduling + per-block visual styling (§5) |
| File structure | Single HTML file | Still single HTML file by default (keeps hosting trivial) but spec'd so Claude Code can split into `/src` if it judges the modal logic too large for one file — see §7 |

---

## 2. Design tokens (unchanged — carry forward exactly)

Reuse the token set from the first draft, which was itself derived from `figma-design-vision-action-plan.md` §4 and §6. Do not invent new colors. Reproduced here for convenience:

```css
--asr-sky-top:#E5CD96; --asr-sky-mid:#E9B96B; --asr-sky-low:#DCA85C; --asr-ground:#B98E5C;
--maghrib-top:#5E4C6E; --maghrib-mid:#C4694A; --maghrib-low:#E8A36C; --maghrib-ground:#6E4A3E;
--isha-top:#1F2438; --isha-mid:#2E3450; --isha-low:#3C4566; --isha-ground:#23283C;
--fajr-top:#343B5C; --fajr-mid:#67769F; --fajr-low:#C7A3A6; --fajr-ground:#4A4458;
--dhuhr-top:#CFE0E2; --dhuhr-mid:#EFE8D2; --dhuhr-low:#F6EFDC; --dhuhr-ground:#D8C49A;
--lantern:#E8B85C; --sun:#F6E2B4;

--paper:#F6EFDC; --paper-deep:#EFE8D2; --clay:#C9A77D; --clay-line:#B89572;
--ink:#3A3228; --ink-soft:#6B5E4D;
--teal:#1E5F63; --lapis:#28407A; --madder:#A8442E; --gold:#C99A3A;

--serif: "Fraunces", Georgia, serif;     /* block names, intention line — sparingly */
--sans: "Hanken Grotesk", system-ui, sans-serif;  /* everything else */
```

Note the first draft only used four of the five light palettes (Fajr was implied, not built as CSS variables). This spec needs all five, since the sun now traverses the full day across the whole scroll (§3).

---

## 3. The scroll-linked sun

### 3.1 Concept

The sun is not part of any one section's artwork. It is a **fixed-position element pinned near the top of the viewport**, persistent across the entire page, that moves horizontally (and slightly vertically, on an arc) as the user scrolls — simulating its real path across the sky from Fajr's first light to Isha's dark. It is the one continuous thread tying every section together, visually enacting the product's own thesis: *the light is the day*.

This is the page's signature element (per the frontend-design skill's call for one memorable, deliberate risk). Spend the page's "boldness budget" here; keep everything else disciplined and quiet.

### 3.2 Mechanics

- **Position:** `position: fixed`, top of viewport, `z-index` above section backgrounds but below modal/nav. Implemented as a small SVG or div (a soft-edged radial-gradient disc, matching `--sun` / `--lantern` color depending on time-of-day band — see below).
- **Horizontal path:** maps scroll progress (0–100% of total page scroll) to an arc from left edge to right edge — a shallow rise-and-fall arc (like a real sun's elevation), not a straight line. Use a parametric arc: `x = progress * viewportWidth`, `y = peakHeight - (sin(progress * PI) * arcDepth)`. The sun should be lowest (near horizon) at the very top and very bottom of the page, and highest around the Dhuhr section.
- **Color/glow hand-off:** the sun's fill and glow color crossfade through the five palettes as it crosses each section's boundary — `--sun` (pale, Dhuhr-bright) at midday sections, warming to `--lantern` (amber) by Asr, then dimming/reddening through Maghrib, and finally **the sun should visually become the lantern** — by the Isha section it has transformed from a sun-disc into the small warm lantern glow already used in the founding-offer section. This is a deliberate continuity: same element, same emotional anchor, different name for the same warmth. Don't build them as two unrelated assets that swap; cross-fade/morph one element's color, size, and glow-radius.
- **Implementation approach:** use a single `requestAnimationFrame`-driven scroll listener (not CSS `scroll-timeline` alone, for broader browser support — though `animation-timeline: scroll()` is acceptable as a progressive enhancement if Claude Code prefers, with the JS approach as fallback). Read `window.scrollY / (document.body.scrollHeight - window.innerHeight)` for progress; throttle with `requestAnimationFrame`, not on every scroll event.
- **Reduced motion:** if `prefers-reduced-motion: reduce`, the sun should still indicate position (e.g. snap to each section's correct point without the smooth animated arc) rather than being removed entirely — it's informational (which light-band you're in), not just decorative.
- **Section sky-bands:** each full-bleed section's background should itself shift through the five palettes in the same order the sun visits them, so the sun's position and the page's background light stay in sync. Order top to bottom: Fajr (hero, optional — could start at Dhuhr/Asr if hero stays Asr per first draft) → Dhuhr → Asr (hero, if hero stays as-is) → Maghrib → Isha. **Decision needed:** keep the hero as Asr (matches first draft's signature long-shadow moment and PRD's "Asr is the hero shot" instruction in the vision doc §7) and treat the page's scroll narrative as Asr → Maghrib → Isha, i.e. afternoon into evening, rather than a full Fajr-to-Isha cycle. This is simpler, matches existing copy, and avoids the sun having to "rewind" if the hero is mid-day. Claude Code should implement this simpler Asr→Maghrib→Isha version unless told otherwise.

### 3.3 Constraints carried from the avoid-list (vision doc §3)

- The sun is a plain disc/glow. No face, no character, no "happy sun" cartoon styling.
- No lens-flare, no photorealistic rendering — flat, soft-edged, miniature-painting consistent with the rest of the art direction.

---

## 4. Section-by-section structure (carry forward content, rebuild around the sun)

Keep the first draft's content and copy. Re-confirm structure here for Claude Code's build order:

1. **Masthead** — wordmark (placeholder brand name, see §0 header), tagline.
2. **Hero** — Asr panorama placeholder SVG (slot for commissioned art later, per vision doc §6 layer stack), product promise headline, sub-copy, CTA to founding offer.
3. **The problem** — "your calendar is the skeleton of your day" section, Maghrib-toned background.
4. **The loop** — four beats (Arrives / Commit / Protect / Return), dark Isha-leaning background.
5. **The intention differentiator** — paper-toned section, intention card mock + day-view five-light strip. **This section is being replaced/extended by §5's interactive modal** — see below.
6. **Founding offer** — price, platform fake-door buttons, email capture, pre-order CTA, Isha/lantern background. (Payment link wiring is out of scope for this spec — see §6.)
7. **Footer** — promises, fine print.

Each section needs a `data-light-band="fajr|dhuhr|asr|maghrib|isha"` attribute on its root element so the sun-tracking script can read section boundaries via `getBoundingClientRect()` and know which palette to crossfade toward.

---

## 5. New feature: "Plan your day" modal

### 5.1 What it is

A modal, launched from a button inside the intention/differentiator section (replacing or sitting alongside the static intention-card mock from the first draft), that lets a visitor **interactively try** the day-planning concept: drag each of the five daily prayers onto a visual timeline, drop them into a chosen time slot within their valid window, and pick a visual "block style" (a quick-pick tag, mirroring the PRD's intention quick-picks: work / family / rest / Quran / errands) for the work-block that follows.

This is a **demo/preview**, not a real scheduling backend — there is no account, no persistence beyond the browser session (per PRD §4 item 8, local-only, no accounts — the landing page should not contradict this principle even in miniature). On page refresh it resets. This should be stated lightly in the modal's own copy (e.g. a small note: "This is a preview — refresh to reset") so it's honest about what it is.

### 5.2 Interaction spec

- **Trigger:** a button/link in the intention section, e.g. "Try planning a day →".
- **Modal layout:** warm-paper surface (per vision doc §6 UI surfaces: cream, soft borders, grain texture, generous radius — consistent with the existing `.intention-card` styling in the first draft). Contains:
  - A horizontal timeline strip spanning the five prayer windows (Fajr, Dhuhr, Asr, Maghrib, Isha), visually similar to the `.dayview` strip already built in the first draft, but now interactive.
  - Five small draggable "prayer chips" (one per prayer name), initially sitting in a tray above or beside the timeline, undocked.
  - The user drags a chip onto the timeline; it should snap to the correct prayer's window segment only (a chip for Asr can only be dropped within the Asr segment — this enforces the PRD's core constraint that planning happens **within** the valid window, never outside it. Reject/snap-back drops outside the correct segment, with a small inline message like "Pick a moment within Asr's window" rather than a harsh error).
  - Within a segment, the drop position along the segment's width maps to a specific time within that window (e.g. left edge = window start, right edge = window end). Show a small time label that updates live while dragging (e.g. "3:40 PM").
  - Once a prayer chip is placed, the segment that follows it (i.e. the work-block until the next prayer) becomes clickable, revealing a small inline quick-pick row (work / family / rest / Quran / errands — same vocabulary as PRD §4 item 4) to tag what that block is for. Selecting one visually re-colors or re-labels that block segment (subtle — a small icon or accent border, not a jarring change, consistent with vision doc's "no confetti energy" principle for the real app's settled state).
- **Completion state:** once all five prayers are placed, show a brief, quiet affirming state (text only, e.g. "That's your day, shaped around five arrivals.") — no celebratory animation, no score, no streak-like mechanic. This must stay consistent with PRD §4 item 5 and vision doc §5's "contentment, not celebration" rule even though this is just a marketing demo — getting the tone right here is itself a way of demonstrating the product's ethos to a prospective user.
- **Accessibility:** drag-and-drop must have a keyboard-operable equivalent (e.g. focus a chip, press Enter/Space to pick it up, arrow keys to move along the segment, Enter again to drop) — do not ship a mouse-only interaction. Announce drop results via `aria-live` region for screen readers ("Asr planned for 3:40 PM").
- **Mobile/touch:** drag-and-drop must work with touch events, not just mouse events. If implementing from scratch, use pointer events (`pointerdown`/`pointermove`/`pointerup`) rather than separate mouse/touch handlers, since pointer events unify both.

### 5.3 Visual consistency rules

- Use the same five-light palette per segment as the existing `.dayview` strip (Fajr/Dhuhr/Asr/Maghrib/Isha colors from §2 tokens).
- Prayer chips: small, warm-paper pills with the prayer name in the sans typeface — no icons that anthropomorphize (no little praying-figure icons; per vision doc, the mat itself is the only "character" and even it has no face).
- Quick-pick tags: reuse the `.chip` styling already established in the first draft's intention card.
- The modal must work at the same visual quality bar as the rest of the page — this is a feature preview, not a wireframe, since the founder has chosen to position it as real v1 scope (§0).

### 5.4 What this section is explicitly not

- Not a real calendar integration (PRD Non-Goals explicitly excludes calendar integration — this demo must not imply the real app reads or writes to Google Calendar, Outlook, etc.)
- Not collecting or transmitting any data entered in the demo — it's purely client-side state for the duration of the page view.
- Not a religious timing tool — the segment boundaries are illustrative prayer windows for demo purposes; the modal's copy should avoid any language that could read as a ruling on prayer timing (consistent with PRD §5 Non-Goals and the existing footer disclaimer).

---

## 6. Explicitly out of scope for this build pass

Carried forward from the previous conversation — do not build these now, just leave the existing placeholders/hooks in place:

- Real `FORM_ENDPOINT` integration (Tally/Formspree/etc.) — leave as a clearly marked token.
- Real `PAYMENT_LINK` — leave as a clearly marked token. Note for whoever wires this up later: the founder has not yet decided between (a) waitlist-only, (b) "reserve, charged at launch" via a Stripe delayed-charge flow, or (c) charge-now-with-refund-guarantee. Don't assume one — surface this as an open decision if asked to wire up payments.
- Final brand name — keep using a placeholder, swappable in one location.
- Commissioned hero/section artwork — keep the placeholder SVGs, built so real art can drop into the same layer slots later (vision doc §6 layer stack: sky → far dunes → mid dunes → caravan+mat → foreground → shadow).

---

## 7. Technical implementation notes for Claude Code

- **Default to a single HTML file** (inline `<style>` and `<script>`, as the first draft was built), since this is a static landing page that needs to be trivially hostable anywhere with no build step. Only split into multiple files (`/src/sun.js`, `/src/planner-modal.js`, etc.) if the combined file becomes unwieldy to maintain — use judgment, but bias toward staying single-file unless there's a clear maintainability reason not to.
- No frameworks needed or expected (no React/Vue) — vanilla JS is sufficient for both the scroll-sun and the drag-and-drop modal, and keeps the page lightweight, consistent with the PRD's general preference for a lightweight footprint (echoing the Tauri-over-Electron reasoning in PRD §10, even though that's about the eventual desktop app, not this page).
- Respect `prefers-reduced-motion` throughout (sun arc, modal transitions, chip drag animations).
- Respect keyboard focus visibility (`:focus-visible` outlines) throughout, including inside the new modal — this was already a quality bar in the first draft and must extend to all new interactive elements.
- The modal should trap focus while open and restore focus to the triggering button on close (standard modal accessibility pattern).
- Test the sun animation's performance — `requestAnimationFrame`-throttled scroll handling, not unthrottled `scroll` event listeners, to avoid jank.

---

## 8. Acceptance checklist

Before considering this build pass done, confirm:

- [ ] Sun is visible and tracking scroll position from page top to bottom, color/size morphing into the lantern by the Isha section
- [ ] Each section's background palette matches the sun's current light-band
- [ ] Reduced-motion users still get correct sun positioning, just without animated easing
- [ ] Plan-your-day modal: all five prayers draggable, each snaps only within its correct window segment
- [ ] Plan-your-day modal: keyboard-only operation works end to end
- [ ] Plan-your-day modal: quick-pick block tagging works after a prayer is placed
- [ ] No copy anywhere implies a religious ruling on timing (PRD §5 Non-Goals)
- [ ] No copy implies calendar integration, accounts, or cloud sync (PRD §5 Non-Goals)
- [ ] All four open tokens (`FORM_ENDPOINT`, `PAYMENT_LINK`, brand name, hero art) remain clearly marked placeholders
- [ ] Page still works as a single static HTML file with no build step
