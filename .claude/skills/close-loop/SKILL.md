---
name: close-loop
description: Close out an implemented Plaza feature (WORKFLOW.md Step 5) — reconcile specs, flip STATUS.md, update CHANGELOG, archive feature file and prompts, sync repo instructions. Use when the user says a feature is implemented/verified and asks to "close the loop", "reconcile", "close out", or "step 5".
---

# Close the Loop (Step 5)

Reconcile the spec hub with what was actually built. All sub-steps happen in this single pass — the loop is closed only when the hub reflects reality exactly.

## 0. Preconditions

- The implementation has been **human-verified** (build run, endpoints/UI exercised, diff read). If not, stop.
- Every prompt for the feature has dispatch-header `Status: Verified`. If any is still `Generated` or `Applied`, stop and report which.

Ask the user (if not already stated): what deviated from the plan during implementation? Deviations are absorbed into the spec, not ignored.

## 1. Verify the spec against the code (drift check)

Do not reconcile from memory or from the user's description alone.

- For each affected service, resolve the service repo path from the prompt's dispatch-header `Target repo:` line.
- In that repo, read the implementation files the prompt named (schema, controller/routes, components) and confirm the spec's claimed contracts — field names, types, endpoint paths/guards, component names — match what was actually built.
- Any mismatch: **reality wins.** Note it as a deviation to absorb into the spec (step 2) and record in the changelog (step 4).
- While in each repo, capture the implementing commit SHA(s) or PR link(s) (`git log --oneline -10`, or ask the user) for the changelog entry.
- If a repo path is unreachable, say so and ask the user for the deviations and commit refs instead — do not silently skip the check.

## 2. Reconcile the service spec(s)

- Update the affected spec(s) to match what was actually built — field names, endpoint shapes, component names. Reality wins over the original plan.
- Remove the feature's `<!-- PENDING: ... -->` markers.
- Update each touched spec's single `Last updated` line. Do NOT append history entries (single-log rule: dated logs live only in `CHANGELOG.md`).

## 3. Flip the status board

- In `STATUS.md`, mark the feature row complete (✅ for the relevant columns) and move it from **In Flight** to the **Shipped** section.
- **The Notes cell is one line max** — a short pointer or superseded-decision note. Implementation detail belongs in `CHANGELOG.md` and the archived design doc, never in the board.
- Touch `00-architecture-overview.md` only if system-wide architecture changed; bump its `Last Updated:` if so.

## 4. Update the changelog

- Prepend one entry to `CHANGELOG.md` (newest-first): date, feature, what shipped, notable deviations.
- End the entry with the traceability refs captured in step 1: implementing commit SHA(s) or PR link(s) per service repo, e.g. `(main-api@a1b2c3d, admin-frontend#124)`.

## 5. Archive the feature file

- Move `Features/FEATURE-{name}.md` → `Features/Implemented/`, adding this banner at the top:

```markdown
> **ARCHIVED — historical design record. NOT a source of truth.**
> Implemented YYYY-MM-DD. Current state lives in {service-spec}.md.
> Do not edit; this captures the original design reasoning only.
```

## 6. Archive the prompt(s)

- Move each `Prompts/PROMPT-{service}-{feature}.md` → `Prompts/Implemented/`.

## 7. Sync repo instructions if conventions changed

- If the feature changed any shared convention (naming, module layout, storage patterns, env conventions): update `CONVENTIONS.md`, update the affected canonical file(s) in `repo-instructions/`, and remind the user to copy them into the service repo(s). The hub copies are canonical.

## 8. Finish

Report: drift-check result (deviations found), files reconciled, STATUS row flipped, changelog entry (with commit refs), archived files, and any convention syncs performed. Flag anything the specs still don't capture.
