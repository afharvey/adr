---
name: adr
description: Write and maintain Architecture Decision Records (ADRs). Use this skill whenever a non-trivial architectural, infrastructure, or tooling decision is being made or has just been made — choosing between technologies, changing a topology, adopting/dropping a dependency, or settling a trade-off worth remembering. Trigger it proactively when the conversation reaches a decision point ("let's go with X over Y", "we'll self-manage instead of...", "switch from A to B"), not only when the user says the word "ADR". Also use it to supersede or amend an existing decision, or when asked where a past decision is recorded or why something was chosen.
---

# Architecture Decision Records

An ADR captures a single architectural decision: the context that forced it, the
options considered, the choice made, and its consequences. The point is to preserve
the **why** so a future reader (human or agent) doesn't relitigate a settled
question or undo a load-bearing constraint without knowing what it was load-bearing
for.

## Where ADRs live

`docs/adr/` in the repo, one Markdown file per decision:

```
docs/adr/
├── 0000-template.md          # the template (copy this; never assign it a number)
├── 0001-octavia-tcp-passthrough.md
├── 0002-dns-01-acme-via-gcore.md
└── ...
```

Filenames are `NNNN-kebab-case-title.md`. The number is a zero-padded sequential
integer that **never changes once assigned** and is never reused.

To create the next one: list `docs/adr/`, take the highest existing number, add one.
Don't guess — read the directory.

## The rules that matter

1. **One decision per ADR.** If you're recording two decisions, write two files.
2. **Accepted ADRs are immutable.** Once status is `Accepted`, the record of the
   decision and its reasoning is frozen. You do not edit it to reflect a new reality.
   If the decision changes, write a *new* ADR that supersedes it (see below). The only
   edits permitted to an accepted ADR are fixing the `Status` line when it gets
   superseded, and trivial typo fixes.
3. **Record the rejected options and why.** The losing alternatives and the reason
   they lost are the most valuable part of the record. An ADR with only the chosen
   option is barely worth writing.
4. **Write in past/decisive tense.** "We will use Octavia as a dumb TCP passthrough,"
   not "we could maybe consider Octavia."
5. **Keep it short.** A page or two. ADRs are not design docs; if a decision needs
   extensive exploration, that exploration lives in `docs/design/` and the ADR links
   to it and states the conclusion.

## Status lifecycle

- `Proposed` — written, not yet agreed. Open for discussion.
- `Accepted` — agreed and in force. Now immutable.
- `Superseded by ADR-NNNN` — a later ADR replaced this decision. Set this on the old
  ADR; do not delete it. History stays in the tree.
- `Deprecated` — no longer applies, but nothing replaced it (e.g. the component was
  removed entirely).

When superseding: create the new ADR with status `Accepted`, give it a
`Supersedes ADR-NNNN` line, and edit only the `Status` line of the old one to
`Superseded by ADR-MMMM`. The old reasoning stays intact.

## Workflow

When a decision is being made or has just been made in conversation:

1. Confirm it's ADR-worthy. Architectural/infra/tooling decisions with non-obvious
   trade-offs or long-lived consequences qualify. Routine, easily-reversed, or purely
   stylistic choices do not.
2. Read `docs/adr/` to find the next number.
3. Copy the template, fill it in from the conversation — especially the
   alternatives that were weighed and rejected.
4. Default the status to `Proposed` and confirm with the user before flipping to
   `Accepted`, unless they've clearly already decided.
5. If this supersedes an earlier ADR, update the old one's `Status` line.

## Template

Copy `docs/adr/0000-template.md` (reproduced here for reference):

```markdown
# NNNN. <Short title of the decision>

- **Status**: Proposed
- **Date**: YYYY-MM-DD
- **Deciders**: <who made or owns this>
<!-- Supersedes ADR-NNNN  (only if applicable) -->

## Context

What forces are at play — technical, operational, sovereignty, cost? What problem or
constraint makes a decision necessary now? State the situation neutrally; don't
pre-justify the outcome here.

## Decision

The choice, stated plainly and decisively. One decision.

## Alternatives considered

- **<Option A>** — why it was rejected.
- **<Option B>** — why it was rejected.
- (The chosen option can appear here too, framed as why it won.)

## Consequences

What becomes easier, harder, or constrained as a result — including the downsides we
are knowingly accepting. Note any follow-on work or new constraints this creates.
```

## Example (for calibration)

A good ADR for this codebase reads like: *"Octavia as a dumb TCP passthrough"* —
context is that pods sit on a private subnet behind Gcore with origin mTLS; the
decision is TCP passthrough with no TLS termination on the LB; the rejected
alternative is terminating TLS at Octavia, rejected to avoid Barbican cert-sync
complexity; the consequence is that TLS termination has to happen elsewhere in the
cluster and that's a separate open decision. Short, names the loser, names the cost.