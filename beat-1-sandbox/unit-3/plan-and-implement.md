# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

Davii-Ng

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5987630159

```
Plan for #72, from my reproduction above (https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5871767498, commit `2f4e82f`, Python 3.13.11, passlib 1.7.4).

Following the issue's direction: fail closed with `False` and drop the strict `xfail`.

**Cause:** `verify_password` (`core/security.py:37`) is a bare `return bool(pwd_context.verify(...))`. For a hash passlib can't identify, it raises `UnknownHashError: hash could not be identified`, and nothing catches it.

**Change:**
- `core/security.py`: catch `passlib.exc.UnknownHashError` in `verify_password` and return `False`.
- `tests/unit/test_security.py`: remove the strict `xfail` on `test_verify_with_wrong_hash_format`.

**Not touching:** `hash_password`, the `CryptContext` config, or `api/routes/auth.py`. That route already treats `False` as a failed login. I'm also not catching broader exceptions like `ValueError`.

**Test:** re-run my repro on branch `fix/72-verify-password-unknown-hash`. The direct call should print `False`, the covering test should pass, and `tests/unit/test_security.py` should go from `24 passed, 1 xfailed` to `25 passed`.

**Open question:** others above report `ValueError` for truncated `$2b$` hashes, and PR #78 catches it. I've since tried `''`, `None`, and a truncated hash on my branch; results and before/after output are in my next comment. I won't widen the catch without a reviewer saying so.

I used Claude Code to help draft this. I checked every claim against my own repro and the code.
```

---

## Your branch

**Branch**

fix/72-verify-password-unknown-hash

**Evidence**

Environment: Windows 11, Python 3.13.11, passlib 1.7.4, bcrypt 4.3.0, pytest 9.1.1, the same as my Unit 2 repro.

Before, on `main` at `2f4e82f`:

```
$ venv/Scripts/python -m pytest tests/unit/test_security.py -v -o addopts=""
================== 24 passed, 1 xfailed, 1 warning in 5.51s ===================

$ venv/Scripts/python -c "from core.security import verify_password; print(verify_password('password', 'not_a_valid_bcrypt_hash'))"
  File "...\venv\Lib\site-packages\passlib\context.py", line 1132, in identify_record
    raise exc.UnknownHashError("hash could not be identified")
passlib.exc.UnknownHashError: hash could not be identified

$ venv/Scripts/python -m pytest "tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format" --runxfail -o addopts="" -q
E           passlib.exc.UnknownHashError: hash could not be identified
1 failed, 1 warning in 0.22s
```

After, on `fix/72-verify-password-unknown-hash`. I ran these on the branch before committing; the change was then committed unchanged as `c8f424f`:

```
$ venv/Scripts/python -m pytest tests/unit/test_security.py -v -o addopts=""
======================== 25 passed, 1 warning in 6.44s ========================

$ venv/Scripts/python -c "from core.security import verify_password; print(verify_password('password', 'not_a_valid_bcrypt_hash'))"
False

$ venv/Scripts/python -m pytest "tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format" -o addopts="" -q
1 passed, 1 warning in 0.29s
```

Full unit suite (`pytest tests/unit -v -m unit`): `main` gives `375 passed, 53 xfailed`, and the branch gives `376 passed, 52 xfailed`. Only this issue's xfail flipped.

## Eval iterations

**Run history**

1. Run 1: no score. The harness crashed before grading anything (`FileNotFoundError: [WinError 2] The system cannot find the file specified`) because `subprocess` could not find `claude.cmd` on Windows. I fixed it with `shutil.which("claude")`, the same fix as in my Unit 2 harness.
2. Run 2: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. This was graded against `ai301-unit3-starter/skill/`.
3. Run 3: `agreement: 20/20 scored items  (bar: 18/20: PASS)`. This was graded against the installed copy at `~/.claude/skills/plan-check/`, after I replaced rubric examples I had quoted from scored packages with neutral ones. This is the run in `eval-run.txt`.

**Package analysis**

pkg-16 (pandas-dev/pandas#57666). My rubric decided reject, and the gold label is reject. The plan blames the pandas post-read cast (`_finalize_pandas_output`) and proposes re-padding the zeros. `diagnosis-grounded` failed with this evidence: "Plan blames the post-read cast, but repro step 4 shows the column is already int64 `1` 'before any cast to string could run'; step 3 shows string column_types preserves zeros." My rubric reads it that way because its pass condition makes the cause predict every control: "for each control, the outcome the cause predicts is the outcome the control shows." If the cast were the defect, step 4 (native pyarrow, no pandas cast) would still have the zeros, and it doesn't. The plan is long and confident, and it still fails, because the rubric grades the cause against the evidence and not the polish. The same package also failed `comment-faithful` (the comment states the contradicted cause as certain) and `thread-and-policy` (MEMBER jorisvandenbossche's explanation is not engaged).

**Check rationale**

> | test-decisive | The plan's test plan read against the repro evidence's failing step and its expected/actual lines. Evidence guide: "Test plan". | Passes when at least one test re-runs the reproduced trigger (the repro's command/steps, or an automated test encoding that exact case) AND states the expected observable result after the fix: an output, exit code, value, rendered state, or the named symptom being absent on that specific run ("prints `1.01`, exit 0", "the saved file opens and lists 3 rows", "no leftover temp-file handle across 20 runs"). Fails if the only tests are generic ("run the full test suite", "CI passes", "nothing regresses"), the expected result is a feeling or adjective ("should be snappier", "works correctly", "behaves better", "no other regressions"), or no test touches the reproduced case. | required |

It reads this way for three reasons.

- **Expected result required.** "Re-run the repro" is not enough on its own. The test plan has to state the expected result after the fix, because that is the "expected after stated" criterion.
- **Specific absence counts.** A named symptom being absent counts as an observable. I rejected a stricter version that required a positive output, because a correct test like "no stray sequences across 5 reattach cycles" (pkg-14, gold accept) would have failed it.
- **Generic suites fail.** The full-suite-only clause is there because calib-04 is a solid plan whose only test is `cargo test --workspace`, and nothing in that test observes the fix.

The quoted examples changed between runs 2 and 3. They originally used strings taken from scored packages (`"prints [[0,1,2,3,5,7,8,9]]"`, `"no rgb: strings across 5 reattach cycles"`). I replaced them with neutral examples so the rubric teaches the pattern instead of the answer key.

**Trade-offs**

`thread-and-policy` is the check that changes results. pkg-20 (ghostty) is an excellent bounded plan that passes every other check, and it is rejected only because the comment has no AI disclosure while ghostty's policy says "All AI usage in any form must be disclosed". Without this check, pkg-20 and pkg-04 both flip to accept, and the 2-package `thread-convention` category drops to 0/2, which fails the category floor.

The case I accept it will miss: the check only counts maintainer-side direction (OWNER, MEMBER, COLLABORATOR, or a steering CONTRIBUTOR). A plan that ignores a non-maintainer's open PR still passes. pkg-03 and pkg-08 engage open PRs, so nothing in the set fails because of this, but a real thread where a classmate's PR is the right answer would get through.

Nothing else moved between runs 2 and 3. I re-ran the full set rather than a `--only` canary, and every package kept the same verdict (both runs: `clear-accept 7/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`). That shows the example rewrite changed wording only, not what any check decides.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
