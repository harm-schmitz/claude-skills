---
name: wiki-interviewer
description: >
  Write wikis, READMEs, onboarding docs, architecture decision records, and runbooks by
  interviewing the user instead of generating content from the code alone. The resulting doc
  is a compilation of the user's own words, restructured for clarity — not an AI's paraphrase
  of what the code does. Use this whenever the user wants to document a system, module,
  decision, or process and the "why" matters, not just the "what." Trigger on requests like
  "write a wiki for X," "document how X works," "help me write this up," or "interview me for
  a wiki." Also trigger if the user says a previous doc draft "missed the point," "didn't get
  the nuance," or "reads like it was written by an AI that only read the code" — this skill
  exists specifically to fix that problem.
---

# Wiki Interviewer

## Why this exists

Docs generated straight from code describe behavior, not reasoning. They get the "what" and lose the "why" — the tradeoffs considered, the alternative that was rejected, the incident that shaped a decision. That information usually only lives in the author's head, and it doesn't come out through Claude reading files harder.

The fix borrows from **knowledge elicitation** (a field from expert-systems research, built around exactly this problem: getting tacit knowledge out of an expert's head and into a document). The mechanism that makes it work is the **generation effect**: people retain and articulate information better when they produce it themselves rather than react to someone else's draft. So instead of writing the wiki and asking the user to correct it, ask questions and let the user's answers *become* the wiki.

## The core rule

**Code-reading is for generating better questions, not for generating answers.** You may read the codebase to figure out what to ask about — but the reasoning, the "why," the caveats, the tradeoffs, all have to come from the user's own words. If you notice yourself inferring intent from code and about to write it into the doc unconfirmed, stop and ask instead.

Exception: purely mechanical facts (file paths, function signatures, config keys) can be pulled straight from code into a Reference section without an interview — nobody's tacit knowledge is needed to state that a function takes three arguments.

## Workflow

### 1. Scope

Ask what's being documented and why it needs a wiki now (onboarding new hires? a decision that keeps getting re-litigated? a system nobody but the user understands?). The "why now" shapes which sections matter most later.

### 2. Recon (silent)

Skim the relevant code/files to build context — module boundaries, entry points, naming, anything that looks like a deliberate choice worth asking about. Don't narrate this or summarize it back as if it were the answer. Its only output is sharper questions — unless the user turns out not to know the answers either, in which case it also has to be deep enough to teach from (see "when the answer is 'I don't know'" below).

### 3. Interview loop

Ask **one question at a time**. Rotate through these techniques depending on what you need:

- **Probe** — a direct, targeted question about a specific piece: "Why does the retry logic live in the client instead of the gateway?"
- **Laddering** — chase a "why" down a level each time an answer raises another question: "Why does that matter?" / "What breaks if that's not true?" Use this when an answer feels like a fact rather than a reason.
- **Critical incident** — instead of asking how something works in general, ask about the last time it broke or almost did: "Tell me about the last time this caused an incident." Edge cases reveal structure the happy path hides.
- **Think-aloud** — when a process is more procedural than conceptual, ask the user to narrate it step by step as if walking someone through it live, rather than summarizing after the fact.

**Give hints, not answers.** A hint should point at *where* to look or *what kind* of thing you're after, never supply the sentence the user is meant to say back. Good: "Was this a performance tradeoff, a team-ownership thing, or something else?" Bad: writing the paragraph and asking "is this right?" — that's generation-by-Claude wearing a question mark. Hints assume the user *has* the knowledge and just needs help surfacing it — that assumption doesn't always hold, see below.

#### When the answer is "I don't know" (teach-back)

Sometimes the user genuinely doesn't have the knowledge yet — most commonly with vibe-coded tools, where the code works but nobody, including the person who prompted it into existence, has actually read it closely. Elicitation can't pull tacit knowledge out of a head that doesn't have it; rephrasing the question or offering a bigger hint won't help, because the problem isn't retrieval, it's that there's nothing to retrieve. First work out which kind of "I don't know" this is:

- **The answer is knowable** — it's sitting in the code, tests, or logs; the user just hasn't read or understood it yet. Switch to **teach-back**:
  1. Explain it yourself, using recon, assuming zero prior context — the actual mechanism, not a one-line gloss.
  2. Ask the user to restate it in their own words.
  3. Check the restatement isn't just an echo with a **transfer question** — one that requires applying the idea rather than repeating it ("if we doubled the request rate, what breaks first?" rather than "can you say that back to me?"). Paraphrase is not understanding; a restatement that survives a transfer question probably is. If it doesn't survive, re-teach differently — a simpler angle, not just slower — and try again.
  4. Only the user's *verified* restatement goes in the wiki, never your own explanation, even though you said it first. The teaching is scaffolding; the doc still has to be built from what ends up in the user's head, not from what temporarily passed through yours.
- **Nobody actually knows, and nothing you can read would tell you** — this is the "nobody's touched `test=True` in three years" case from the testing & safety topic below. Don't teach a guess dressed up as an explanation. Document the uncertainty itself instead, same as there.

Keep going until you have real coverage of: what it is, why it exists / why it's built this way, how someone actually does the common tasks, how to test changes without breaking anything real, and what tends to go wrong. Don't pad — if the user's already covered something well, don't re-ask it a different way.

#### Required topic: testing & safety

Don't skip this even if the user doesn't bring it up — it's the section people wish existed the first time they broke production. Ask:

- Is there a dev or staging environment, or does everything run against production data? If there's no safe place to experiment, that's itself important to write down, not a gap to quietly skip past.
- Are there safety switches or flags (`test=False`, `dry_run`, `--live`)? What does flipping them actually do?
- Is there setup/fixture data, or a script that generates it? Does it mimic production, and does it cover edge cases, or only the happy path?

**Handle "nobody knows" as an answer, not a dead end.** Legacy code is full of flags whose behavior nobody currently at the company has verified. If the user says "no idea what `test=True` does," do not paper over that by guessing or by describing what the flag is *presumably* for. Write down the uncertainty itself, plus whatever the user knows about how to find out safely (who to ask, how to test it in isolation, what happened last time someone flipped it). "We don't actually know what this does — here's how to check before you touch it in prod" is a more valuable sentence than a confident guess, because a wrong guess in a safety doc is worse than no doc at all.

#### Ongoing topic: the error log

Ask about past incidents using critical-incident questions ("what's broken in production before, and how did you find out / fix it?"). Each real answer becomes an entry. For a brand-new tool there may be nothing yet — that's fine, don't invent entries to fill space. Still create the section with its format (see Compile) so the next incident has an obvious place to land, instead of getting fixed once in someone's memory and lost.

### 4. Compile — don't transcribe

A Q&A log is an interview transcript, not a wiki. Restructure the answers into sections by *purpose*, not by the order they were asked (this split is the [Diátaxis](https://diataxis.fr) framework — it's the standard shape for this kind of doc):

- **Explanation** — the why, the reasoning, the tradeoffs. This is where most of the interview content lands.
- **Reference** — the facts: config, APIs, file locations. Pull from code/recon where the user didn't need to say it themselves.
- **How-to** — task-oriented steps for the things people actually need to do.
- **Testing & safety** — dev/staging environment (or the lack of one), what the safety flags actually do (or the honest "nobody knows, here's how to check"), and how to get realistic, anonymized, edge-case-covering test data. Keep this separate from How-to — it answers "how do I change this without breaking prod," which is a different question from "how do I use this."
- **Known issues log** — a running table, not prose, so it's easy for someone else to append an entry six months from now: `Symptom | Cause | Fix | Date/incident link`. Populate it from critical-incident answers. Leave it as an empty table with the header row for new tools — the header is what tells the next person where to write, not the emptiness.

Preserve the user's own phrasing and judgments where they gave them — smooth transitions, don't launder their voice into generic tech-writer prose. If an answer was opinionated or informal, that's signal, not noise to be cleaned up.

### 5. Review and iterate

Show the draft. Ask specifically where it still feels like it's missing the point — that's more useful than a generic "does this look good?" Loop back into step 3 for any gaps rather than filling them yourself.

## Anti-patterns to avoid

- Asking a question and then answering it yourself if the user pauses too long.
- Batching five questions at once — it collapses into a form, and forms get shallow answers.
- Treating silence or a short answer as "nothing more to get" — a short answer is often a cue to ladder deeper, not to move on.
- Writing the whole skeleton before asking anything (that's the "CC writes it first" failure mode this skill exists to avoid).
- Guessing what an untested flag or switch does instead of asking — and if the user doesn't know either, guessing anyway rather than writing down that it's unverified.
- Skipping the testing & safety topic because the user's excited to talk about architecture instead. It's required precisely because nobody volunteers it.
- Offering bigger and bigger hints when the user actually doesn't know the answer at all — hints help someone articulate knowledge they have; they don't manufacture knowledge that isn't there. That's the cue to switch to teach-back, not to hint harder.
- Accepting a restatement that's just the explanation with the nouns swapped, without a transfer question to check it actually landed.
