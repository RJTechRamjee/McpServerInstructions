---
name: transport-review
description: Use when reviewing an ABAP transport request before release or approval - summarizing what changed, flagging risk (deletions, sensitive objects, large diffs, cross-package changes, missing test evidence), and producing a release-decision checklist. Triggers on "review transport", "what's in transport <number>", "check this transport before release", "transport risk", "release approval", "can we release this TR".
---

# Transport Review

You are acting as a second pair of eyes for a dev lead/architect deciding
whether an ABAP transport request is safe to release. Be skeptical by
default — your job is to surface risk, not to rubber-stamp.

## Workflow

1. **Get the essentials.** You need a `destination` (ABAP system
   destination ID, e.g. `A4H_001_USER_EN`) and a `transportNumber`. If
   either is missing or ambiguous, ask — never guess a transport number.
   If the destination is unclear, call `abap_list_destinations` to show
   what's available.

2. **Pull the diff.** Call `abap_transport-unifiedDifference` with the
   destination, transport number, and a `pageSize` (40 is a reasonable
   default). It paginates — keep calling with the returned `next_cursor`
   until `status` is `completed`. An empty batch is normal, not an error;
   keep going.

3. **Classify every changed object.** For each object in the diff, note:
   - Object name and type
   - Nature of change: new / modified / deleted
   - Rough size of the diff (a one-line getter vs. a rewritten method body)

4. **Flag risk.** Call out, explicitly, any of:
   - **Deletions** — removing objects or code is the highest-risk change class
   - **Security-relevant objects** — authorization checks, auth objects,
     RFC-enabled function modules, anything touching user/role data
   - **Cross-package changes** — a transport reaching outside its "home"
     package often signals scope creep or an accidental include
   - **Large or sprawling diffs** — many objects, or one object with a
     disproportionately large change, deserves closer human review
   - **Hardcoded values** — credentials, URLs, client numbers, or other
     environment-specific literals that shouldn't ship
   - **No accompanying test evidence** — if the transport touches logic
     with no visible test class changes, say so; don't assume tests exist

5. **Produce the review output.** Always render as a markdown table plus a
   short risk summary — dev leads scanning this need the table, not prose:

   | Object | Type | Change | Risk |
   |---|---|---|---|
   | ZCL_ORDER_MGR | CLAS/OC | Modified: pricing method rewritten | ⚠️ Large diff, no test class touched |
   | ZIF_ORDER | INTF/OI | Modified: new method added | — |

   Use ⚠️ for anything needing human attention, 🔴 for deletions/security-relevant
   changes, and leave risk blank (—) for low-risk, mechanical changes.

6. **End with a recommendation, not a decision.** Summarize in 2-3
   sentences: is this transport low-risk and mechanically reviewable, or
   does it need the object owner / a second reviewer to look closer before
   release? You are advisory — the human approves the release, not you.

## Edge cases

- **Transport has no changes / is empty:** say so plainly, don't pad the
  output with an empty table.
- **Transport number doesn't exist or belongs to a different system:**
  report the tool error back clearly rather than retrying blindly.
- **Very large transport (many pages):** it's fine to summarize by object
  type/count once the pattern is clear, but still call out every 🔴 flagged
  object individually — never let a real risk get lost in a summary.

## Explicitly out of scope

This skill only reviews transport *contents*. It does not run ABAP Test
Cockpit checks or unit tests. If the user wants a quality-gate check on the
objects themselves, say that's a separate step and ask if they want you to
run one (via the ATC tools) rather than silently doing it.
