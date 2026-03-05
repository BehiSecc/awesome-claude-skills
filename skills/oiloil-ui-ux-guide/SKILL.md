---
name: oiloil-ui-ux-guide
description: Modern, clean UI/UX guidance + review skill. Use when you need actionable UX/UI recommendations, design principles, or a design review checklist for new features or existing systems (web/app). Focus on CRAP (Contrast/Repetition/Alignment/Proximity) plus task-first UX, information architecture, feedback & system status, consistency, affordances, error prevention/recovery, and cognitive load. Enforce a modern minimal style (clean, spacious, typography-led), reduce unnecessary copy, forbid emoji as icons, and recommend intuitive refined icons from a consistent icon set.
---

# OilOil UI/UX Guide (Modern Minimal)

Use this skill in two modes:

- `guide`: Provide compact principles and concrete do/don't rules for modern clean UI/UX.
- `review`: Review an existing UI (screenshot / mock / HTML / PR) and output prioritized, actionable fixes.

Keep outputs concise. Prefer bullets, not long paragraphs.

## Workflow (pick one)

### 1) `guide` workflow
1. Identify the surface: marketing page / dashboard / settings / creation flow / list-detail / form.
2. Identify the primary user task and primary CTA.
3. Apply the system-level guiding principles first (mental model and interaction logic).
4. Then apply the core principles below (start from UX, then refine with CRAP).
5. If icons are involved: apply `references/icons.md`.

### 2) `review` workflow
1. State assumptions (platform, target user, primary task).
2. List findings as `P0/P1/P2` (blocker / important / polish) with short evidence.
3. For each major issue, label the diagnosis: execution vs evaluation gulf; slip vs mistake (see `references/design-psych.md`).
4. Propose fixes that are implementable (layout, hierarchy, components, copy, states).
5. End with a short checklist to verify changes.

Use `references/review-template.md` when you need a stable output format.

## Non-negotiables (hard rules)
- No emoji used as icons (or as UI decoration). If an emoji appears, replace it with a proper icon.
- Icons must be intuitive and refined. Use a single consistent icon set for the product (avoid mixing styles).
- Minimize copy by default. Add explanatory text only when it prevents errors, reduces ambiguity, or improves trust.

## System-Level Guiding Principles

Apply these as first-order constraints before choosing components or page patterns.
Full definitions and review questions: `references/system-principles.md`.

Key principles: concept constancy · primary task focus · UI copy source discipline · state perceptibility · help text layering (L0–L3) · feedback loop closure · prevention + recoverability · progressive complexity · action perceptibility · cognitive load budget · evolution with semantic continuity.

## Core Principles (minimal set)

### A) Task-first UX
- Make the primary task obvious in <3 seconds.
- Allow exactly one primary CTA per screen/section.
- Optimize the happy path; hide advanced controls behind progressive disclosure.

### B) Information architecture (grouping & findability)
- Group by user mental model (goal/object/time/status), not by backend fields.
- Use clear section titles; keep navigation patterns stable across similar screens.
- When item count grows: add search/filter/sort early, not late.

### C) Feedback & system status
- Cover all states: loading, empty, error, success, permission. Details in `references/checklists.md`.
- After any action, answer: "did it work?" + "what changed?" + "what can I do next?"
- Prefer inline, contextual feedback over global toasts (except for cross-page actions).

### D) Consistency & predictability
- Same interaction = same component + same wording + same placement.
- Use a small, stable set of component variants; avoid one-off styles.

### E) Affordance + Signifiers (make actions obvious)
- Clickable things must look clickable (button/link styling + hover/focus + cursor). On web, custom clickable elements need `cursor: pointer` and focus styles.
- Primary actions need a label; icon-only is reserved for universally-known actions.
- Show constraints before submit (format, units, required), not only after errors.
- For deeper theory (affordances, signifiers, mapping, constraints): see `references/design-psych.md`.

### F) Error prevention & recovery
- Prevent errors with constraints, defaults, and inline validation.
- Make destructive actions reversible when possible; otherwise require deliberate confirmation.
- Error messages must be actionable (what happened + how to fix).

### G) Cognitive load control
- Reduce choices: sensible defaults, presets, and progressive disclosure.
- Break long tasks into steps only when it reduces thinking (not just to look "enterprise").
- Keep visual noise low: fewer borders, fewer colors, fewer competing highlights.

### H) CRAP (visual hierarchy & layout)
- Contrast: emphasize the few things that matter (CTA, current state, key numbers).
- Repetition: tokens/components/spacing follow a scale; avoid “almost the same” styles.
- Alignment: align to a clear grid; fix 2px drift; align baselines where text matters.
- Proximity: tight within a group, loose between groups; spacing is the primary grouping tool.

## Spacing & layout discipline (compact rule set)

Use this when implementing or reviewing layouts. Keep it short, but enforce it strictly.

- Rule 1 - One spacing scale:
  - Base unit: 4px.
  - Allowed spacing set (recommended): 4 / 8 / 12 / 16 / 24 / 32 / 40 / 48.
  - New gaps/padding should use this set; off-scale values need a clear reason.
- Rule 2 - Repetition first:
  - Same component type keeps the same internal spacing (cards, list rows, form groups, section blocks).
  - Components with the same visual role should not have different spacing patterns.
- Rule 3 - Alignment + grouping:
  - Align to one grid and fix 1-2px drift.
  - Tight spacing within a group, looser spacing between groups.
- Rule 4 - No decorative nesting:
  - Extra wrappers must add real function (grouping, state, scroll, affordance).
  - If a wrapper only adds border/background, remove it and group with spacing instead.
- Quick review pass:
  - Any off-scale spacing values?
  - Any baseline/edge misalignment?
  - Any wrapper layer removable without losing meaning?

## Modern minimal style guidance (taste with rules)
- Use whitespace + typography to create hierarchy; avoid decoration-first design.
- Prefer subtle surfaces (light elevation, low-contrast borders). Avoid heavy shadows.
- Keep color palette small; use one accent color for primary actions and key states.
- Copy: short, direct labels; add helper text only when it reduces mistakes or increases trust.

## Motion (animation) guidance (content/creator-friendly, not flashy)
- Motion explains **hierarchy** (what is a layer/panel) and **state change** (what just happened). Avoid motion as decoration.
- Default motion vocabulary: fade; then small translate+fade; allow tiny scale+fade for overlays. Avoid big bouncy motion.
- Keep the canvas/content area stable. Panels/overlays can move; the work surface should not “float.”
- Prefer consistency over variety: same component type uses the same motion pattern.
- Avoid layout jumps. Use placeholders/skeletons to keep layout stable while loading.

## Anti-AI Self-Check (run after generating UI)

Before finalizing any generated UI, verify these items. Violating any one is a mandatory fix.

- **Gradient restraint** — Gradients must convey meaning (progress, depth, state distinction). Purely decorative gradients: at most one per page. If background, buttons, and borders all use gradients simultaneously, that is overuse — pick one and flatten the rest.
- **No emoji as UI** — Already a non-negotiable. Re-check: no emoji slipped in as section icons, status indicators, or button labels.
- **Copy necessity** — For every piece of text, ask: if I remove this, can the user still understand through layout, icons, and position alone? If yes, remove it. Text is the last resort, not the default.
- **Decoration justification** — Every purely visual effect (blur, glow, animated entrance, layered shadows) must answer: "what does this help the user understand?" No answer → remove.

## References
- System-level guiding principles (concept constancy, copy discipline, state perceptibility, etc.): `references/system-principles.md`
- Interaction psychology (Fitts/Hick/Miller, cognitive biases, flow, attention): `references/interaction-psychology.md`
- Design psychology (affordances, signifiers, mapping, constraints, gulfs, slips vs mistakes): `references/design-psych.md`
- Icon rules and "intuitive refined" guidance: `references/icons.md`
- Review output template and scoring: `references/review-template.md`
- Expanded checklists (states, affordance, lists, forms, settings, motion, dashboards, copy): `references/checklists.md`
