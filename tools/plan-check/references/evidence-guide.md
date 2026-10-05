# Evidence guide: where evidence lives in a plan package

This is the rubric's map. For each evidence family it says where to
look (in an eval bundle, and live on GitHub or in the draft), what
good looks like, and gives worked pass/fail examples. The examples are
invented scenarios, not eval packages: learn the pattern, not the
names.

An eval bundle has these parts, in this order: the header (`source`,
`captured`), `Repo facts`, `Issue`, `Thread highlights`,
`Repro evidence`, `Candidate plan`, `Candidate plan comment`. In eval
mode the bundle is the whole world: quote it, fetch nothing.

One rule runs through every family: grade the substance, never the
shape. Headings, length, and a confident tone are not evidence. A
six-line plan that names the cause, the file, the limit, and the
expected output is ready; a polished two-page plan that blames the
wrong component is not.

## Diagnosis and grounding

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Stated cause | the `Candidate plan`'s "Diagnosis" / "Cause" text, or the first sentences that say what is wrong and where | the draft `plan.md`'s diagnosis |
| Failing run | the `Repro evidence` step that shows the bug, plus its `Actual:` line | the student's posted repro comment on the issue |
| Controls | every run in `Repro evidence` labelled "Control", every step that varies one thing (flag removed, other version, other build, feature off, a native/upstream tool run directly), and any `--debug` / trace / log step | same, in the posted repro comment |
| Thread causes | causes claimed in `Thread highlights` (by anyone) | the issue thread |

What good looks like:

- Build the control ledger first: for each control, write what changed
  and what happened. Then ask, for the plan's cause: "if this cause
  were true, what would this control show?" Every control must show
  what the cause predicts. One contradiction fails the check.
- The two classic contradictions: (1) a control shows the blamed
  component working (same component, run without the trigger or in
  another context, behaves correctly); (2) a trace step shows the
  damage already present before the blamed code runs, or the failure
  happening with the blamed code out of the loop.
- A cause copied from the thread gets the same test. A confident
  thread comment, an open PR, or a "root cause" label never outranks
  the package's own controls.
- Consistent but unproven is fine. The plan does not have to prove the
  cause; it must not be contradicted, and it must be a cause (where
  and why), not a restatement of the symptom.
- The change must act on the cause. Documentation, a user-side
  workaround, or a blanket recover/try-catch around the crash is not
  a fix when the evidence localizes a code defect.

**Example D1 (fail): a control shows the blamed component working.**
Issue: `notes sync --dry-run work/` silently skips every file whose
name contains a space, but only when `--dry-run` comes before the
path. Repro: failing run; control A: `--dry-run` after the path, files
with spaces are listed; control B: no `--dry-run`, files with spaces
sync; trace: the argument parser binds the path to `--dry-run`'s
optional value. Plan: "the filename-escaping function in `paths.py`
mishandles spaces; rewrite it." Control B sends the same names through
the same escaping function and they sync, so escaping is not the
defect, and the trace points at argument parsing. Fail.

**Example D2 (fail): the plan adopts a thread claim the evidence rules
out.** Issue: exporting a 500-page document to PDF takes 40 s. Thread:
a commenter says "root cause is the font cache being rebuilt; fixed in
my PR." Repro: step 2, same export with images stripped: 2 s; step 3,
font cache pre-warmed and images kept: 39 s. Plan: "as identified in
the thread, stop the font-cache rebuild." Step 3 shows the full cost
with the cache already warm, and step 2 pins the images. Fail, however
polished the plan.

**Example D3 (pass): consistent with every control, with an honest
unknown.** Issue: the sidebar badge count stays stale after a sync.
Repro: failing run; control A: reopening the sidebar shows the right
count; control B: the previous release refreshes live. Plan: "after
sync, the sidebar model is not invalidated, so it keeps pre-sync
counts until rebuilt; I have not confirmed whether the invalidation is
dropped in the sync callback or in the sidebar's subscription, and
will confirm with the debug log before choosing." Reopening rebuilds
the model (fits A); a regression in the invalidation path fits B.
Unproven, uncontradicted, named. Pass.

**Example D4 (fail): workaround or suppression in place of the fix.**
Issue: the image resizer throws `IndexError` on PNGs with a palette of
exactly 256 colours; the repro isolates the palette size and a trace
lands in one off-by-one loop in `palette.py`. Plan: "catch exceptions
around `resize()` and fall back to copying the original image." The
evidence localizes a code defect; the plan hides the symptom and
leaves the defect. Fail. (Same verdict for "document the workaround in
the FAQ" when the repro shows a code-level culprit.)

## Scope

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Proposed changes | every numbered step, "Proposed changes" item, "Change:" line, and "while in there / while touching" clause in the `Candidate plan` | the draft `plan.md` |
| Stated limits | "Not in scope", "Out:", "Deferred", "leave for a follow-up", "file separately" lines in the plan | same |
| The behavior to fix | the issue title/body and the repro's `Expected:` / `Actual:` lines | the issue and the posted repro comment |

What good looks like:

- List every change item. Tag each one: fix (acts on the reproduced
  behavior), sibling (the same defect pattern at another site in the
  same code path), test, or extra. One `extra` fails the check.
- Extras: refactors and module splits, redesigns ("state machine",
  "unified abstraction", "rebuild on a proper model"), new options /
  settings / props / UI, dependency upgrades and library migrations,
  new CI jobs or matrices, fixes for other symptoms, and open-ended
  "maybe also check other X".
- A right core fix does not rescue a bundle. "Splitting it into a PR
  series" does not rescue it either: the plan still commits to all of
  it.
- The plan states its limits, in any words. Deferring a larger rework
  with a reason, filing an adjacent symptom as its own issue, or
  "noting other suspicious sites in the PR without fixing them" are
  signs of a bounded plan, not gaps.

**Example S1 (pass): one fix plus siblings plus a limit.** Issue: the
CSV exporter writes a stray delimiter when the last column is empty.
Plan: guard the trailing-delimiter write in `write_row()`, apply the
same guard to the header writer, which shares the loop, and add the
repro row as a regression test. "Not in scope: quoting rules or the
dialect options. If I spot other empty-field quirks I will list them
in the PR rather than fix them here." Pass.

**Example S2 (fail): right fix, wrong bundle.** Issue: dates in the
export are off by one day for users east of UTC; the repro isolates a
naive conversion in one function. Plan: (1) convert with the user's
timezone there (the direct fix); (2) while in the area, replace the
date library across the codebase; (3) add a timezone picker to
Preferences; (4) add a caching layer for formatted dates. Items 2-4
are extras the issue never asked for. Fail, even though item 1 is
exactly right.

**Example S3 (fail): the fix is buried in a redesign.** Issue: one
error branch in the uploader leaks a file handle; the repro shows the
success path closes it. Plan: "resource handling needs a real owner":
introduce a resource-manager framework for every I/O call, add a
`keepOpen` config flag, restructure the uploader into modules, and
migrate its tests to a new harness. Fail: one missing `close()`
became a rewrite.

**Example S4 (pass): honest deferral.** Thread discusses two fixes for
a search that misses accented names: normalize the query string, or
rebuild the index with a new collation. Plan: "normalize the query in
`search()`. Deferred: the index rebuild, which may be the better
long-term shape but is more than this bug needs; the mobile client's
copy of this code, which I cannot test, flagged in the PR for someone
who can." Pass: arguable deferrals, stated with reasons, are bounded.

## Executability

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Location | file paths, function or type names, or a named area in the `Candidate plan` ("Files:", "In scope:", approach steps) | the draft `plan.md` |
| Chosen change | the approach steps: what will be done at that location | same |
| Open decisions | "or", "maybe", "somewhere", "not sure", "whichever", "TBD", question marks inside the plan | same |

What good looks like:

- The stranger test: could someone who has never seen the thread open
  the named file or area and start making the named change, after one
  read, without asking the author? Location plus chosen change is the
  bar.
- A bounded unknown passes when the area and the approach are fixed:
  "exact function to pin with the debug log I already have", "the
  check may sit one layer up; same change either way".
- Investigation is not a plan. Steps that only say how to find out
  (profile, investigate, look into, experiment, figure out) with no
  change chosen fail.
- A decision deferred to build time fails: two unlike options left
  open ("in our fork or upstream, whichever is easier"), or "somewhere".

**Example E1 (pass): file, function, change.** "In
`lib/auth/session.rb`, `expired?`: compare against
`issued_at + SESSION_TTL` instead of `updated_at + SESSION_TTL`." A
stranger can start. Pass.

**Example E2 (pass): area named, function to pin, approach fixed.**
"Change goes in the plugin loader's reload path: unregister the old
plugin's hotkeys before registering the new ones. Exact function to be
pinned from the `--verbose` log I already have, which shows where
registration happens." Pass: the change is chosen, only the line is
open, and the plan says how to find it.

**Example E3 (fail): investigation with no chosen change.** "Profile
the dashboard load, look into whether the ORM is the problem, consider
caching, and optimize whatever turns up hot." No file, no change.
Fail.

**Example E4 (fail): decisions deferred.** "Add a null check
somewhere before rendering; fix the parser either in our fork or the
upstream library, whichever is easier; maybe also look at the other
renderers." Every real decision is left for build time. Fail.

## Test plan

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Test plan | the `Candidate plan`'s "Test plan" / "Test:" text, plus tests named in approach steps | the draft `plan.md` |
| Trigger to re-run | the failing step in `Repro evidence` | the posted repro comment |
| Observable target | the repro's `Expected:` line | same |

What good looks like:

- At least one test re-runs the reproduced trigger, by hand or encoded
  as an automated test of that exact case, and says what will be seen
  after the fix: a specific output, exit code, value, state, or the
  named symptom being absent on that run.
- "Expected after" must be stated, not implied: "re-run the repro" with
  no stated result is weaker than "re-run the repro; prints X, exit 0".
  Absent-symptom targets count when specific ("no leftover handle in
  `lsof` after 20 runs").
- Generic suites are support, not proof. "Run the full test suite and
  make sure nothing regresses" proves nothing about this bug, even in
  an otherwise excellent plan.
- Adjectives are not observables: fast, better, correct, works,
  nothing else broken.

**Example T1 (pass): repro re-run with exact output.** "Re-run the
repro: `calc --round 2 1.005` prints `1.01`, exit 0; the control
(`1.004`) still prints `1.00`; add both as regression cases." Pass.

**Example T2 (pass): absence on a specific run.** "Run the repro
upload 20 times: `lsof` shows no leftover handle to the temp file
after any run; the success-path control is unchanged." Specific run,
specific symptom, stated absence. Pass.

**Example T3 (fail): suite only, in a strong plan.** A bounded,
grounded, thread-aware plan whose test plan is "run `npm test` and
make sure CI stays green." Nothing observes the fix itself. Fail on
this check, whatever the other checks say.

**Example T4 (fail): a feeling.** "After the change the dashboard
should feel snappy and the profiler output should look much better."
No number, no target. Fail.

## Honesty

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Comment claims | each sentence of the `Candidate plan comment` that promises a change, test, result, scope, or certainty | the draft plan comment |
| What the plan holds | the `Candidate plan`: changes, limits, tests, risks, unknowns | the draft `plan.md` |
| What the evidence shows | the `Repro evidence` block | the posted repro comment |

What good looks like:

- Read the comment one sentence at a time and point each promise or
  claim at the plan line or evidence line that holds it. Summaries and
  omissions are fine; additions and upgrades are not.
- Upgrades to watch: "I traced the root cause" when the plan says
  "likely"; "verified" when nothing was verified; "one contained
  change" over a plan that touches several areas; a test or follow-up
  the plan never mentions.
- Unknowns stay unknowns: if the plan flags an open question, the
  comment either flags it or stays silent on it; it never states the
  opposite.
- No fixed dates or guarantees. "PR by Friday", "this week",
  "guaranteed" fail. "Will send the PR shortly" or "will report back
  once the tests pass" are intent, and pass.

**Example H1 (pass): faithful summary.** Plan: fix the retry-counter
reset in `queue/worker.go`, a unit test for the reset, the related log
noise filed separately, unsure whether the reset belongs in the worker
or the dispatcher. Comment: "Plan: reset the retry counter in the
worker, with a unit test; I'll file the log noise separately. It may
belong one layer up in the dispatcher; I'll confirm while
implementing." Every sentence is held by the plan. Pass.

**Example H2 (fail): certainty the plan does not hold.** Plan: "the
cause is likely the cache layer; not yet confirmed." Comment: "I traced
the root cause to the cache layer and verified the fix." Nothing
verified a fix. Fail.

**Example H3 (fail): scope shrunk in the telling.** The plan lists four
change areas across two packages. Comment: "a small, contained fix to
one file." Fail: the maintainer would approve something other than
what will be built.

**Example H4 (fail vs pass): timing.** "I'll have the PR up by Friday"
fails. "I'll report back once the regression test passes" passes.

## Comms

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Maintainer-side direction | `Thread highlights` entries by OWNER, MEMBER, COLLABORATOR, or a CONTRIBUTOR who opened the issue or is steering its design: a named culprit or fix site, a settled or rejected approach, a posted patch or test build, a "works as intended" ruling | the issue thread; the author badge next to each name |
| Competing work | open PRs and patches named in the thread | linked PRs in the issue sidebar |
| AI policy | the `contribution policy` line in `Repo facts` | `CONTRIBUTING.md`, `AI_POLICY.md` or similar, PR templates |

What good looks like:

- Find every maintainer-side direction first. The plan or comment must
  engage each one: follow it, build on it, or name it and say why it
  diverges. Silence fails, and so does swapping in a different remedy
  without a word about the maintainer's located culprit or test build.
- No maintainer direction in the thread: the thread part passes.
  Non-maintainer comments are context, not direction. An open PR from
  another contributor is worth engaging, but ignoring it alone does
  not fail.
- AI policy, three cases. Treat every package as AI-assisted work.
  - **Requires disclosure of all AI use** (any form, including issues
    and comments): the comment must say AI was used. Silence fails,
    even under a perfect plan.
  - **Conditions only** (human in the loop, understand the work,
    comments in your own words, disclosure asked in the PR but not in
    comments): no disclosure line is needed in a plan comment. Fail
    only if the comment visibly breaks a condition.
  - **Silent or permissive**: passes.
- Bug-report template asks belong to bug reports. Never fail a plan
  comment on them.

**Example C1 (fail): maintainer direction ignored.** Thread: a MEMBER
reproduced, named `lexer.c:scan_heredoc()` as the culprit, and pushed
a draft branch asking for testers. Plan and comment: "users can avoid
this by quoting the delimiter, so I'll add a lint warning and a docs
note." Neither mentions the named culprit or the draft branch. Fail.

**Example C2 (pass): direction followed and cited.** Thread: an OWNER
rejected "deep-copy the config on every read" as too slow and proposed
copy-on-write. Plan: copy-on-write in `Config.set()`, "following the
direction in the thread". Pass.

**Example C3 (pass): divergence named.** Thread: a COLLABORATOR listed
three options. The plan picks option 2, says why (smallest change,
same approach the sibling command already uses), and defers option 1
with a reason. Engaged. Pass.

**Example C4 (the three AI-policy cases, same comment).** Comment: a
good, specific plan comment with no AI mention. Repo A: "all AI usage
in any form must be disclosed": fail. Repo B: "AI-assisted code
welcome; state the tool in the PR; comments in your own words": pass,
no line needed in the comment. Repo C: no AI policy: pass.
