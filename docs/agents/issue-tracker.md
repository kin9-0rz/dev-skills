# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files under `.scratch/`. This file is the only tracker contract; GitHub Issues is at most a read-only mirror and never carries claims, blocking edges, or answers.

## Layout

- One effort per directory: `.scratch/<effort>/`
- Handoff spec: `.scratch/<effort>/spec.md` (six sections; see below)
- Execution tickets: one file per ticket at `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01` in dependency order. Never a combined tickets file.
- Decision requests: `.scratch/<effort>/requests/NN-<slug>.md`
- Resume notes: `.scratch/<effort>/resume/NN.md`
- Decision trail for work with no PR: `.scratch/<effort>/decision-trail.md`
- Wayfinder maps: `.scratch/<effort>/map.md` with decision tickets under the same `issues/` directory.

## Ticket fields

Every execution ticket starts with:

```markdown
# <title>

Status: open
Owner:
Blocked by:
PR:
```

`Owner:` is required from the claim onward. Execution tickets have no `Type:` line; `Type:` belongs only to wayfinder decision tickets (`research` / `prototype` / `grilling` / `task`).

## Spec fields

`spec.md` carries exactly these six numbered sections, plus `## 可选`, `## Spec changes`, and `## Self-check`:

1. Goal and problem
2. Scope (`### In` / `### Out`)
3. Acceptance evidence: category, tier, check/command, required result
4. Ticket graph table: id, title, blocking edges, single-ticket behavior
5. Source pointers: tickets, `GLOSSARY.md`, ADRs, research branch
6. Constraints: runtime prerequisites, interfaces/files that must not change, environment entry commands

`## Self-check` maps every `### In` item to at least one ticket acceptance item.

## State machine

`open → claimed → resolved`.

- **Frontier**: `Status: open`, empty `Owner:`, every `Blocked by` ticket resolved. First by number wins.
- **Claim**: set `Status: claimed` and `Owner: <name>`, then commit on trunk: `claim: NN <slug>`. The file is the lock; a later session that sees an owner skips the ticket. Claim before any work.
- **Resolve**: only after the artifact reached the agreed destination (PR merged, or the no-PR branch plus decision-trail receipt) and the receipt is present. Set `Status: resolved` and record the PR link in a trunk metadata commit.

Resolved tickets stay resolved. Only a wrong resolve reopens one: set `Status: open`, clear `Owner:`, and move `## Answer` to `## Comments`.

## Where writes go

Tracker writes — claim, `Owner:`, `PR:`, status, decision requests, resume notes, decision trail — are metadata commits directly on trunk. A code branch contains zero `.scratch` files.

The planning side writes ticket bodies. The execution side appends only `## Comments` pointers.

## When a skill says "publish to the issue tracker"

- **Spec** → write `.scratch/<effort>/spec.md`.
- **Tickets** → write one file per ticket under `.scratch/<effort>/issues/`, in dependency order.
- **Decision request** → write `.scratch/<effort>/requests/NN-<slug>.md`, keep the ticket `claimed`, and leave `## Comments` as a pointer only.

Do not create GitHub issues, apply triage labels, or post `ready-for-agent`.

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path, or resolve the number under `.scratch/<effort>/issues/`.

## Wayfinding operations

Used by `wayfinder`. The map is a file with one child file per decision ticket.

- **Map**: `.scratch/<effort>/map.md` with Notes, Decisions so far, and Fog sections.
- **Decision ticket**: `.scratch/<effort>/issues/NN-<slug>.md` with a `Type:` line and a `Status:` line (`open` / `claimed` / `resolved`).
- **Blocking**: `Blocked by: NN, NN`; unblocked when every listed ticket is resolved.
- **Frontier**: open, unblocked, unclaimed; first by number.
- **Claim**: `Status: claimed` plus `Owner:` and a claim commit.
- **Resolve**: append `## Answer`, set `Status: resolved`, then append a gist plus link to the map's Decisions so far, all in one trunk metadata commit.
