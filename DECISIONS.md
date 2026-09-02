# Decisions

## Reference sqlite-rs's grammar file, don't mirror it (2026-09-02)

Issue #1 asked whether this repo should also carry a `sqlite-rs.ebnf`,
promoted/copied from sqlite-rs's own `.openspec/grammar/sqlite.ebnf`.

**Decision: reference it, don't duplicate it.**

Reasoning:

- sqlite-rs's grammar file already has a canonical home and its own
  maintenance rule (mirrors sqlite-rs spec 005: any parser-growing ticket
  there must grow that file in the same PR). A copy here would be a second
  copy of the same obligation, enforced by a different repo's process —
  guaranteed to drift the first time someone updates one without the
  other.
- sqlite-rs's file also carries V-block annotations and `parse.y` line
  refs tied to its own value-block plan (`plan.md`) and a pinned SQLite
  version (3.53.4). That provenance metadata doesn't transplant cleanly
  into a "cross-engine" file that isn't tracking sqlite-rs's plan.
- This repo's actual value-add (per `README.md`) is being the place
  where t-rust-db's *own* grammars live in a notation consistent with
  sqlite-rs's conventions — not a warehouse for other repos' grammar
  files. `ALIGNMENT.md` already exists for side-by-side comparison notes;
  a link is enough for that purpose.

Acted on this by adding a pointer in `README.md` to sqlite-rs's grammar
file instead of copying it.
