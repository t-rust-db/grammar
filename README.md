# grammar

SQL grammar reference for t-rust-db products, in standard EBNF.

- **`column-rs.ebnf`** — column-rs's actual supported SQL subset, derived
  directly from `db-core/sql-parser`'s `column` section's parser functions
  (not aspirational — every rule here has a corresponding parser function
  that accepts it).

## `db-core/sql-parser`'s two sections

As of `db-core` ADR 0002, `sql-parser` is one crate with two
Cargo-feature-gated sections, mirroring `sql-vm`'s `batch`/`row`/`stream`
split:

- **`column`** — column-rs's analytics subset. What `column-rs.ebnf`
  describes.
- **`row`** — sqlite-rs's own grammar (DDL, DML, transactions, `PRAGMA`,
  ...), migrated in from sqlite-rs's `src/parser/*` (tokenizer, AST,
  recursive-descent parser, three-way parse outcome, AST printer) as a
  mechanical port — not an independent reimplementation. Its own AST is
  **not** `sql_expr::Query` (see ADR 0002's amendment); it's sqlite-rs's
  AST, unchanged.

This repo still has no `sqlite-rs.ebnf` — see below, that call still
holds, and holds *more* strongly now that `row` is a copy of sqlite-rs's
own parser rather than an independent implementation that could drift.

## Why this repo exists

sqlite-rs (a separate, pre-existing project — see
`.openspec/grammar/sqlite.ebnf` there) already tracks the full SQLite
grammar this way. This repo mirrors that notation and maintenance
convention for t-rust-db's own products, starting with column-rs, so
anyone comparing the two engines' supported SQL reads consistent EBNF
rather than two different documentation styles.

**This does not mean every engine describable this way gets a file
here.** sqlite-rs implements (a growing subset of) real SQLite syntax;
column-rs is a much smaller analytics-over-Parquet language with no
DDL/DML/write path at all. `db-core/sql-parser::row` now implements a
slice of that same SQLite syntax too — as a migrated copy of sqlite-rs's
own code, not a second independent grammar. See `ALIGNMENT.md` for what
was checked and deliberately left unmerged, and `DECISIONS.md` for why
`row` doesn't get its own `.ebnf` file here either.

sqlite-rs's own grammar lives at `sqlite-rs/.openspec/grammar/sqlite.ebnf`
in that repo. This repo references it rather than mirroring a copy here —
see `DECISIONS.md` for why.

## Maintenance rule

Any change to `column-rs`'s grammar (`db-core/sql-parser::column`) MUST
update `column-rs.ebnf` here in the same PR/commit. `sql-parser::row`
has no `.ebnf` file to maintain here (see above) — its grammar changes
alongside sqlite-rs's own `.openspec/grammar/sqlite.ebnf`, which is
sqlite-rs's obligation, not this repo's.
