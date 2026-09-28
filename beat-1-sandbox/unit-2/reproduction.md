# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

## Your identity upstream

### GitHub username

Davii-Ng

## Posted upstream

### Claim comment

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5871672501

```
Hi, I'd like to take this one as a first contribution. The current `main` is at commit `2f4e82f` (2026-09-16), and I have not run anything yet, so I am not claiming a reproduction at this point.

As I understand it, `verify_password` in `core/security.py` lets passlib's `UnknownHashError` propagate when the stored hash is malformed, where it should fail closed and return `False`. There is already a covering test in `tests/unit/test_security.py` marked `@pytest.mark.xfail` (manifest ID H-05).

My next step is to run that xfail test on `2f4e82f` and confirm it fails with `UnknownHashError`. Then I will read how `verify_password` calls passlib to see where the exception should be caught, and I will report what I find here before opening a PR.
```

### Reproduction comment

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5871767498

````
Reproduction report for #72. **Result: reproduced** at commit `2f4e82f` (current `main`). `verify_password()` raises `passlib.exc.UnknownHashError` for the malformed stored hash in the `H-05` test, instead of returning `False`.

**Environment**

- Windows 11 (10.0.26200)
- Python 3.13.11 in a fresh venv
- passlib 1.7.4, bcrypt 4.3.0, pytest 9.1.1
- Code: commit `2f4e82f52efbcfcc57d65b3fa5348672163ca088`, no local changes
- The issue names no OS or Python version, so I can't say whether mine differs. The exception comes from passlib, so I don't expect it to depend on the OS, but I only tested on Windows.

**Steps**

```bash
git clone https://github.com/codepath/pathreview-ai301-fa26-s3.git
cd pathreview-ai301-fa26-s3
python -m venv venv
venv/Scripts/python -m pip install "passlib[bcrypt]>=1.7.4" "bcrypt>=4.0.1,<5.0.0" python-jose pytest pydantic pydantic-settings

# 1. As shipped: the strict xfail hides the error
venv/Scripts/python -m pytest tests/unit/test_security.py -v -o addopts=""
# -> 24 passed, 1 xfailed (test_verify_with_wrong_hash_format)

# 2. Call the function directly with the test's own inputs
venv/Scripts/python -c "from core.security import verify_password; print(verify_password('password', 'not_a_valid_bcrypt_hash'))"

# 3. Same test with the xfail marker ignored
venv/Scripts/python -m pytest "tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format" --runxfail -o addopts="" -q
```

(`venv/Scripts/python` is the Windows path; on Linux or macOS use `venv/bin/python`. `-o addopts=""` overrides the repo's default pytest options.)

**Output**

Step 1 shows the test is marked xfail, but not why it fails. Step 2:

```
  File ".../passlib/context.py", line 1132, in identify_record
    raise exc.UnknownHashError("hash could not be identified")
passlib.exc.UnknownHashError: hash could not be identified
```

Step 3:

```
E           passlib.exc.UnknownHashError: hash could not be identified
1 failed, 1 warning
```

**Expected:** `verify_password("password", "not_a_valid_bcrypt_hash")` returns `False`, which is what the test asserts.

**Actual:** it raises `passlib.exc.UnknownHashError: hash could not be identified`, matching the issue. `verify_password` in `core/security.py` (line 37) is a single `return bool(pwd_context.verify(plain_password, hashed_password))` with no exception handling, so the exception reaches the caller.

**Not tested:** I did not try a fix, and I only checked the one malformed value from the test. Other malformed shapes (empty string, `None`) are untested.

**Next step:** I will catch `UnknownHashError` in `verify_password`, return `False`, remove the xfail marker, and post what I find here before opening a PR.

I used Claude Code to help run these steps and draft this comment. I ran every command above on my machine, and the output is copied from those runs.
````

## Eval iterations

### Run history

16/20, 18/20, 16/20, 20/20, 19/20

The last score matches the agreement line in the committed `eval-run.txt` (`agreement: 19/20 scored items (bar: 18/20: PASS)`). One further full run errored partway when my Claude usage limit was hit (13 of 20 packages returned `claude exited 1`), so it produced no score and is not in the list. I also ran several partial `--only` runs, which never print a bar verdict, so they are not counted either.

### Package analysis

**pkg-12** (prettier/prettier#19795, range formatting appends a stray token). The gold label is `accept`. In my final run my rubric decided `reject`, failing `steps-followable`.

The report says it "ran the issue's script verbatim" with a `repro.mjs`, but the report does not paste `repro.mjs`, and the issue itself has no script, only inline input strings and range values in prose. My `steps-followable` row says a file that is described rather than pasted passes only if every trigger-relevant part is named precisely enough to rebuild it. The grader read "verbatim" as pointing at a script that was never shared, so it treated the starting state as unstated. The final run only printed the failed check name. In an earlier run on the same rubric text, the grader's stated evidence was: "'ran the issue's script verbatim' but issue has no script; repro.mjs contents and input strings not shown in report".

The gold reading is that a stranger can rebuild the script from the issue's inline inputs and the report's stated `rangeStart`, `rangeEnd` and `parser`. Re-grading pkg-12 alone on the same unchanged rubric passed it (its `steps-followable` evidence read "inputs are public in the issue"), so this package sits on the edge of my rule and the grader resolves it differently from run to run. I left it as a known miss rather than tune the rule to one package.

### Check rationale

The `behavior-matches` row, exactly as it reads in the uploaded `rubric.md`:

> | behavior-matches | The output excerpt, log, or screenshot in the report, read against the exact failure the issue describes (error text, exit code, crash vs graceful error, symptom). | The artifact shows the issue's own behavior, not an adjacent one. Same error class and same exit status as the issue. A config error (exit 2) is not a panic (exit 101). Garbled output with a live process is not a crash. A compile or usage error caused by an altered input is not the reported bug. The artifact comes from the version and input the issue targets, or the difference is stated. Fail if the artifact shows a different failure, only shows the tool runs, or is absent. Exception: an honest cannot-reproduce report (real attempt, real artifact, states it did not trigger the bug and names what differed) passes this check — it is judged under `honest-outcome` instead, not here. | required |

It reads this way because my first full run scored 16/20 and two of the four misses (pkg-09 and pkg-10) were this check. Both are honest cannot-reproduce reports: a real attempt, real artifacts, and a plain statement that the bug did not trigger. My `honest-outcome` row already says such a report passes, but the original `behavior-matches` row required the artifact to show the issue's own behavior, which a cannot-reproduce artifact never does. The two required checks contradicted each other, and the literal one failed both packages. I added the final "Exception" sentence so a cannot-reproduce report is judged only under `honest-outcome`. I rejected the alternative of making `behavior-matches` `preferred`, because it is the check that catches an adjacent failure (a graceful error shown for a crash), and demoting it would let those through.

### Trade-offs

The exception means `behavior-matches` can no longer fail a cannot-reproduce report on its own. Everything then rests on `honest-outcome` catching a report that says "could not reproduce" while showing the wrong artifact, or that overstates what it tried. I accept that a weak cannot-reproduce report whose artifact happens to look plausible is judged more loosely than a "reproduced" one.

I checked the effect on packages that already agreed with canaries rather than trusting one full run. After the `behavior-matches` and `repo-conventions` changes I ran `--only pkg-03,pkg-05,pkg-09,pkg-10,pkg-01` (pkg-01 as an already-agreeing `clear-accept` canary): pkg-09, pkg-10 and pkg-03 flipped to agree and pkg-01 stayed agreeing. The next full run still disagreed on four packages, but on different checks (`steps-followable` and `claim-specific`), and re-grading some of them on the unchanged rubric flipped them back, so part of that was grader variance rather than a new rubric gap. The `claim-specific` and `steps-followable` edits fixed the two real ambiguities behind it, confirmed with `--only pkg-01,pkg-03,pkg-05,pkg-09,pkg-10,pkg-12`. The full run after that scored 20/20, and the committed run scored 19/20 with only pkg-12 missing.
