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

## Revisited after `db-core/sql-parser`'s row/column split — decision holds (2026-09-03)

`db-core#24` asked to revisit the above now that `sql-parser` unifies
into one crate with `column`/`row` Cargo-feature sections (`db-core` ADR
0002), with `row` migrating in sqlite-rs's own parser
(`src/parser/tokenizer.rs`/`ast.rs`/`grammar.rs`/`error.rs`/`printer.rs`)
essentially unchanged (`db-core#23`).

**Decision: still reference, still don't mirror — no `sqlite-rs.ebnf`
added here.** If anything, the original reasoning is stronger now:

- The original concern was two grammar files drifting because they're
  maintained by different processes. `sql-parser::row` isn't a second
  *independent* grammar someone could accidentally grow out of sync with
  sqlite-rs's `.openspec/grammar/sqlite.ebnf` — it's a migrated *copy* of
  the code sqlite-rs's own grammar file already documents. There is
  nothing new here for a `.ebnf` file to describe that sqlite-rs's own
  doesn't already cover.
- `sql-parser::row`'s own AST is sqlite-rs's AST (ADR 0002's amendment:
  folding it into `sql_expr::Query` would have been a redesign, not a
  port) — so even the "genuinely different, much smaller language" case
  `column-rs.ebnf` exists for doesn't apply to `row`. `row` isn't a new
  language variant; it's sqlite-rs's language, running in a different
  crate.
- Should `sql-parser::row` ever diverge from a straight port (its own
  bug fixes, its own grammar extensions not yet upstreamed to
  sqlite-rs), that's the trigger to revisit this again — not before.

`column-rs.ebnf` itself needed one small fix, unrelated to this
question: its header comment cited `db-core/sql-parser/src/lib.rs`,
which no longer exists as a single file post-split — updated to
`db-core/sql-parser/src/column.rs`, plus a note pointing at the new
`row` section. `README.md` gained a section describing both
`sql-parser` sections and why only `column` gets a `.ebnf` file here.
