# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72

**Verdict output**

```
[Live-mode read-out: ranked, all three accepted]
1. #72 (accept) - FastAPI-side security fix in core/security.py, ~1-2 hours
2. #70 (accept) - ingestion parser test fixture, tier-2
3. #73 (accept) - README / .env.example docs and config mismatch

All checks pass for all three: repo-alive, maintainer-responds, scope-bounded, unclaimed, ai-policy-allows.
```

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "not archived; newest default-branch commit 2026-09-16 (4 days before 2026-09-20)"},
      {"name": "maintainer-responds", "grade": "pass", "evidence": "clause (c): all 5 recent commits human-authored (Aburke225), newest 2026-09-16, oldest 2026-08-18; clause (b): same-day maintainer replies on #52 and #43"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "one bug fix in core/security.py plus removing one xfail marker; opened by COLLABORATOR, labeled good first issue, tier-1"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: none; 0 PRs in repo; 0 comments; two classmate fork-commit references are not PRs or claims, and Path Review house rule ignores classmate claims"},
      {"name": "ai-policy-allows", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, PR template and README contain no AI statement; AGENTS.md and AI_POLICY.md return 404"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/70",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "not archived; newest default-branch commit 2026-09-16 (4 days before 2026-09-20)"},
      {"name": "maintainer-responds", "grade": "pass", "evidence": "clause (c): all 5 recent commits human-authored (Aburke225), newest 2026-09-16, oldest 2026-08-18; clause (b): same-day maintainer replies on #52 and #43"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "one fixture fix across 3 named files, cause stated (fixture indented four spaces), remove one xfail marker; opened by COLLABORATOR, tier-2"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: none; 0 PRs in repo; 0 comments; timeline shows only label and rename events"},
      {"name": "ai-policy-allows", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, PR template and README contain no AI statement; AGENTS.md and AI_POLICY.md return 404"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/73",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "not archived; newest default-branch commit 2026-09-16 (4 days before 2026-09-20)"},
      {"name": "maintainer-responds", "grade": "pass", "evidence": "clause (c): all 5 recent commits human-authored (Aburke225), newest 2026-09-16, oldest 2026-08-18; clause (b): same-day maintainer replies on #52 and #43"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "make README.md and .env.example agree on the LLM API key; two files, opened by COLLABORATOR, labeled good first issue, tier-1"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: none; 0 PRs in repo; 0 comments; timeline shows only label events"},
      {"name": "ai-policy-allows", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, PR template and README contain no AI statement; AGENTS.md and AI_POLICY.md return 404"}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

**Run history**

1. 18/20 (full run). Two disagreements: issue-19 (gold accept, my rubric rejected on `scope-bounded`) and issue-15 (gold reject, my rubric accepted).
2. 20/20 (full run, saved as `eval-run.txt`), after revising `scope-bounded`.

Between the two full runs I also did a 3-issue smoke run (`--limit 3`, 3/3) and two `--only` runs (4/4 on issue-12, 14, 09, 20; 5/5 on issue-15, 19, 05, 09, 10). These were partial, so they don't count as runs of record. The last score above, 20/20, matches the agreement line in `eval-run.txt`.

**Issue analysis**

issue-19 (zxcalc/zxlive#517, "Selecting large subgraphs in proof mode freezes the UI"). Gold label: accept. My first rubric: reject, failed on `scope-bounded`.

The issue body lists "two potential causes which should be fixed" (slow matchers, UI waiting on the matching thread) and three "Additional suggestions" (multi-processing, only match expanded categories, apply rewrites in a separate thread). My first `scope-bounded` wording failed an issue that was "explicitly an umbrella or tracking issue (a list of sub-items meant to be split into separate work)". A numbered list of causes plus suggestions looks like sub-items on the surface, so the check rejected it. But it is one reported bug, "Type: bug, Priority: High", opened by a COLLABORATOR, and the list is the maintainer's diagnosis of that single bug, not a set of separate tasks. The gold label accepts it for that reason. I fixed the rubric, not the run: I added an explicit carve-out for a bug that enumerates its causes.

**Check rationale**

`scope-bounded`, as currently written in `rubric.md` (two clauses quoted):

> "the issue has been open more than 365 days AND has two or more abandoned attempts behind it (closed unmerged linked PRs, or PRs mentioned in the thread that never merged)"

> "One reported bug stays one bounded piece of work even when a maintainer enumerates its likely causes or lists optional follow-up ideas inside it — that is a diagnosis, not an umbrella."

Why: the second clause is the fix for issue-19 above. The first clause is the fix for issue-15 (zulip#19589), which my first run accepted although gold says reject. It carries a `good first issue` label, but has been open since 2021 with 97 comments and two closed, unmerged linked PRs (#20840 and #23123). The original wording had no way to see "years of history plus abandoned attempts", so a friendly label got it through. Requiring both an age over 365 days and two or more abandoned attempts is deliberately conjunctive, so an old issue alone, or one failed attempt alone, does not reject.

**Trade-offs**

The carve-out for a bug that lists its causes gives up some protection against a real umbrella that is dressed up as a diagnosis. A maintainer could write a genuine multi-task issue in the same "causes and suggestions" shape and my check would let it through.

I checked the damage with `--only issue-15,issue-19,issue-05,issue-09,issue-10`: 5/5 agreed, so the carve-out did not stop the check from rejecting the true umbrellas (issue-05, issue-10) or the endorsed feature (issue-09). Then a full run gave 20/20. I accept that this check will miss a well-disguised umbrella that has no abandoned-attempt history.

---

## Selection rationale

**Selection rationale**

1. Fit and time: #72 is a bug fix in `core/security.py`, a FastAPI-side password check that should fail closed and return `False` when the stored hash is malformed. I work mostly in Python and FastAPI, so it fits. The maintainer estimates 1 to 2 hours, and the work is one fix plus removing one `@pytest.mark.xfail` marker. It does not touch the growth areas I listed (async, serving, performance), so it's a safe first contribution rather than a stretch.
2. What the verdict got right, and what I weighed beyond it: the verdict correctly saw an active repo (last commit 4 days ago), a maintainer-opened bounded bug with a good-first-issue label, no assignee, no PRs, and no AI-policy restriction. What the rubric could not weigh: my own fit and the hours I have, that two classmates' fork commits already reference #72, and that the repo has zero PRs so far, so I have no evidence of how fast the maintainer reviews.
3. Anticipated difficulty in claiming it: #72 is popular, so I expect to share it with other students. The house rule says a shared issue is fine because credit attaches to the PR. The practical risks are that CI fails with `XPASS(strict)` unless I remove the xfail marker, and that my first fork PR may wait for a maintainer to approve CI.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
