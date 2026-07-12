<!--
  FEATURE FILE TEMPLATE
  =====================
  Copy to `Features/FEATURE-{kebab-name}.md` and fill it in.

  This is a design scratchpad, not a spec. It is AI-drafted from your intent —
  describe what you want, have the agent read the relevant service specs, and let
  it draft this file against them; then you refine. Settled decisions cascade
  into the service specs (WORKFLOW.md step 2); this file is archived to
  Features/Implemented/ after the feature ships.

  Delete this comment block before committing.
-->

# FEATURE: {Feature Name}

**Status:** {Draft | In progress | Shipped}
**Affected services:** {list, e.g. main-api, admin-frontend}
**Not affected:** {services you considered and ruled out — useful to record}

---

## 1. Intent (what the author asked for)

> One short paragraph in the user's own words: what they want and why.

## 2. Decisions

- The handful of design choices that matter, each with its *why*.
- Call out the one or two cross-cutting constraints that ripple across services
  (the things easiest to get wrong).

## 3. Per-service impact

### {service-a}
- Modules / schema / endpoints / slices / views this feature adds or changes.

### {service-b}
- …

## 4. Open questions

- Anything unresolved. Mirror blocking ones into the overview's Open Questions
  log. **Prompt generation (step 3) refuses to run while a blocking question
  remains open** — resolve or explicitly defer each one.

## 5. Out of scope (v1)

- What you are deliberately *not* doing yet, so reviewers don't expect it.

## 6. Implementation order

- One line declaring the cross-service dependency order, e.g.
  `main-api → admin-frontend + storefront-frontend`. Every generated prompt
  inherits it (as its `Prerequisites:` header line), so sequencing is never
  implicit. Single-service features: just name the service.

## 7. Recommended model (per service)

- **{service}:** {tier} — {one-line reason}.
- Decide the model tier at design time, not session time: the cheapest tier
  that fits (a fully-specified CRUD task doesn't need a flagship model);
  reserve the top tier for genuinely complex work, usually intricate UI. Each
  generated prompt carries its line in the dispatch header.
