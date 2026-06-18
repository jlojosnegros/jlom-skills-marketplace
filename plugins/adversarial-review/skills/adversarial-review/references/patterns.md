# Failure patterns and worked examples

This file shows the recurring ways AI-generated analyses fail and how a good
adversarial audit catches each one. Read it when you want concrete shape for the
verdicts in SKILL.md. Each pattern has the smell (how to spot it), the trap (why
it slips through), and the audit move (what to write).

## Table of contents

1. The fabricated metric
2. The self-source loop (JIRA gospel)
3. The half-read source
4. The unreliable source laundered into fact
5. The confident silence
6. The existence claim nobody grepped
7. The stale fact
8. The clean document (don't manufacture findings)

---

## 1. The fabricated metric

**Smell.** A precise quantity that could only be known by measuring or looking
up — "reduces build time by 30 min", "cuts bundle size 42%", "p99 latency
180ms" — appears with no benchmark, no log, no command output attached.

**Trap.** The number is plausible. It's the right order of magnitude. The
surrounding prose is competent, so the reader's guard is down. Nobody asks "where
did 30 come from?"

**Audit move.** Verdict: **FABRICATION RISK**, severity CRITICAL if anything is
being built on it.

> **Claim:** Switching to incremental compilation reduces artifact build time by
> ~30 minutes (§3).
> **Verdict:** FABRICATION RISK.
> **Why:** No benchmark output, CI timing, or before/after measurement is present
> anywhere in the document or its sources. A 30-minute delta cannot be known
> without measurement; this number has no provenance.
> **Impact if wrong:** The entire two-week optimization effort is predicated on
> this saving existing. If it's 3 minutes, the work is misprioritized.
> **What would settle it:** Run the build both ways and compare wall-clock time,
> e.g. `time <build cmd>` on a clean checkout with and without the change, ≥3
> runs each.

---

## 2. The self-source loop (JIRA gospel)

**Smell.** A claim is stated with total confidence and, when you trace it, its
only support is a JIRA comment — and that comment reads like AI output, or is of
unknown authorship, with no link to a diff, spec, or measurement.

**Trap.** The ticket _looks_ canonical. It's in the system of record. It's been
restated across comments. Repetition feels like corroboration. The agent that
wrote it yesterday reads it today as external truth.

**Audit move.** Verdict: **UNRELIABLE — SELF-SOURCED**. Name the loop explicitly.
Repetition is not verification.

> **Claim:** "As established in PROJ-1423, the 30-min saving is confirmed."
> **Verdict:** UNRELIABLE — SELF-SOURCED.
> **Why:** PROJ-1423's comment is the _origin_ of the figure, not an independent
> confirmation of it. The comment cites no measurement and appears to be the
> agent's own earlier output. The claim is circular: it is sourced to a
> restatement of itself.
> **Impact if wrong:** This is the mechanism by which a fabricated number became
> "fact". Every downstream doc citing PROJ-1423 inherits the error.
> **What would settle it:** Independent measurement (see pattern 1). Until then,
> the JIRA comment should be annotated as unverified.

If asked, supply a corrected comment:

> Suggested edit to PROJ-1423: "⚠️ Unverified: the ~30-min build-time saving
> stated here has not been measured. Do not treat as established until a
> before/after benchmark is attached."

---

## 3. The half-read source

**Smell.** The document's summary tracks the opening of a source closely, then
omits a caveat, exception, or conclusion that the source's later sections make.
The recommendation aligns with the abstract but contradicts the body.

**Trap.** The cited source is real and authoritative, so the citation looks
solid. But the source was only partially consumed — the agent read the first
screen and stopped.

**Audit move.** Verdict on the affected claim: **CONTRADICTED** or **UNVERIFIED**
depending on severity, and call out the partial read in Pass 3.

> **Claim:** The migration guide says the upgrade is drop-in (§2).
> **Verdict:** CONTRADICTED.
> **Why:** The guide's intro does call it drop-in, but its "Breaking changes"
> section (further down the same page) lists three signature changes affecting
> exactly the modules in this PR. The document appears to have read the intro
> only.
> **What would settle it:** Re-read the full guide, especially the breaking-
> changes section, and reconcile.

---

## 4. The unreliable source laundered into fact

**Smell.** A claim rests on a forum answer, a content-farm article, an SEO
listicle, or a random blog — but the document states it as settled.

**Trap.** Once a claim is paraphrased into a confident sentence, the weakness of
its origin disappears. The reader never sees that "best practice" came from a
2019 Medium post.

**Audit move.** Downgrade. Verdict **UNVERIFIED** (or CONTRADICTED if a primary
source disagrees), severity HIGH if load-bearing. Name the weak source and the
primary source that _should_ have been used.

> **Why:** This rests on a Stack Overflow answer, not the library's official
> docs. The official docs (linked) specify the opposite default. Primary source
> beats forum.

---

## 5. The confident silence

**Smell.** A load-bearing claim is stated flatly, as established fact, with no
hedge — and yet nothing in the sources could have established it. The document
never says "I did not verify this."

**Trap.** Absence of a caveat reads as presence of verification. Confident tone
substitutes for checking. This is the _aggravating_ factor in the user's case:
not just that it was wrong, but that no warning was given.

**Audit move.** Flag the undisclosed non-verification itself as a finding, not
just the claim.

> **Finding [HIGH] Undisclosed non-verification of the central premise**
> The document states the saving as fact with no caveat, but contains no
> evidence it was measured. Presenting an unmeasured figure without flagging it
> as unmeasured is the failure that cost two weeks. Every load-bearing claim
> should carry its verification status; this one carried false confidence.

---

## 6. The existence claim nobody grepped

**Smell.** "This function is unused", "no test covers this path", "only service A
calls this endpoint" — checkable claims that nobody actually checked.

**Trap.** These are easy to _say_ and easy to _verify_, which is exactly why an
unverified one is damning. They feel like observations; they're assertions.

**Audit move.** If you can grep/search, do it and upgrade to VERIFIED or
CONTRADICTED. If you can't, UNVERIFIED with the exact command that would settle
it.

> **What would settle it:** `grep -rn "functionName" src/` across the repo plus a
> check of dynamic/reflection call sites; "unused" is unsafe until both are
> clean.

---

## 7. The stale fact

**Smell.** A claim about something that changes — a library version, an API
default, a price, who owns a service, whether a ticket is open — sourced (if at
all) to something that may no longer be current.

**Trap.** It was true once. The source is real. But "current" decays silently.

**Audit move.** Verdict UNVERIFIED with a currency note; for repo/ticket state,
re-check the live source.

---

## 8. The clean document — don't manufacture findings

Not every document is rotten. When the load-bearing claims are genuinely traced
to good sources and you checked them, **say so plainly and stop.** A short audit
that reads "11 of 11 load-bearing claims verified against the PR diff and the
linked spec; safe to act on; one MEDIUM note on a missing rollback risk" is a
real result and exactly as valuable as catching a fabrication.

Inventing severity to look thorough is itself a failure of the same family you're
auditing — confident assertion unsupported by evidence. Hold the line in both
directions: don't wave through the rotten, don't dramatize the sound.
