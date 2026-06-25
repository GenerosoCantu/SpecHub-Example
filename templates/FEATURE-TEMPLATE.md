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

- Anything unresolved. Mirror blocking ones into the overview's Open Questions log.

## 5. Out of scope (v1)

- What you are deliberately *not* doing yet, so reviewers don't expect it.
