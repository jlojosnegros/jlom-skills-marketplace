# adversarial-review

A Claude Code skill that **adversarially audits a finished analysis before you
act on it**. You give it a report, design doc, PR/MR review, or JIRA-ticket
analysis along with its sources, and it attacks the document looking for the
specific ways AI-generated analyses fail silently:

- **Fabricated facts and unverified numbers** — a precise figure ("reduces build
  time by ~30 min") stated with no benchmark, log, or measurement behind it.
- **Partially-read sources** — a citation that tracks a source's intro but misses
  a caveat or conclusion further down.
- **Unreliable sources** — a claim laundered into fact from a forum post or SEO
  page instead of a primary source.
- **Self-referential claims** — the agent cites its own earlier output (e.g. a
  JIRA comment it wrote yesterday) as if it were independent truth. This is the
  loop that turns one fabricated number into weeks of wasted work.
- **Confident silence** — a load-bearing claim asserted flatly that was never
  actually checked, with no warning that it wasn't.

The output is a single markdown audit you can feed back into the model: a
findings report with severity (numbered `F-1`, `F-2`, … so you can say
"regenerate `F-1` only"), a claim-by-claim verification table marking each claim
`VERIFIED` / `UNVERIFIED` / `UNRELIABLE` / `CONTRADICTED` / `FABRICATION RISK`, an
assumptions inventory, alternative explanations with distinguishing tests, a
"verify before acting" checklist, suggested JIRA corrections to break the
self-source loop, and an investigation log of every check actually performed.

## Core principle

Presumption of guilt: every claim is `UNVERIFIED` until concrete, external,
inspectable evidence upgrades it. **A document cannot be its own source** — if
the only support for a claim is another sentence in the same document or an
earlier AI-authored note, it's `UNRELIABLE — SELF-SOURCED`, no matter how
confident it sounds.

It is honest in both directions: it won't wave through a rotten document, and it
won't manufacture findings to dramatize a sound one. A clean audit ("11 of 11
load-bearing claims verified against the diff, safe to act on") is a real result.

## Usage

The skill triggers on its own when you ask Claude to review, audit, challenge, or
verify a document — or proactively when a document makes a quantitative or causal
claim with no inspectable source. Typical prompts:

```text
Audit this design doc before I act on it — sources are the linked PR and PROJ-1423.
```

```text
Adversarial review of this report. Did it actually verify the build-time claim?
```

For best results, give it the **sources** alongside the document (the PR/MR diff,
the JIRA ticket, the requirement doc, the repo path, the URLs). A claim it can't
verify because the source is absent comes back `UNVERIFIED` — not waved through.

## What's inside

- `skills/adversarial-review/SKILL.md` — the skill: stance, 5-pass audit
  procedure, output format, severity rubric.
- `skills/adversarial-review/references/patterns.md` — worked examples of each
  failure pattern (the fabricated metric, the JIRA gospel loop, the half-read
  source, …) and how a good audit catches it.

## Feedback loop

The intended workflow: run the audit on a generated report, hand the audit
document back to the model, and ask it to regenerate only the findings marked
`UNVERIFIED` / `FABRICATION RISK` — after running the checks listed in "verify
before acting". This closes the loop that otherwise lets unchecked claims
propagate into JIRA and back.
