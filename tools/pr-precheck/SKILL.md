---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

Answer one question about one PR package: **is this ready to submit?**

A PR package is a candidate pull request (its title, description,
commit list, diff, and test evidence), read against two things: the
plan it claims to implement (including that plan's deviation notes),
and the issue that plan belongs to (its thread and the repo's stated
asks). Grade nothing else. Do not review the code for style, do not
second-guess the plan's diagnosis, and do not grade more than one
package per run. The plan was already accepted; this tool asks
whether the PR delivers it, proves it, keeps it reviewable, and meets
the repo's standards.

## Inputs and modes

Decide the mode first, from what you were given.

**Eval mode**: you were given a package bundle (a markdown file with
`Repo facts`, `Issue`, `Thread highlights`, `Plan context`, and
`Candidate PR` sections: title, description, commits, diff, test
evidence). The bundle is the whole world. Every fact comes from the
bundle text. Fetch nothing, read no other file, run no command.
Ignore `scope.md` and `voice-guide.md`. Grade every check in the
rubric and apply the full verdict rule. Set `item` to the bundle id
(for example `pkg-07`).

**Live mode**: you were asked to check the student's own draft PR.
Read these inputs, from these sources:

- **Plan**: `plan.md` in the working directory, including its
  `## Deviations` section. A house-chain student reads the house plan
  and house repro pack they were given instead; the same checks apply.
- **Diff**: run `git diff main...HEAD` (three dots) from the working
  copy. That is every committed change the branch makes relative to
  `main`. Also run `git log --oneline main..HEAD` for the commit list.
  Uncommitted changes are not part of the PR: do not grade them, but
  mention them in the summary if `git status` shows any.
- **Title and description**: `pr_draft.md`. The first line is the
  title; the rest is the description (the filled PR template).
- **Test evidence**: `test_evidence.md`.
- **Issue side**: the issue named in the request. Read its body and
  thread (via `gh`, or the public GitHub API with `curl` or a web
  fetch). Read the repo's PR template
  (`.github/PULL_REQUEST_TEMPLATE.md`) and `docs/CONTRIBUTING.md` from
  the working copy.

If a live-mode input file is missing, do not stop: grade the checks
that need it as `fail` with the evidence "`<file>` not found", and
say so in the summary. Set `item` to the issue URL (or the PR URL if
one exists).

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else.

- If its `Repo:` line still holds the `<ORG>/<PATH-REVIEW-REPO>`
  placeholder, stop without grading. Tell the student to fill the
  `Repo:` line in `scope.md` with their section's Path Review repo.
  Emit no verdict.
- If the issue or PR belongs to any other repository, refuse to grade
  and say which repo the scope allows.
- Apply its house rules while reading evidence. In particular, a
  classmate's PR on the same issue does not count against this one,
  and the repo's PR template must be used with real content in every
  section.

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` after the rubric. Hold the PR
title and description (from `pr_draft.md`) against each of its rules.
In the summary, list every rule the draft breaks: quote the rule's
name and the phrase that breaks it. The voice guide never changes a
check grade or the verdict by itself, because voice is the student's
own and the rubric holds the universal standards. In eval mode,
ignore `voice-guide.md` entirely.

## Component reads

This file is the frame. The judgment lives in three other files.
Read all three before grading:

- `rubric.md` defines the checks (what each one reads and its pass
  condition) and the verdict rule.
- `references/evidence-guide.md` maps where each evidence family
  lives in a bundle and in a live PR, with worked pass/fail examples.
  When a case looks like one of its examples, grade it the way the
  example is graded.
- `procedure.md` is the operating procedure: the read order, the
  evidence notes to take, the order checks run in, and how the
  verdict is assembled. Execute it as written, step by step.

If the procedure is silent on something you need to do, do not invent
a step. Make the narrowest reasonable move, and name the gap in the
summary as "procedure gap: ...".

If `rubric.md` has no filled check rows or no verdict rule, or
`procedure.md` has no filled steps, refuse to grade: say which file is
empty and emit no verdict. A tool without its judgment files would
only produce noise that looks like judgment.

## Verdict and output

The verdict is binary. `accept` means ready to submit. `reject` means
hold. There is no third verdict and no score; put any reservation in
a check's evidence line.

Before the JSON, write a short readable summary: one line per check
with its grade and deciding fact, the deciding check for a reject,
any procedure gaps, and (live mode) any voice-guide notes.

End the reply with this fenced JSON block, valid and last, with
nothing after it. Use the rubric's check names, in rubric order:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or
  quote that decided it. "Looks fine" is not evidence.
- **Grade the thing, not the polish.** Read the diff itself against
  the plan, the evidence itself against the test plan, and the
  description itself against the diff. A terse PR can be ready; a
  confident, well-formatted one can hide drift. Length, headings, and
  tone decide nothing.
- **The rubric decides.** If a check passes by its stated condition
  but feels wrong, it still passes. Note the tension in the summary;
  the fix belongs in the rubric.
- **The procedure decides how.** Follow `procedure.md` as written and
  report its gaps instead of improvising.
- **Unclear is a fail.** Apply the rubric's verdict rule to `unclear`.
  Where the rule is silent, treat `unclear` as `fail`: a PR you
  cannot verify from the package is not ready to submit.
- **Honest shortfalls are not failures.** A PR that does less than
  everything can still be ready when the plan's deviation notes and
  the description both say what was left out and why.
