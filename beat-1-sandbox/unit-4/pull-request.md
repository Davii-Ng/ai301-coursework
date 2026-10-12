# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

---

## Your pull request

**Pull request**

TODO: paste the PR's own page here (https://github.com/codepath/pathreview-ai301-fa26-s3/pull/<number>)

**Branch**

fix/72-verify-password-unknown-hash

## Eval iterations

**Run history**

1. Run 1: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. This was the first full run of my installed tool at `~/.claude/skills/pr-precheck` (all 24 packages including calibration, all agree).
2. Run 2: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. This was the confirming full run after I rewrote example phrases in the rubric, procedure and evidence guide that echoed scored packages. This is the run in `eval-run.txt` (written 2026-10-12T01:00:07Z, `rubric.md sha256:4e45562f7ddf46ec`).

**Package analysis**

pkg-07 (golangci/golangci-lint#6671). My rubric decided reject, and the gold label is reject. The fix itself is fine: it groups suggested-fix edits by the file each one targets. Four checks passed, and `test-evidence-decisive` failed with this evidence: "Only the one-file control is re-run ('Verified the fix on a single-file case'); the two-file repro and go build ./... are never shown, and the added test checks two edits in a.go only."

My rubric reads it that way because the check requires the evidence to re-run "every failing case the plan's test plan names (the reproduction's own trigger, not a nearby path)", and it fails when "it exercises a different path from the reproduction (a control, a simpler case, a different input)". The plan's repro is a two-file module where `--fix` corrupts `a.go`. The one-file case is the plan's own control: it worked before the fix too, so passing it proves nothing about the fix. The added unit test can't stand in for the repro either, because it only checks two edits in `a.go` and never covers the cross-file case.

**Check rationale**

> | diff-reviewable | Every hunk in the diff and the commit list. Evidence guide: "Diff quality". | Passes when every hunk carries part of the change or its tests, and nothing unrelated rides along. Fails if the diff contains any of: a leftover debug statement (debug print, log, or commented-out debug line); commented-out code blocks or earlier attempts kept "for reference"; dead code (an unused function, an unused-symbol suppression added to keep scaffolding); a hunk that only reformats, re-indents, reorders imports, or reflows code the fix does not touch (lines removed and re-added unchanged); or a new TODO unrelated to the fix. Unchanged context lines inside a hunk that also carries the fix are not churn. Commit messages alone never fail this check, but a "tmp" or "misc" commit is a cue to look for debris. | required |

It reads this way because of three decisions.

- **Context lines inside a fix hunk.** I rejected a stricter rule, "any line removed and re-added unchanged is churn". Some correct diffs show unchanged lines removed and re-added around a one-argument change. pkg-02 (gold accept) does this in its `save_html` hunk, where only `clear=clear` changes to `clear=False`. The stricter rule would have failed it. The sentence about unchanged context lines inside a fix hunk keeps those lines out of the churn count. The churn clause only fires on a hunk with no fix line in it at all.
- **Commit messages are cues, not fails.** A "wip" commit over a clean diff is still a clean diff. The check grades the hunks; the commit names only tell the grader where to look.
- **Examples changed between runs 1 and 2.** In run 1 the examples were `"wip" or "cleanup while debugging"` and `` `allow(dead_code)` ``, both too close to text in the scored packages. I replaced them with neutral ones (`"tmp" or "misc"`, "an unused-symbol suppression added to keep scaffolding") so the rubric teaches the pattern, not the answer key.

**Trade-offs**

The case I accept `diff-reviewable` will miss: because context lines inside a fix hunk are exempt, a contributor could slip a small reformat of a neighbouring line into the same hunk as the fix, and the check would pass it. I accept that miss, because the stricter version fails a clean gold-accept PR (pkg-02). A one-line reformat inside the fix's own hunk costs a reviewer far less than a whole churn hunk, which this check still catches. pkg-12, pkg-15 and pkg-18 all fail it on standalone churn or debris.

Nothing else changed between run 1 and run 2, and here is how I know. Both full runs graded all 24 packages (20 scored plus 4 calibration). Every package got the same verdict in both, and both printed the same category line: `clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. The edits between them only swapped example wording; no pass or fail condition changed.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
