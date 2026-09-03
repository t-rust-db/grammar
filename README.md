# grammar

SQL grammar reference for t-rust-db products, in standard EBNF.

- **`sql.ebnf`** — the single grammar source of truth, in three sections:
  1. **Shared** — productions genuinely identical between column-rs and
     sqlite-rs (an identifier's unquoted character shape, the `EXPLAIN
     [QUERY PLAN]` prefix). Small and deliberately conservative — see
     the file's own header for what was checked and rejected.
  2. **column-rs only** — column-rs's actual supported SQL subset,
     derived directly from `db-core/sql-parser`'s `column` section's
     parser functions (not aspirational — every rule has a corresponding
     parser function that accepts it).
  3. **sqlite-rs only (including DDL)** — sqlite-rs's full SQL grammar:
     DDL, DML, transactions, `PRAGMA`, and its own richer expression
     grammar. Promoted here from sqlite-rs's own `.openspec/grammar/
     sqlite.ebnf` now that sqlite-rs itself is being absorbed into the
     `t-rust-db` org (`t-rust-db/sqlite-rs`) rather than staying a
     separate, pre-existing project.

Previously two separate files (`column-rs.ebnf`/`sqlite-rs.ebnf`); see
`DECISIONS.md` for why they were unified into one, and what actually
qualifies as "shared" versus merely similar-looking.

## `db-core/sql-parser`'s two sections

As of `db-core` ADR 0002, `sql-parser` is one crate with two
Cargo-feature-gated sections, mirroring `sql-vm`'s `batch`/`row`/`stream`
split:

- **`column`** — column-rs's analytics subset. What `sql.ebnf`'s section
  2 describes.
- **`row`** — sqlite-rs's own grammar (DDL, DML, transactions, `PRAGMA`,
  ...), migrated in from sqlite-rs's `src/parser/*` (tokenizer, AST,
  recursive-descent parser, three-way parse outcome, AST printer) as a
  mechanical port — not an independent reimplementation. Its own AST is
  **not** `sql_expr::Query` (see ADR 0002's amendment); it's sqlite-rs's
  AST, unchanged. What `sql.ebnf`'s section 3 describes.

## Why this repo exists

This repo is the place t-rust-db's SQL grammars live, in one consistent
EBNF notation, so anyone comparing the org's engines' supported SQL
reads the same documentation style rather than several different ones.
column-rs is a much smaller analytics-over-Parquet language with no
DDL/DML/write path at all; sqlite-rs implements (a growing subset of)
real SQLite syntax. See `ALIGNMENT.md` for what was checked across the
two, and `DECISIONS.md` for the sqlite-rs-grammar-file and file-merge
history.

## Maintenance rule

Any change to either engine's grammar MUST update `sql.ebnf`'s
corresponding section in the same PR/commit:
- `db-core/sql-parser::column` changes -> update section 2.
- `db-core/sql-parser::row` changes, or sqlite-rs's own parser changes
  (`sqlite-rs/.openspec/grammar/sqlite.ebnf`) -> update section 3 here
  too, until sqlite-rs's absorption into `t-rust-db` is far enough along
  that this becomes the only copy (see `DECISIONS.md`).

Before editing a rule in section 1 (Shared), re-verify it's still
identical in both engines AND still self-contained (see `sql.ebnf`'s own
header for what that means) — if either stops being true, move the rule
back into each engine's own section instead of leaving a shared rule
that's only accurate for one of them.
