---
name: clue
description: Record and maintain Clues — retrospective, evidence-based findings about pre-existing, undocumented systems you are reverse-engineering. Use this skill whenever you uncover a non-obvious fact about how something already built actually works — a behaviour you had to dig for, a load-bearing config you didn't know about, why a system does the surprising thing it does. Trigger it proactively when the conversation reaches a discovery ("oh, it reads the legacy field", "so that's why it does X", "I think this is what's happening here", "turns out the fallback masks it"), not only when the user says the word "clue". Also use it to confirm or refute an earlier clue, to supersede one with a refined finding, or when asked where a past finding is recorded or how we know something about a system. Use the adr skill instead when you are recording a decision you are making while building; a clue is the opposite — a fact you have uncovered about something already built.
---

The first question is what project are we working on?

# Clues

A clue captures a single uncovered fact about a system you did not build: the finding
itself, the evidence that supports it, how confident you are, and what it implies. The
point is to preserve the **how you know** so a future reader (human or agent) doesn't
re-derive the fact from scratch, or — worse — trust a hunch as settled truth and build
on sand.

A clue is the mirror image of an ADR. An ADR records the **why** of a decision *while you
build*; a clue records what you have **reverse-engineered** about something *already
built*, often years ago, with no documentation and no one left to ask. Where an ADR is
written from the position of authorship, a clue is written from the position of an
investigator reconstructing intent from artifacts.

## Where clues live

You must ask the user what project they're working on. Project names are "kebab-case".

`.afharvey/clue/project/` in the repo, one Markdown file per finding:

```
.afharvey/clue/
├── 0000-template.md          # the template (copy this; never assign it a number)
├── project
    ├── 0001-auth-cm-keyed-by-legacy-cluster-id.md
    ├── 0002-octavia-health-check-is-tcp-only.md
    └── ...
```

Filenames are `NNNN-kebab-case-title.md`. The number is a zero-padded sequential
integer that **never changes once assigned** and is never reused.

To create the next one: list `.afharvey/clue/project/`, take the highest existing number,
add one. Don't guess — read the directory.

## The rules that matter

1. **One finding per clue.** If you've uncovered two facts, write two files.
2. **Evidence is mandatory.** Cite where you know it from — `path/to/file.go:42`, commit
   SHAs, log lines, observed behaviour. The evidence is the heart of the record: it's what
   lets the next reader trust the clue or re-check it. A finding with no evidence is a
   guess; if that's all you have, say so and mark it `Suspected` with Low confidence.
3. **State your confidence honestly.** Distinguish what you have *verified* from what you
   *infer*. A confident-sounding clue that was never checked is the most dangerous kind.
4. **Keep refuted clues.** A hypothesis you chased and disproved is as valuable as the
   rejected alternatives in an ADR — it stops the next investigator burning a day
   re-chasing it. Mark it `Refuted` and note what disproved it; do not delete it.
5. **Living while suspected, supersede once confirmed.** Amend a `Suspected` clue freely
   as you learn. Once you flip it to `Confirmed`, treat it as settled — if it later proves
   wrong or gets refined, write a *new* clue that supersedes it rather than rewriting
   history. The trail of how understanding evolved is itself a clue.
6. **Keep it short.** A finding and its evidence, not an essay. If a clue needs extensive
   reconstruction, that exploration lives elsewhere (e.g. `.afharvey/design/`) and the clue
   links to it and states the conclusion.

## Status lifecycle

- `Suspected` — a hypothesis backed by partial or circumstantial evidence. Open to
  amendment as you learn more.
- `Confirmed` — verified with solid, cited evidence. Now treat it as settled.
- `Refuted` — investigated and found false. Keep it; note what disproved it so no one
  re-chases the dead end.
- `Superseded by CLUE-NNNN` — a later clue refined or corrected this finding. Set this on
  the old clue; do not delete it. History stays in the tree.

  When superseding: create the new clue with its own status, give it a
  `Supersedes CLUE-NNNN` line, and edit only the `Status` line of the old one to
  `Superseded by CLUE-MMMM`. The old finding and its evidence stay intact.

`Confidence` (Low / Medium / High) is a separate axis from `Status`: a `Suspected` clue
can be a weak hunch or a strong-but-unverified inference, and saying which matters.

## Workflow

When you uncover a fact about an existing system, or revisit an earlier finding:

1. Confirm it's clue-worthy. A non-obvious fact you had to dig for — surprising behaviour,
   a load-bearing constraint, a "why does it do that" finally answered — qualifies.
   Anything obvious from a glance at the code does not.
2. Read `.afharvey/clue/project/` to find the next number.
3. Copy the template, fill it in — especially the **Evidence**, which is what makes the
   clue trustworthy and re-checkable.
4. Default the status to `Suspected` and set an honest `Confidence`. Flip to `Confirmed`
   only once you've actually verified it.
5. If this corrects or refines an earlier `Confirmed` clue, write it as a new clue and
   update the old one's `Status` line to `Superseded by CLUE-NNNN`. If you disproved a
   `Suspected` clue, mark it `Refuted`.

## Template

Copy `.afharvey/clue/0000-template.md` (reproduced here for reference):

```markdown
# NNNN. <Short title of the finding>

- **Status**: Suspected
- **Confidence**: Low | Medium | High
- **Date**: YYYY-MM-DD
- **Investigator**: <who dug this up>

<!-- Supersedes CLUE-NNNN  (only if applicable) -->

## Finding

What you now believe to be true, stated plainly. One finding.

## Evidence

How you know — the code paths, configs, commits, logs, or observed behaviour that
support this. Cite specifics (`path/to/file.go:42`, commit SHAs, log lines). This is the
heart of the record: it's what lets the next reader trust the clue or re-check it.

## Context

What you were doing or trying to understand when you found this, and where in the system
you were looking. Enough for a future reader to know what prompted the dig.

## Implications

What this means for the work — gotchas, load-bearing constraints, "don't touch X because
Y", what is now safe or unsafe to assume.

## Open questions

What's still unknown or unverified — the leads worth chasing next, and what would confirm
or refute this clue.
```

## Example (for calibration)

A good clue for this codebase reads like: *"Auth configmap still keyed by the legacy
cluster id"* — context is that you were chasing why a rename didn't take effect; the
finding is that the auth component reads the pre-rename `clusterId` field through a silent
fallback; the evidence is `auth-auth.cm.working.yaml` plus the originating commit that
added the fallback; confidence is High because you traced the read path; the implication
is that renaming the field breaks silently because the fallback swallows the absence; the
open question is whether anything still *writes* the old field. Short — names where it
lives, what proves it, and what it breaks.
