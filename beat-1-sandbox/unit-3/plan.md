# Plan: #72, `verify_password` raises `UnknownHashError` on a malformed hash

Repro: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/72#issuecomment-5871767498 (commit `2f4e82f`, Windows 11, Python 3.13.11, passlib 1.7.4, bcrypt 4.3.0)

## Repro evidence

- `verify_password('password', 'not_a_valid_bcrypt_hash')` raises `passlib.exc.UnknownHashError: hash could not be identified` from passlib's `identify_record`.
- The covering test with `--runxfail` gives `1 failed`. As shipped, `tests/unit/test_security.py` gives `24 passed, 1 xfailed`.

## Diagnosis

`verify_password` (`core/security.py:37`) is a bare `return bool(pwd_context.verify(...))`, with no exception handling. passlib raises `UnknownHashError` when it can't identify the hash, and that error reaches the caller. The valid-hash tests pass, so verification itself works.

## Scope

- **In:** catch `UnknownHashError` in `verify_password` and return `False`. Remove the strict `xfail` marker, as the issue and CONTRIBUTING require.
- **Out:** `hash_password`, the `CryptContext` config, `api/routes/auth.py` (it already treats `False` as a failed login), broader catches (`ValueError`, `Exception`), and logging.

## Files and approach

1. `core/security.py`: import `UnknownHashError` from `passlib.exc` and wrap the call:
   ```python
   try:
       return bool(pwd_context.verify(plain_password, hashed_password))
   except UnknownHashError:
       return False
   ```
2. `tests/unit/test_security.py`: delete the `@pytest.mark.xfail` decorator on `test_verify_with_wrong_hash_format`.

## Test plan

Re-run my repro steps on branch `fix/72-verify-password-unknown-hash`:

1. Direct call: expect it to print `False`, with no traceback.
2. The covering test without the marker: expect `1 passed`.
3. `tests/unit/test_security.py`: expect `25 passed` (was `24 passed, 1 xfailed`).
4. `make test-unit`: expect no new failures compared with `main`.

## Risks and unknowns

- I only tested one malformed value. I'll try `''`, `None`, and a truncated `$2b$` hash, and post the results in the thread. If any of them raises something other than `UnknownHashError`, I won't widen the catch on my own. I'll ask in the thread instead.
- Returning `False` turns bad stored data into a failed login, with no log. This is what the issue asks for.

## Deviations

No deviation in the change itself. Results:

- Direct call: `UnknownHashError` → `False`.
- Covering test: `1 failed` → `1 passed`.
- `tests/unit/test_security.py`: `24 passed, 1 xfailed` → `25 passed`.
- `pytest tests/unit -m unit`: `375 passed, 53 xfailed` → `376 passed, 52 xfailed`.
- `ruff`, `black`, and `mypy` are clean on the changed files.

The other inputs:

- `''` now returns `False` (it raised before).
- `None` returned `False` before too.
- `'$2b$12$abc'` still raises `ValueError: salt too small`. I didn't widen the catch. This is an open question in the thread (see PR #78).

I built the branch before posting the plan comment. The build matches the plan.
