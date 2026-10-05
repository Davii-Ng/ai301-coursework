# Procedure: how this skill grades a plan package

Follow these steps in order. Each step names what to read and what to
write down. Write the notes in your working (not in the final JSON);
the checks are graded from these notes, so two executors with the same
package build the same notes and reach the same grades.

The rubric (`rubric.md`) says what each check decides. The evidence
guide (`references/evidence-guide.md`) says where each signal lives
and shows worked pass/fail examples. When a case looks like one of
those examples, grade it the way the example is graded.

## Read order

Read the evidence before the plan. A confident plan read first anchors
the reader on its story, and then the controls get read as support for
it instead of as a test of it. So the plan and the comment come last.

1. **Mode.** Eval mode (a bundle file): the bundle is the only source;
   skip `scope.md` and `voice-guide.md`. Live mode: read `scope.md`
   first and apply it as `SKILL.md` says, then gather the issue,
   thread, and the student's posted repro comment per the evidence
   guide's "Live" columns.
2. **Issue.** Note in one line the behavior the issue reports
   (trigger, symptom, expected). This is the one behavior the plan is
   allowed to fix.
3. **Thread highlights.** For each entry, note the author's role. List
   every maintainer-side statement (OWNER, MEMBER, COLLABORATOR, or a
   CONTRIBUTOR who opened the issue or is steering the design) that
   names a culprit or fix site, settles or rejects an approach, posts
   a patch or test build, or rules "works as intended". List causes
   claimed by anyone, and any open PRs.
4. **Repo facts.** Copy the `contribution policy` line. Classify the
   AI policy now as one of: requires disclosure of all AI use /
   conditions only / silent or permissive (evidence guide, "Comms").
5. **Repro evidence.** Build the control ledger (next section). Do
   this before reading the plan.
6. **Candidate plan.** Read it whole once, then gather from it
   (next section).
7. **Candidate plan comment.** Read it last, sentence by sentence.

## Evidence gathering

Fill these nine notes. Quote short phrases from the package where you
can; "absent" is a valid note.

- **N1 Behavior.** From step 2: the issue's trigger, symptom,
  expected result.
- **N2 Control ledger.** From `Repro evidence`: one line for the
  failing run (what was run, what was seen), then one line per control,
  version comparison, feature toggle, native/upstream run, or
  debug/trace step: what changed versus the failing run, what
  happened, and what that rules out ("component X works without the
  flag", "damage already present before step Y runs", "cost present
  with Z out of the loop").
- **N3 Stated cause.** From the plan's diagnosis/cause text: quote the
  cause. If the plan only restates the symptom or names no cause,
  write "no cause". Note whether the cause is taken from the thread.
- **N4 Change items.** Every change the plan commits to (numbered
  steps, "Proposed changes", "Change:", "while in there" clauses).
  Tag each: fix / sibling / test / extra, using the evidence guide's
  "Scope" list.
- **N5 Limits.** Quote the plan's not-in-scope / out / deferred /
  file-separately statement, or "absent".
- **N6 Location and approach.** Files, functions, or areas named;
  the concrete change chosen at each; every open decision ("or",
  "maybe", "somewhere", "not sure", "whichever", investigation-only
  steps).
- **N7 Tests.** Each test in the plan: does it re-run the repro
  trigger (or encode that exact case)? What expected result after the
  fix does it state, word for word?
- **N8 Thread and policy.** From step 3 and step 4: the maintainer-side
  directions, and for each, the plan or comment phrase that engages it
  (or "not engaged"). The AI-policy case, and whether the comment
  contains an AI disclosure.
- **N9 Comment claims.** Each sentence of the comment that promises a
  change, test, result, scope, timing, or certainty, paired with the
  plan or evidence line that holds it (or "not held").

## Check execution

Run all six checks, in this order, every time. Do not stop at the
first fail: the output must name every reason to hold.

1. **diagnosis-grounded** from N2 + N3 (+ N4 for "acts on the
   cause"). For each N2 control, write what the N3 cause predicts the
   control would show, then compare with what it did show. Any
   mismatch: fail, and quote the control. "No cause": fail. Then check
   the change acts on that cause; a workaround, docs-only, or
   catch-all-suppression change for a code-level defect fails.
   Otherwise pass. Do not pass a cause because the thread or the plan
   sounds sure of it.
2. **bounded-scope** from N4 + N5. Any item tagged `extra`: fail, and
   name it. N5 "absent": fail. Otherwise pass.
3. **stranger-executable** from N6. A location and a chosen change at
   it: pass. Investigation-only steps, no location, or an open
   decision between unlike options: fail, and quote it. A named,
   bounded unknown inside a chosen approach does not fail.
4. **test-decisive** from N7 against N2's failing run. At least one
   test re-runs that trigger and states the expected result after the
   fix: pass. Otherwise fail, and quote the test plan.
5. **comment-faithful** from N9. Any claim "not held", any upgraded
   certainty, scope described differently from N4, or a fixed
   deadline: fail, and quote it. Otherwise pass.
6. **thread-and-policy** from N8. Any maintainer-side direction "not
   engaged": fail. AI-policy case "requires disclosure" with no
   disclosure in the comment: fail. Otherwise pass.

When evidence is absent: in eval mode the bundle is the whole world,
so a missing cause, limit, location, or observable is a fail on the
check that needs it, not `unclear`. Use `unclear` only when the
package holds the evidence but it supports two readings you cannot
settle with the rubric and the guide's examples; say what the two
readings are.

A check may be graded from the notes alone, without re-reading the
package, once its notes are filled. Re-read the package only to
settle an `unclear` or to fetch an exact quote.

Grade the substance, never the shape: no check passes or fails on
length, headings, tone, or polish.

## Verdict assembly

1. Apply the rubric's verdict rule: `accept` only if all six checks
   are `pass`. Any `fail` or `unclear` makes it `reject`.
2. For each check, the `evidence` field is one line: the fact or
   quote that decided it. For a failed check, quote the deciding
   phrase from the package (the contradicting control, the extra
   item, the open decision, the vague test, the unheld claim, or the
   ignored direction).
3. Before the JSON, write a short summary: one line per check, and
   for a reject, which check(s) hold the package. In live mode, add
   voice-guide notes as `SKILL.md` says; they never change the
   verdict.
4. Emit the JSON block from `SKILL.md`, last, with `item` set to the
   bundle id (eval) or issue URL (live), and the six checks in the
   order above.
