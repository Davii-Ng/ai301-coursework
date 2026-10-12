# Procedure: how this tool grades a PR package

Follow these steps in order. Each step names what to read and what to
write down. Take the notes in your working, not in the final JSON;
checks are graded from the notes, so two executors with the same
package build the same notes and reach the same grades.

`rubric.md` says what each check decides. `references/evidence-guide.md`
says where each signal lives and shows worked examples. When a case
looks like one of those examples, grade it the way the example is
graded.

## Read order

Read what the PR must deliver before reading what the PR says it
delivers. The plan comes first, then the diff, then the evidence, and
the description last. A description read first anchors the reader on
its own story ("implements the plan exactly"), and then the diff gets
skimmed for confirmation instead of checked. So:

1. **Mode.** Eval mode (a bundle file): the bundle is the only
   source; skip `scope.md` and `voice-guide.md`. Live mode: read
   `scope.md` first and apply it as `SKILL.md` says; then collect the
   inputs `SKILL.md` lists (`plan.md`, `git diff main...HEAD`,
   `git log --oneline main..HEAD`, `pr_draft.md`, `test_evidence.md`,
   the issue thread, the PR template, `docs/CONTRIBUTING.md`).
2. **Repo facts.** Copy the `pull requests:` asks into a list (every
   template section, checklist item, and required file). Classify the
   AI policy as: requires disclosure / conditions only / silent.
3. **Issue and thread.** Note the reported behavior in one line. List
   every maintainer-side comment (OWNER, MEMBER, COLLABORATOR, or a
   CONTRIBUTOR steering the design) that asks a question or gives
   direction about the change.
4. **Plan.** Note the planned change, every deliverable it commits to,
   the files it names, its not-in-scope line, its deviation or
   deferral notes, and every case and command in its test plan.
5. **Diff and commits.** Build the hunk ledger (next section).
6. **Test evidence.**
7. **Title and description.** Last.

## Evidence gathering

Fill these notes. Quote short phrases where you can; "absent" is a
valid note.

- **N1 Asks.** From step 2: each template section, checklist item, and
  required file, and the AI-policy case.
- **N2 Maintainer direction.** From step 3: each maintainer question
  or direction, with the author's role.
- **N3 Plan.** From step 4: planned change; deliverables (numbered);
  files; not-in-scope; deviation and deferral notes; test-plan cases
  (each failing case and its expected result) and test-plan commands.
- **N4 Hunk ledger.** From the diff: one line per hunk with its file,
  what it does, and a tag: `planned`, `test`, `housekeeping`
  (changelog, release-notes entry, i18n string, regenerated file),
  `disclosed` (matches a deviation or deferral note in the plan or
  description), `unplanned`, or `debris` (debug leftover, commented-out
  code, dead code, unrelated TODO, or a hunk with no fix line that only
  reformats, re-indents, or reorders). A hunk may carry two tags
  (for example `planned` plus a `debris` line inside it). Note the
  commit messages beside the hunks they map to.
- **N5 Deliverable coverage.** For each N3 deliverable: the hunk that
  delivers it, or "missing" plus whether a deviation note or the
  description discloses it.
- **N6 Test coverage.** For each N3 failing case: where the evidence
  re-runs it (quote the command and the after output) or which added
  test encodes it, and whether the before state is shown or quoted.
  Flag any evidence that runs a different path (a control, a simpler
  case, another input). For each N3 command: shown with its outcome,
  or absent.
- **N7 Description claims.** Each factual claim in the title and
  description (what changed, entries included, tests added, ticked
  boxes, "exactly", "in full", "no other changes"), paired with the N4
  hunk that makes it true, or "not in diff".
- **N8 Standards.** For each N1 item: where the description or diff
  meets it, or "absent". For the AI case: whether a disclosure is
  present. For each N2 entry: the plan or description phrase that
  engages it, or "not engaged".

## Check execution

Run all five checks, in rubric order, every time. Do not stop at the
first fail.

1. **plan-fidelity** from N4 + N5. Any hunk tagged `unplanned`: fail,
   and name the file and what it does. Any deliverable "missing" with
   no disclosure: fail. Otherwise pass.
2. **description-matches-diff** from N7. Any claim "not in diff", or a
   fidelity claim ("exactly", "no other changes") while N4 has an
   `unplanned` hunk: fail, and quote the claim. Otherwise pass.
3. **test-evidence-decisive** from N6. Every failing case covered by a
   re-run with its after result shown or by an added test of that
   exact case, the before state shown or quoted, and every named
   command shown with its outcome: pass. A case with no coverage, a
   different-path run standing in for the trigger, a claim with no
   observable, or a named command never shown: fail, and name what is
   missing.
4. **diff-reviewable** from N4. Any `debris` tag: fail, and quote the
   line or describe the hunk. Otherwise pass.
5. **standards-met** from N8. Any required ask "absent", a missing
   disclosure under a requires-disclosure policy, or a maintainer
   direction "not engaged": fail, and name it. Otherwise pass.

When evidence is absent: in eval mode the bundle is the whole world,
so a missing before/after, file, or section is a `fail` on the check
that needs it, not `unclear`. Use `unclear` only when the package
holds the evidence but it supports two readings the rubric and the
guide's examples cannot settle; say what the two readings are.

Once a check's notes are filled, grade it from the notes. Re-read the
package only to settle an `unclear` or fetch an exact quote.

## Verdict assembly

1. Apply the rubric's verdict rule: `accept` only if all five checks
   are `pass`; any `fail` or `unclear` makes it `reject`.
2. The deciding check for a reject is the first failing check in
   rubric order. Name it first in the summary, and quote the exact
   phrase or hunk that failed it.
3. Each check's `evidence` field is one line: the fact or quote that
   decided it.
4. Write the summary (one line per check; the deciding check; any
   procedure gaps; live-mode voice-guide notes), then emit the JSON
   block from `SKILL.md`, last, with the five checks in rubric order.
