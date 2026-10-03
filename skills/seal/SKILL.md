---
name: seal
description: >-
  Finalizes a deliverable or change so the repo is its complete, accurate, versioned record — audit
  existing docs for accuracy FIRST, then document thoroughly, semver-bump (lockstep across surfaces),
  commit EVERYTHING (incl. generated artifacts; over-include rather than under), and gate on a
  regression test that would catch the bug/feature. Use when a piece of work is done and ready to
  record/ship: the user says "seal", "seal it", "lock it in", "ship it", "make it official", or after
  fixing a bug / finishing a feature / cutting a deliverable. Finalizes ONE artifact, not a whole
  session.
---

# /seal — sign, seal, deliver one piece of work

Make the repo the **complete, accurate, versioned record** of a finished change
— a bug fix, a feature, a client deliverable. The bias is *over-include*:
better too much in the repo than too little. The discipline that makes it safe
is **audit the docs for accuracy BEFORE writing new ones** — stale docs are
worse than missing ones, and the audit catches drift before it ships.

**Why this exists:** finished work rots in three ways — undocumented (nobody
knows what changed), under-versioned (can't tell which build is which), and
un-tested (the bug silently comes back). And docs drift: a config comment saying "sanitized placeholders" while the real values sit right there, a
CHANGELOG that lies about what's deployed. /seal closes all of those at the
moment the work is done, while the context is fresh. A bug that ships with no test and no doc trail comes back; the audit-first step catches the drift and the regression gate catches the bug.

## Input

- Optional arg: what's being sealed (e.g. `/seal export-csv`, `/seal the retry fix`).
  If omitted, infer the artifact from the session's most recent finished work.

## Procedure

### 1. Audit the docs FIRST (before touching them)

Read every doc that *touches this work* and verify each claim against current
reality — **do not write new docs until the existing ones are true.** Surfaces
to check (whichever apply): `CHANGELOG.md`, `README`, in-repo guides, `*.yaml`
config + its comments, in-app help pages, public docs site, version stamps.

For each: does it match what the code/data/deck actually is *now*? Flag and fix
(or surface to the user) any drift — stale comments, lying changelogs,
placeholder-vs-real ambiguity, a "parked task" that a blind update would trample.
**If the audit finds something the user must decide (real vs placeholder data,
content the user owns), STOP and ask — don't paper over it.** This step is the
seal's whole value; it's caught real ship-blockers.

For a large or multi-surface audit (many docs, a whole repo, a public docs site),
run **/groundtruth** on the affected surface — it builds a claim-by-claim ledger
against primary reality and propagates fixes across sibling surfaces — then return
here to finish the seal (version bump + regression gate + commit).

### 2. Document the change thoroughly

Now write/update the docs. Bias to over-document:
- CHANGELOG entry (newest-first), in the file's existing voice.
- README / guide / in-app doc updates if behavior or usage changed.
- Code comments explaining *why* (esp. the non-obvious — the trap that was hit).
- For products with a public docs surface, update it in the same pass (the
  "every user-facing change touches the docs" rule).

### 3. Semver

Bump the version correctly and in **lockstep** across every surface that carries
one (`VERSION` file, `package.json`, config `version:`, deliverable filename, `/health`, app title, etc.):
- **patch** — bug fix / no behavior or content change (e.g. a compile fix).
- **minor** — additive feature, backward-compatible.
- **major** — breaking change.
A single product version, structurally; surfaces must not disagree.

### 4. Regression gate (hard stop)

The change does not seal until a test locks it in:
- A bug fix ships a test that **fails before the fix, passes after** — proving it
  catches the regression.
- A feature ships behavioral tests for what it added.
- **When the real check needs an environment CI can't run** (PowerPoint COM, a
  GPU, paid hardware), do NOT skip it — ship a **static proxy** that guards the
  bug *class* in plain CI (e.g. a reserved-word scan that guards a macro naming trap without compiling the macro). A proxy guard beats no guard.
- The test must run where it'll actually catch future regressions (the CI
  pipeline), not just once locally.

If no test is added, that's a STOP — either add one or log an explicit,
named reason in the commit/CHANGELOG for why it's untestable even by proxy.

### 5. Upload everything (over-include)

Commit ALL the artifacts of the change, erring toward too much:
- Source + config + docs + tests.
- **Generated outputs too** on artifact repos (rendered decks, PNGs, exports) —
  the repo should be able to reproduce *and* hand back the exact deliverable.
- Remove superseded versioned artifacts via `git rm` (rename old→new) so history
  stays clean, but keep the record complete.
- Use judgment on pure-code repos where a giant binary would bloat clones —
  there, commit the source-of-truth and document where the artifact lives. On client/deliverable repos, commit the binary.

### 6. Verify + commit + push

- Run the build / tests / type-check; confirm green (incl. the new regression test).
- Commit in coherent, atomic pieces (engine vs deliverable), the repo's own commit voice (read its log).
- Push if the user has asked for pushes in this session; otherwise stop at the commit and say so.

### 7. Confirm

Reply terse: what was sealed, the new version(s), the regression test added, what
was committed/pushed (hashes), and anything the audit surfaced that needs the
user. Don't re-narrate the whole change.

## Notes

- Pairs with the standing **fix-with-a-test / feature-with-a-test** rule (the
  regression gate is where /seal enforces it; the rule is that it's a habit
  during the work, not only at seal time).
- /seal is per-artifact. For a whole-session wrap-up, write a session summary instead.
