# Evidence guide: where proof lives in a reproduction package

Every rubric check needs an evidence source. This guide maps the five
proof families to concrete places you can look: in the eval bundle
(a markdown file with sections `Repo facts`, `Issue`, `Thread
highlights`, `Candidate claim comment`, `Candidate repro report`) and, in
live mode, on GitHub or in the student's draft. In eval mode the bundle
is the whole world: quote it, fetch nothing. Every date and version is
measured against the bundle's capture date; live mode measures against
today.

Read the issue first (what it says the bug is), then the report (what
was shown). Every family below compares the report to the issue.

## Family 1: is the environment recorded?

A run nobody can place is a run nobody can trust.

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Environment record | the `Environment:` line at the top of the `Candidate repro report` (OS, tool version or commit, runtime, plus factors like shell, driver, arch, build profile) | the same line in the draft comment |
| Issue's target | the `Issue` body's environment line and version, the thread's later corrections, and `latest release` under `Repo facts` | the issue body, the thread, the Releases box in the repo sidebar |
| Factor that matters | any environment word the issue ties to the bug ("only on Windows", "with fish", "release build") | same, from the issue body and maintainer comments |

What good looks like:

- OS, exact version or commit, and runtime are all named. "Latest" or
  "recent" is not a version.
- Every factor the issue ties to the bug is named. If the issue says the
  build profile changes the failure, the profile is in the record.
- If the version or platform differs from the issue's, the report says
  so in words. A difference the report never mentions is a silent
  deviation: fail. A stated difference passes; a newer version is often
  fine.
- No environment line at all fails, however good the log looks.

## Family 2: are the steps followable?

Steps are for a stranger who has never seen this repo.

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Commands and actions | the command block and any numbered steps in the `Candidate repro report` | the draft's steps; the repo's README or CONTRIBUTING for install and run |
| Starting state | the first lines of the report: clone, install, input file, config, or an inline heredoc | same |
| Trigger | the step the `Issue` body names as the trigger, matched against the report's steps | the issue's own steps |
| Inputs | files or configs the steps use, and whether they are shown inline or linked publicly | links in the draft; check that they open without access |

What good looks like:

- Exact commands, in order, from the starting state to the trigger. A
  stranger can paste the block and reach the trigger.
- The issue's trigger is kept intact. A report that runs a similar
  command, a shorter input, or a different flag skipped the trigger:
  fail.
- Inputs are inline or public. A private monorepo, an unshared config,
  or "my usual setup" fails.
- Short is not unfollowable. A terse four-line block with all the inputs
  passes. Do not grade length, headings, or numbering.
- A described (not pasted) file passes only if every trigger-relevant
  part of its contents is named precisely enough to rebuild: the exact
  keys, sections, flags, or values the trigger depends on. A description
  that omits or blurs the triggering content fails the same as an
  unshared file.

**Worked example — described vs pasted input, same bug.** Issue: a
parser crashes only when a config has two conflicting keys, `mode: fast`
and `mode: safe`, in the same file. Report A: pastes the literal file.
Report B: writes "a config with `mode: fast` and `mode: safe` both set"
without pasting it — passes; both conflicting keys, the only thing the
trigger depends on, are named exactly. Report C: writes "a config with a
few conflicting settings" — fails; the specific keys that trigger the
crash are never named, so a stranger cannot rebuild the file that
triggers it.

## Family 3: does the behavior shown match the issue?

The deciding family. Read the artifact, not the narration around it.

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Artifact | the output excerpt, log, `$ echo $?` line, or screenshot text in the `Candidate repro report` | the draft's pasted output; run the steps yourself only if the draft is unclear |
| Reported failure | the error text, exit code, and symptom quoted in the `Issue` body and `Thread highlights` | the issue body and thread |
| Control run | a second run in the report that should succeed, or differ | same |

What good looks like:

- The artifact shows the issue's own error class and exit status, from
  the input and version the issue targets. Same message, same kind of
  failure.
- Look for the adjacent behavior and fail on it:
  - a graceful error shown for a crash (exit 2 config error for an
    exit 101 panic, exit 1 arg-validation for a capacity overflow);
  - a syntax or compile error caused by an input altered from the
    issue's (a colon for `=`, an unbound variable);
  - garbled output with the process still alive, narrated as a crash;
  - an old version's error shown for an issue confirmed on latest;
  - an artifact that only shows the tool starts (version banner, session
    list) and nothing of the symptom.
- A control run that differs only in the trigger strengthens the proof.
  It does not rescue a wrong artifact.
- No artifact at all, only narration ("I reproduced it"), fails.

**Worked example — honest cannot-reproduce is not a `behavior-matches`
fail.** Issue: a race condition that drops one of two log lines under
load. Report: ran the exact steps 20 times, both log lines appeared
every time, states plainly "could not trigger the drop," and names what
may differ (issue's host had 1 CPU core, report's had 8). The artifact
shows the *opposite* of the bug — pass `behavior-matches` anyway; this
report is graded on whether it honestly reports that mismatch
(`honest-outcome`), not on whether it reproduced the symptom. Contrast: a report with the same successful run but that *claims*
"reproduced" anyway — the artifact still passes `behavior-matches` (it
is a real attempt with a real artifact), but the false claim fails
`honest-outcome`. The mismatch between claim and artifact is always
graded there, never here.

## Family 4: is the outcome stated honestly?

The claim must stop where the evidence stops.

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Stated result | the `Expected`, `Actual`, and conclusion lines in the `Candidate repro report`, plus the claim comment's first sentence | the draft's conclusion |
| Backing | the artifact each claim points to | same |
| Cause or scope claims | any "root cause", "verified", "on all platforms", "guaranteed" wording | same |

What good looks like:

- Each stated result is what the artifact shows. `Expected` and `Actual`
  are observations, not swapped or restated from the issue.
- An evidenced cannot-reproduce passes: it shows a real attempt with
  artifacts, names what differed from the issue (input shape, version,
  OS, shell), and says what a triggering setup may need. This is a pass,
  not a soft fail.
- A confident "reproduced" fails unless the artifact shows the bug.
- A root cause with no shown evidence fails. The same sentence framed as
  a hypothesis ("likely X; not confirmed") passes.
- A claim that generalizes past what was tested (another OS, another
  release, "always") fails.

## Family 5: do the words respect the repo?

Comms lives in the claim comment, and in both comments against the
repo's rules.

| Signal | In the eval bundle | Live (GitHub / draft) |
|---|---|---|
| Claim specificity | the `Candidate claim comment`, and the `Candidate repro report` when both are being graded together: symptom named, version tested, next step. The version may be stated in either the comment or the paired report — they are read as one submission when both are present | the draft claim, and the draft repro comment when both are being graded together |
| Bug-report template | the `bug reports:` line under `Repo facts` (what the template asks for) | `.github/ISSUE_TEMPLATE/` and the new-issue page |
| Contribution policy | the `contribution policy` line under `Repo facts` | `CONTRIBUTING.md` in the root or `.github/`, files it links to |
| AI policy | the same line; dedicated files such as `AI_POLICY.md`; an `AGENTS.md` file is instructions for AI agents, not a ban | the same files |

What good looks like:

- The claim names the symptom, the version tested, and one concrete next
  step (a file, function, or fix direction to look at). Test: could it
  be pasted onto a different issue unchanged? If yes, it is boilerplate:
  fail.
- The version does not have to repeat in the comment if the paired
  report already states it: a claim comment that says "reproduced on
  current release, will look at X next" passes this part of the check
  when the report's `Environment:` line names the exact version, since
  the comment and report are graded as one submission. A claim-only
  draft with no report still needs the version in the comment itself.
- No promise the evidence cannot back: "guaranteed fix", "done in 2
  days", "can confirm" with no artifact.
- The report covers every template item this bug's evidence depends on.
  A routine field the template asks of every report, that this report's
  proof does not lean on, is not a reason to fail on its own — see the
  worked example below for the line between "depends on" and "routine
  ask."
- AI policy, three cases:
  - **Requires disclosure**: the comment or report must say AI
    assistance was used. Treat course packages as AI-assisted work. No
    disclosure fails, even when every proof check passes.
  - **Conditions** (personal understanding, testing, human-voiced
    comments): the comments must meet them. Follow the terms; they are
    not a reason to walk away. No separate disclosure line is needed
    once the comment itself reads as meeting the condition.
  - **Silent or permissive**: no disclosure is needed. Silence passes.

**Worked example — "depends on" vs "routine ask."** A template asks
every reporter for `env dump` (full dependency tree) alongside OS and
version. Bug A: a version-conflict crash where the dependency tree is
the evidence that two packages pin incompatible versions — a report
missing the dump fails, because the proof leans on exactly that field.
Bug B: a CLI flag typo that panics regardless of what's installed — a
report giving OS, version, and the exact command, but not the
dependency dump, still passes; the dump is a routine ask this bug's
proof never touches. Same missing field, different verdict, because the
test is whether *this report's* evidence needs it, not whether the
template lists it.

**Worked example — the three AI-policy cases side by side.** Same
comment in all three: "First contribution attempt — reproduced on
current version, will read the flagged function next." Repo 1's policy
says every comment must disclose AI assistance: this comment never says
so, fails, however good the repro is. Repo 2's policy says comments must
be in the contributor's own words (no disclosure line asked): this
comment reads human-voiced and names concrete specifics, passes, with
no separate disclosure sentence needed. Repo 3 has no AI policy, or a
permissive one: passes as written, silence included.
