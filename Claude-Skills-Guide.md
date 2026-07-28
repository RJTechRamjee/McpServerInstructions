# How Claude Skills Work (Beginner Guide)

This guide explains what a Claude Skill is and how to build one, using a
real skill we built and tested against our SAP A4H system as the worked
example: **transport-review**, which helps a dev lead sanity-check an ABAP
transport before approving its release.

**What you'll end up with:** the ability to package a repeatable piece of
your own expertise (a checklist, a workflow, a set of judgment calls) so
Claude applies it consistently, every time, without you re-explaining it
in every conversation.

---

## Before you start, make sure you have

- [ ] Claude Code (or another Claude app that supports skills) set up in
      this repo
- [ ] A repeatable task you keep re-explaining to Claude, or keep doing
      the same way yourself — that's the signal a skill is worth writing

---

## Step 1: Understand what a skill actually is

A **skill** is a folder containing at minimum one file, `SKILL.md`, with
two parts:

1. **YAML frontmatter** — `name` and `description`. This is the only part
   of the skill Claude reads *up front*, for every skill available in the
   session, before any of them are triggered.
2. **A markdown body** — the actual instructions: workflow steps, output
   format, edge cases, things to watch for. This only gets loaded into
   context when the skill is actually triggered.

That two-tier design is the whole point. You can have dozens of skills
installed and it costs almost nothing until one of them is relevant —
Claude scans short descriptions, not full instruction sets.

### How a skill differs from other things that sound similar

| Thing | What it is | Who decides to use it |
|---|---|---|
| **Skill** | Packaged instructions for *how* to do a recurring task | Claude, automatically, by matching your request against the skill's `description` (or you, explicitly, via `/skill-name`) |
| **MCP tool** (e.g. `abap_transport-unifiedDifference`) | A single capability — one API call | Claude, as a building block, whenever a skill or its own reasoning calls for it |
| **Slash command** | A user-typed shortcut for a specific prompt | You, explicitly, by typing it |
| **Subagent** | A separate Claude instance with its own context window, for isolating a large task | Claude, when a task is big enough to warrant delegating |

A skill is glue, not a tool. It doesn't add new capabilities — it tells
Claude *how to sequence the tools it already has* (in our case, the ADT-MCP
transport tools) the way an experienced person would.

---

## Step 2: Decide where the skill should live

| Location | Scope | Shared with team? |
|---|---|---|
| `.claude/skills/<name>/SKILL.md` in the repo | This project only | **Yes** — commit it, everyone who opens the repo gets it |
| `~/.claude/skills/<name>/SKILL.md` (user home) | Every project you work in | No — personal only |
| A plugin | Distributed separately | Depends on plugin distribution |

For anything you want the team to benefit from — like reviewing our
transports the same way every time regardless of who's doing the review —
**project-level, committed to git** is the right call. That's what we did.

---

## Step 3: Write the `description` like a trigger, not a summary

This is the part people get wrong most often. The `description` isn't
documentation for humans — it's the only signal Claude has, at decision
time, for "should I use this skill right now?" A vague description means
the skill silently never fires.

**Weak:** `Helps with transports.`

**Strong** (what we actually used):
```yaml
description: Use when reviewing an ABAP transport request before release
  or approval - summarizing what changed, flagging risk (deletions,
  sensitive objects, large diffs, cross-package changes, missing test
  evidence), and producing a release-decision checklist. Triggers on
  "review transport", "what's in transport <number>", "check this
  transport before release", "transport risk", "release approval", "can
  we release this TR".
```

Notice it front-loads *when* to use it, then explicitly lists phrases a
real person would type. Write descriptions the way you'd brief a new team
member on when to escalate to you — specific triggers, not a job title.

---

## Step 4: Write the body as a workflow, not a wish list

The body of `SKILL.md` should read like you're handing off a checklist to
a capable but new teammate: concrete steps, in order, with the exact tool
calls to make and the judgment calls to apply at each one. Ours (trimmed):

```markdown
## Workflow

1. Get the essentials — destination + transport number. Ask if missing,
   never guess.
2. Pull the diff — call abap_transport-unifiedDifference, paginate via
   next_cursor until status is "completed".
3. Classify every changed object — name, type, new/modified/deleted, size
   of change.
4. Flag risk — deletions, security-relevant objects, cross-package
   changes, large diffs, hardcoded values, missing test evidence.
5. Produce the output — a markdown table, always, plus a short risk
   summary.
6. End with a recommendation, not a decision — the human approves the
   release, not Claude.
```

Two things worth copying into your own skills:

- **Name the exact tool calls.** Don't make Claude re-derive which MCP
  tool to use — say it outright (`abap_transport-unifiedDifference`,
  handle `next_cursor` pagination). This is what separates a skill from
  a vague suggestion.
- **State what's out of scope.** Our skill explicitly says it does *not*
  run ATC checks, even though that might feel like a natural extension —
  otherwise Claude might silently expand scope on its own.

The full file is at [.claude/skills/transport-review/SKILL.md](.claude/skills/transport-review/SKILL.md).

---

## Step 5: Test it against something real

We tested `transport-review` against A4H directly:

1. Created a throwaway class (`ZCL_TR_REVIEW_DEMO`) in a demo package, in
   transport `A4HK900116`.
2. Ran the skill's exact workflow: `abap_transport-unifiedDifference` →
   classify → flag → table.
3. Got back:

   | Object | Type | Change | Risk |
   |---|---|---|---|
   | ZCL_TR_REVIEW_DEMO | CLAS/OC | New: empty class skeleton, no logic yet | — |

   *Recommendation: Low risk — single new object, no deletions, nothing
   security-sensitive, no implementation yet so no test coverage
   expected. Safe to release as-is.*

If a skill can't produce a sane result against a real (even trivial)
case, its instructions are wrong — fix the `SKILL.md`, not the test.

---

## Step 6: Share it with the team

Because this lives at `.claude/skills/transport-review/` inside the repo,
committing it is the whole distribution mechanism — anyone who clones the
repo and opens it in Claude Code has the skill available immediately, no
install step.

```
git add .claude/skills/transport-review/SKILL.md
git commit -m "Add transport-review skill"
```

---

## Try it yourself

Once merged, try asking Claude, in this repo:

```
Can you review transport A4HK900116 before I release it?
```

Claude should recognize the request matches the `transport-review`
skill's description, load its instructions, and walk the workflow above
without you having to explain any of it again.

---

## Quick troubleshooting

| Problem | Likely fix |
|---|---|
| Skill never seems to trigger | `description` is too vague or missing the phrases people actually type — add concrete trigger language |
| Skill triggers but gives inconsistent output | Body is too loose — name exact tool calls and a fixed output format, don't leave it to improvisation |
| Skill overreaches (does things you didn't ask) | Add an explicit "out of scope" section, as we did for ATC checks |
| Team members don't have the skill | Confirm it's committed under `.claude/skills/` and they've pulled the latest repo state |

---

## Where to go from here

`transport-review` is deliberately narrow — one workflow, done well. As
more day-to-day dev-lead/architect tasks come up (code quality gates via
ATC, object-creation scaffolding, unit test oversight), the same pattern
applies: one skill per workflow, each with a sharp trigger description,
rather than one giant do-everything skill.