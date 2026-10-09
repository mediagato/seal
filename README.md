---
name: seal
type: tool
status: experimental
license: MIT
version: 0.1.0
summary: A Claude skill that finishes one change properly - docs audited, change documented, version bumped everywhere, regression test required, everything committed, pushed only when you ask.
updated: 2026-10-03
verified: 2026-10-03
---

# seal

Finished work rots in three ways. It goes undocumented, so nobody knows what changed. It goes under-versioned, so nobody can tell which build is which. It goes untested, so the bug quietly comes back. And the docs around it drift.

`seal` closes all four at the moment the work is done, while the context is fresh. It treats one change (a bug fix, a feature, a client deliverable) as something that is not finished until the repo is its complete, accurate, versioned record.

## The steps

1. **Audit the docs first.** Read every doc that touches the work and check each claim against what the code and data are now. Do not write new docs until the existing ones are true. If the audit finds something only you can decide (real values or placeholders, content you own), it stops and asks. For a large multi-surface audit it points to [groundtruth](https://github.com/mediagato/groundtruth).
2. **Document the change.** A changelog entry in the file's own voice, README and guide updates where behavior changed, and code comments that explain why, especially for the non-obvious trap that was hit.
3. **Version it.** Bump the version correctly (patch for a fix, minor for an additive feature, major for a breaking change), in lockstep across every surface that carries one: the version file, `package.json`, config, deliverable filenames, `/health`, the app title. One product has one version; surfaces must not disagree.
4. **Regression gate. This is a hard stop.** A bug fix ships a test that fails before the fix and passes after. A feature ships tests for what it added. When the real check needs an environment CI cannot run (a GPU, paid hardware, desktop automation), it ships a static proxy that guards the bug's class. No test means no seal, unless there is an explicit, named reason in the commit or changelog.
5. **Commit everything.** Source, config, docs, tests, and generated outputs on artifact repos, so the repo can reproduce and hand back the exact deliverable. Superseded versioned artifacts go through `git rm`. On pure-code repos where a large binary would bloat clones, commit the source of truth and say where the artifact lives.
6. **Verify, commit, push.** Run build, tests and type checks and confirm they are green, including the new regression test. Commit in coherent, atomic pieces in the repo's own commit voice. Push if you have asked for pushes in this session; otherwise it stops at the commit and says so.
7. **Report.** Terse: what was sealed, the new versions, the regression test added, what was committed and pushed, and anything the audit surfaced that needs a person.

## Illustrative report

The shape of the final message (invented values):

```text
Sealed: csv export no longer drops the last row.
Version: 2.4.1 -> 2.4.2 (VERSION, package.json, /health agree)
Regression test: tests/test_export_last_row.py - fails on 2.4.1, passes now
Docs: CHANGELOG entry, README export section corrected (it claimed a header row that was never written)
Committed: 3 commits, pushed to main
Needs you: docs/guide.md still says exports are Excel-only; I could not tell if that is intended.
```

## Install

Copy `skills/seal` into `~/.claude/skills/` (or a project's `.claude/skills/`). A plugin manifest is included in `.claude-plugin/plugin.json`.

Installed as a plugin (it is named `mediagato-seal` in the directory, since `seal` alone is easily mistaken for other listings) the skill is namespaced, `/mediagato-seal:seal`; copied into `skills/` it is `/seal`. Claude also picks it up from the description, so you do not need to type the command. Ask Claude to "seal it", "lock it in", or "ship it" once a piece of work is done.

## Status

Experimental. It is a procedure for Claude to follow, not a program. It commits to the repo you are in and pushes only if you have asked for pushes in this session. If the repo deploys on push, read the diff first.

## License

MIT. See [LICENSE](LICENSE).

Made at [MEDiAGATO](https://github.com/mediagato).
