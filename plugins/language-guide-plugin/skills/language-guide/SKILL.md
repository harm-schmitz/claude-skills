---
name: language-guide
description: >
  Use this skill whenever writing or reviewing documentation prose — wikis,
  READMEs, PR descriptions, ADRs, design docs, or any other written
  explanation. Triggers when the user says things like "write a wiki page",
  "help me write a README", "review this doc for clarity", "make this
  easier to read", "check the wording on this", or "polish this writeup".

  Also triggers when another skill's own instructions say to apply the
  language guide before writing prose (e.g. the design-docs skill, before
  writing overview.md, data_model.md, or ADRs).

  Applies a small set of general heuristics that catch words which force a
  non-native English reader to stop and puzzle over meaning, and replaces
  them with plain, B2-level alternatives.
---

# language-guide

Shared writing-style guide for any documentation prose — design docs,
READMEs, ADRs, PR descriptions, wikis. Not tied to any single project or
skill: apply it whenever you are writing or reviewing prose meant for this
team to read.

## When to use

- The user asks you to write or review any documentation prose directly.
- Another skill's instructions say to apply this guide before writing its
  own output (e.g. `design-docs`, step 3).
- The user explicitly asks for "the language guide".

## Why this matters

The reader is trying to think about the code or the decision being
documented. Every word that forces them to stop and work out "what does
this term mean" pulls attention away from the actual content — all of it
should go to the substance, none of it to parsing prose. Many readers on
this team are non-native English speakers, so words a fluent English writer
would consider ordinary can still cause a stall. Target B2-level English.

This guide is **not** a blocklist of specific words. A blocklist only
catches the exact word it names — it doesn't stop the next word with the
same underlying problem. Each heuristic below is a general, reusable test
you can run on any sentence, anchored by a real example that surfaced it.

## Detection heuristics

1. **False-friend / secondary-meaning words** — flag any word whose most
   common, first-learned meaning belongs to a different domain than the one
   it's used in here. A non-native reader retrieves the dominant meaning
   first; if that's the wrong one for this sentence, they stall trying to
   reconcile it. Test: would a B2 learner's first guess at this word's
   meaning be correct in this exact sentence? If not, replace it.
2. **Unexplained domain-jargon compounds** — flag any project-specific
   technical term (often a two-word or hyphenated compound) that isn't
   standard, widely-taught vocabulary and appears with no definition on
   first use. Test: could a reader who knows English but not this codebase
   look up this exact phrase and get the right meaning? If not, define it
   inline the first time it's used, or replace it with a plain description.
3. **Metaphorical abstract nouns standing in for concrete facts** — flag
   nouns borrowed from another domain (linguistics: "vocabulary"; math:
   "coordinate"; law: "scope") used to describe a plain technical fact. The
   reader has to resolve the metaphor before reaching the actual meaning —
   that's an extra step of "thinking about words" this guide exists to
   remove. Test: can you say what the thing literally *is* or *does*,
   instead of naming an abstract category it belongs to?

## Examples

### 1. "wholesale" — heuristic #1 (false-friend word)
- Source: `docs/decisions/007_sharing_portal_and_flow_repo_code.md`
  (DataOps_Tools-Django)
- Quote: "Portal imports the flow deployable wholesale."
- Why it fails the test: "wholesale"'s dominant meaning is retail/bulk trade
  (buying or selling in bulk) — a different domain from software packaging.
  The intended meaning here, "as a whole, unmodified," is a secondary sense
  most B2 readers won't reach for first.
- Fix applied: "the whole flow deployable package, unchanged."

### 2. "provenance-only" / "row-scope" / "declared per-accessor row" — heuristic #2
- Source: `jobs/privacy/docs/decisions/001_provenance_only_model.md`
  (DataOps_Tools-Django)
- Why it fails the test: these are compounds coined for this ADR's
  privacy-report domain, not standard software vocabulary. None were
  defined on first use. Even "provenance" alone isn't assumed vocabulary
  for this reader.
- Fix applied: replaced with plain descriptions ("tracks where data comes
  from", "a `Scope` field per accessor declaring which rows they may see")
  instead of trying to gloss the coined terms.

### 3. "is the entire vocabulary — a factual coordinate" — heuristic #3
- Source: same file as #2
- Why it fails the test: "vocabulary" and "coordinate" are metaphors
  (linguistics / geometry) standing in for "this is the only kind of fact
  this file records" and "a location, not a meaning." The reader has to
  unpack the metaphor before getting the technical point.
- Fix applied: "all the report knows how to say about a piece of data: a
  plain fact about *where it lives*."

## Contributing

Found a phrase that fails one of the tests above? Add it here under the
matching heuristic, with the source file, the quote, why it fails, and the
fix applied. Only add a new heuristic when a remark reveals a genuinely
different kind of problem — most new remarks should slot into an existing
one.
