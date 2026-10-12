# Evidence guide: where evidence lives in a PR package

This is the rubric's map. For each evidence family it says where to
look in an eval bundle and in a live PR, what good looks like, and
gives worked pass/fail examples. The examples are invented scenarios,
not eval packages: learn the pattern, not the names.

An eval bundle has these parts, in order: header (`source`,
`captured`), `Repo facts`, `Issue`, `Thread highlights`,
`Plan context` (repro evidence plus the accepted plan), and
`Candidate PR` (`Title`, `Description`, `Commits`, `Diff`,
`Test evidence`). In eval mode the bundle is the whole world.

In live mode the same parts come from: `plan.md` (with its
`## Deviations` section), `git diff main...HEAD` and
`git log --oneline main..HEAD`, `pr_draft.md` (first line is the
title, the rest is the description), `test_evidence.md`, the issue
thread on GitHub, and the repo's `.github/PULL_REQUEST_TEMPLATE.md`
and `docs/CONTRIBUTING.md`.

One rule runs through every family: grade the artifact, not the
write-up. A four-line description over a clean one-hunk diff with a
shown before/after can be ready. A polished description with a full
checklist can sit on top of a diff that does something else.

## Plan fidelity (harness category: silent-drift)

| Signal | In the eval bundle | Live |
|---|---|---|
| Planned change, files, limits | `Plan context`: the "Plan:" text, its "Scope", "Not in scope", and "Files" lines | `plan.md`: Scope, Files and approach |
| Deviation notes | `Plan context` lines that say something was deferred, dropped, or changed; deferral statements in the `Description` | `plan.md` `## Deviations`; the description's notes |
| What actually changed | `Diff`: every `--- a/` / `+++ b/` file and every `@@` hunk; `Commits` | `git diff main...HEAD`; `git log --oneline main..HEAD` |
| Fidelity claims | `Description` phrases: "exactly as planned", "in full", "no other changes", "X added", "Y included" | `pr_draft.md` |

What good looks like:

- List every file and hunk in the diff. Tag each one: planned change,
  test, housekeeping the repo asks for (changelog,
  release-notes entry, i18n string, regenerated file), disclosed
  deviation, or unplanned. One unplanned hunk fails plan-fidelity.
- Then list every deliverable the plan commits to, and find each one
  in the diff. A missing one fails, unless the plan's deviation notes
  or the description say it was left out.
- Commit messages are a map, not proof. A commit named "add new
  option" or "simplify helpers" tells you which hunk to check.
- Then read every factual claim in the description against the diff.
  "Changelog entry added" with no changelog file in the diff fails
  description-matches-diff. So does "no other changes" over a diff
  with other changes.
- Disclosed shortfalls pass. "Deferred X, see the plan's note" in both
  the plan and the description is honest scope, not drift.

**Example F1 (fail, more than the plan): a feature rides along.**
Plan: in `export.py`, write the temp file before deleting the old one;
not in scope: any new option. Diff: that change, plus a new
`--keep-backup` flag in `cli.py`, plus a rewrite of an unrelated
`format_size()` helper. Description: "Implements the plan exactly."
Fails plan-fidelity (two unplanned hunks) and description-matches-diff
("exactly" is false).

**Example F2 (fail, less than the plan): a promised deliverable is
missing.** Plan: two deliverables, a retry on timeout and a new
metric counting retries. Diff: the retry and its test only.
Description: "adds the retry and the retry-count metric." No metric
appears anywhere in the diff, and no deviation note mentions it. Fails both
checks.

**Example F3 (pass): a disclosed deferral.** Plan: reject negative
page sizes in the API; its deviation note says zero-size pages are not
handled yet, and why. Diff: the negative-size check and a test.
Description restates: "Zero is deliberately left as-is; see the
plan's note." Passes both checks.

**Example F4 (pass): housekeeping the repo asks for.** Repo asks for a
changelog entry on every fix. Plan lists only `parser.c` and a test.
Diff: `parser.c`, the test, and one `CHANGELOG.md` line naming the
fix. The changelog line is required housekeeping, not drift. Pass.

## Test evidence (harness category: not-tested)

| Signal | In the eval bundle | Live |
|---|---|---|
| What must be proven | `Plan context`: the repro evidence's failing commands and the plan's "Test plan" (every case it names, every suite it names) | `plan.md` Test plan; the posted repro comment |
| What was proven | `Test evidence` section; before/after blocks sometimes sit in the `Description` | `test_evidence.md`; the Testing section of `pr_draft.md` |
| Tests added | test files in the `Diff` | test files in `git diff main...HEAD` |

What good looks like:

- List every failing case the plan's test plan names. For each, find
  either (a) a re-run on the changed code with its observable result
  shown (output, exit code, value, visible state), or (b) an added
  test that encodes that exact case and asserts the expected result,
  with the suite run shown passing. A case with neither fails.
- The re-run must hit the reproduction's own trigger. A nearby path
  that worked before the fix too (a control, a simpler case, a
  different input) proves nothing about the fix.
- The before state may be shown, or quoted from the repro report.
- Every suite or check command the plan's test plan names appears
  with its outcome. Controls that should stay unchanged may be stated
  in words.
- "Works on my end", "used it all week with no problems", and "the
  suite is green" with nothing re-run are claims, not evidence.

**Example T1 (pass): repro re-run with before and after.** Plan's test
plan: re-run `calc --round 2 1.005`, expect `1.01`; run `make test`.
Evidence: `calc --round 2 1.005` with "before: 1.00" and the after
output `1.01`, exit 0; `make test`: "212 passed". Pass.

**Example T2 (fail): the control, not the bug.** The bug only appears
when an edit spans two files. Evidence: the one-file case re-run,
which already worked, plus a new unit test on one file. The two-file
trigger is never re-run or tested. Fail.

**Example T3 (fail): a claim without an observable.** "Tested on my
machine, the sync works now, and the suite is green." No command, no
output, no before/after. Fail.

**Example T4 (fail): one of two named cases missing.** Plan names two
failure modes, A and B, with expected results for each. Evidence
re-runs A with output; the added test covers only A; B is never
mentioned. Fail, even though A is decisive.

## Diff quality (harness category: unreviewable)

| Signal | In the eval bundle | Live |
|---|---|---|
| The change itself | `Diff` hunks | `git diff main...HEAD` |
| Debris cues | `Commits` messages ("tmp", "misc", "more fixes", "format") | `git log --oneline main..HEAD` |

What good looks like:

- Read every hunk and ask: does any line in it carry the fix or its
  test? If yes, the unchanged lines around it are context, not churn.
  If no, it is churn: a re-indent, an import reorder, a reflowed
  struct literal, or lines deleted and re-added unchanged.
- Debris tells, each one a fail on its own: a leftover debug print or
  log (`DEBUG`, `dbg!`, `console.log`, `print("here")`); a
  commented-out debug line; a commented-out earlier attempt "kept for
  reference"; an unused helper, or an unused-symbol suppression added to
  keep scaffolding; a new TODO unrelated to
  the fix.
- Comments that explain the fix are not debris.
- A commit message alone never fails the check. It tells you where to
  look.

**Example D1 (pass): one clean hunk.** Diff: three changed lines in
`limits.go` with a two-line comment citing the issue, plus a new test.
Pass.

**Example D2 (fail): debris around a right fix.** The fix is correct,
but the hunk also keeps `// println!("len = {}", len);` and a
commented-out first attempt, and adds `fn _probe_size()` that nothing
calls. Fail.

**Example D3 (fail): churn hunk.** Besides the fix, a hunk in the same
file deletes eight `print` lines and re-adds them unchanged, and the
import block is reordered. Neither hunk carries any part of the fix.
Fail.

**Example D4 (pass): context inside a fix hunk.** A hunk shows a call
removed and re-added over three lines because one argument changed
from `retries=0` to `retries=3`. The changed argument is the fix,
so the other lines are context. Pass.

## Standards and comms (harness category: standards-wall)

| Signal | In the eval bundle | Live |
|---|---|---|
| PR template and stated asks | `Repo facts` line `pull requests:` (template sections, checklist items, required entries) and the `contribution policy` line | `.github/PULL_REQUEST_TEMPLATE.md`, `docs/CONTRIBUTING.md` |
| AI policy | the `contribution policy` line | `CONTRIBUTING.md`, any AI policy file; for Path Review, the course's disclosure rule (Notes for Reviewers) |
| Maintainer direction | `Thread highlights` entries by OWNER, MEMBER, COLLABORATOR, or a CONTRIBUTOR steering the design | the issue thread |
| Where the PR answers | `Description`, `Title`, `Diff` (for required files) | `pr_draft.md`, the diff |

What good looks like:

- List every section and checklist item the template asks for, and
  every file the asks require (changelog, release-notes entry,
  issue-closing line). Find each in the description or diff. "n/a" with a reason
  counts. A visibly ignored required checklist fails.
- No template and no stated ask: that part passes.
- Treat every package as AI-assisted work, then read the AI policy:
  requires disclosure of all AI use: the description must name the
  tool and extent, or it fails; conditions only (human in the loop,
  own words): no disclosure line needed unless the policy asks for one
  in PRs; silent: passes.
- A maintainer's question or direction about the change gets engaged
  in the plan or the description: answered, followed, or named with a
  reason to diverge.
- Live mode, Path Review: the template's Testing checklist may leave
  the CI box unticked before the PR is opened. Every ticked box must
  be true of the evidence. The course requires an AI-use note in
  Notes for Reviewers.

**Example S1 (fail): required checklist ignored.** Template: a
required checklist with "links the issue", "tests added", "release
notes updated". Description: two paragraphs, no issue link, no
checklist, and the diff has no release-notes file. Fail, however good the fix and
evidence are.

**Example S2 (fail): disclosure required, absent.** Policy: "PRs must
say whether AI tools were used and for what".
Description: a strong technical write-up with no AI mention. Fail.

**Example S3 (pass): conditions-only policy.** Policy: "AI-assisted
code welcome; comments must be in your own words." Description: "This
change was AI-assisted; I reviewed every line and wrote this text
myself." Pass. (The same description without that sentence would also
pass, since the policy asks for no disclosure line.)

**Example S4 (pass): maintainer question engaged.** Thread: a MEMBER
asks whether the change should also cover the batch-import path.
Description: "Batch import deliberately left alone, per the
discussion with @member above; it retries on its own." Pass.
