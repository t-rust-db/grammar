# grammar

SQL grammar reference for t-rust-db products, in standard EBNF.

- **`column-rs.ebnf`** — column-rs's actual supported SQL subset, derived
  directly from `db-core/sql-parser`'s `column` section's parser functions
  (not aspirational — every rule here has a corresponding parser function
  that accepts it).
- **`sqlite-rs.ebnf`** — sqlite-rs's full SQL grammar, promoted here from
  sqlite-rs's own `.openspec/grammar/sqlite.ebnf` now that sqlite-rs
  itself is being absorbed into the `t-rust-db` org (`t-rust-db/
  sqlite-rs`) rather than staying a separate, pre-existing project. See
  `DECISIONS.md` for the history of this call (it was originally "don't
  mirror," reversed once the absorption changed the premise).

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
  AST, unchanged. What `sqlite-rs.ebnf` describes.

## Why this repo exists

This repo is the place t-rust-db's SQL grammars live, in one consistent
EBNF notation, so anyone comparing the org's engines' supported SQL
reads the same documentation style rather than several different ones.
column-rs is a much smaller analytics-over-Parquet language with no
DDL/DML/write path at all; sqlite-rs implements (a growing subset of)
real SQLite syntax. See `ALIGNMENT.md` for what was checked across the
two, and `DECISIONS.md` for the sqlite-rs-grammar-file history.

## Maintenance rule

Any change to a section's grammar MUST update the corresponding `.ebnf`
file here in the same PR/commit:
- `db-core/sql-parser::column` changes -> update `column-rs.ebnf`.
- `db-core/sql-parser::row` changes, or sqlite-rs's own parser changes
  (`sqlite-rs/.openspec/grammar/sqlite.ebnf`) -> update `sqlite-rs.ebnf`
  here too, until sqlite-rs's absorption into `t-rust-db` is far enough
  along that this becomes the only copy (see `DECISIONS.md`).
