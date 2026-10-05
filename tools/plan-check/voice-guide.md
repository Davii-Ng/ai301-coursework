# Voice guide: how I talk upstream

## Who I am in threads

I am a first-time contributor to this repo, learning its workflow through a course. I use AI assistance for my work and I say so. When I comment, I report what I ran and what I saw, and I say plainly when I do not know something. Maintainers can expect short comments with evidence, and follow-up when I say I will follow up.

## Rules I write by

### Rule: State what I ran, not what I believe

Every claim points to something I ran or read, with the version. If I have not run it, I say it is a guess.

- Wrong: "This is definitely a race condition in the debounce logic."
- Right: "I saw the note save fail 3 of 5 runs on v3.1.4. I suspect the debounce, but I have not confirmed it."

### Rule: Name the specifics, never the boilerplate

A comment names the symptom, the version, and my next step. If it could be pasted onto another issue unchanged, I rewrite it.

- Wrong: "Hi, I'd like to work on this issue. Please assign it to me."
- Right: "I reproduced the empty Content-Type on httpie 3.2.4 with `--offline`. I plan to check where the request body is built in `httpie/client.py`."

### Rule: Say what differed from the issue

If my version, OS, or input differs from the issue's, I state it in the first lines of my comment, not at the bottom and not never.

- Wrong: "Reproduced." (on 1.5.3, when the issue is on latest)
- Right: "Tested on 2.2.1, the issue reports 2.3.0. I could not reproduce here. Output below."

### Rule: A cannot-reproduce is a result, so I post it whole

If I cannot reproduce the bug, I post what I tried, what I saw, and what I think a triggering setup needs. I do not hide it and I do not stretch it into a "reproduced".

- Wrong: "Couldn't get it to work, maybe just me."
- Right: "Ran the exact command on Ubuntu 24.04 with 3 files of equal name length. Order was stable. The issue may need a smaller ARG_MAX or mixed name lengths. I will try that next."

### Rule: Disclose AI use where the repo asks, and in my own words

I check the contribution policy before I post. If it asks for disclosure, my comment says AI helped. I write and read every line myself before it goes out.

- Wrong: (comment posted with no mention of AI, in a repo that requires disclosure)
- Right: "AI assistance disclosure: I used Claude Code to draft the steps. I ran every command myself and the output below is real."

### Rule (plan comments): State the approach at the confidence I actually have

A plan comment commits me to an approach in front of the people who maintain the code. I name the cause as confirmed only when my repro shows it; otherwise I call it my working diagnosis and say what would confirm it.

- Wrong: "The root cause is `verify_password` not catching the error. Fix incoming."
- Right: "My repro shows `UnknownHashError` escaping `verify_password` on a malformed hash. Plan: catch it there and return `False`. I have not yet checked whether other callers rely on the exception; I will before opening the PR."

### Rule (plan comments): Answer the maintainer's direction before proposing my own

If a maintainer already pointed at a culprit, suggested an approach, or rejected one, my comment says so first, then says whether I follow it or why I diverge.

- Wrong: (a maintainer suggested option B; my comment proposes option A without mentioning B)
- Right: "Following the suggestion above to use B. I looked at A too, but it touches the shared parser, which is more than this bug needs."

### Rule (plan comments): Say what I will not do

Every plan comment names its limit, so nobody reads a bigger change into it.

- Wrong: "I'll fix this and clean up the auth module."
- Right: "Plan: one change in `core/security.py` plus a regression test. Not touching the hashing scheme or other auth code."

## Things I never post

- "+1", "same here", or "can confirm" with no output from my own machine.
- A promise of a fix or a date ("I'll have a PR by Friday", "easy fix").
- A root cause stated as fact when I have not shown it.
- A reproduction from a version or input other than the issue's, without saying so.
- A comment I have not read end to end, or output I did not run myself.
- A tired shortcut: pasting the same claim text onto a second issue.
- A plan comment that describes less work than my plan contains, or promises a test my plan does not have.
- "Same approach as above" in place of my own plan.
