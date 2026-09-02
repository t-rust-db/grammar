# grammar

SQL grammar reference for t-rust-db products, in standard EBNF.

- **`column-rs.ebnf`** — column-rs's actual supported SQL subset, derived
  directly from `db-core/sql-parser`'s parser functions (not aspirational —
  every rule here has a corresponding parser function that accepts it).

## Why this repo exists

sqlite-rs (a separate, pre-existing project — see
`.openspec/grammar/sqlite.ebnf` there) already tracks the full SQLite
grammar this way. This repo mirrors that notation and maintenance
convention for t-rust-db's own products, starting with column-rs, so
anyone comparing the two engines' supported SQL reads consistent EBNF
rather than two different documentation styles.

**This does not mean the two engines share a grammar file, or will.**
sqlite-rs implements (a growing subset of) real SQLite syntax; column-rs
is a much smaller analytics-over-Parquet language with no DDL/DML/write
path at all. See `ALIGNMENT.md` for what was checked and deliberately
left unmerged.

## Maintenance rule

Any change to a parser's grammar (currently: `db-core/sql-parser`) MUST
update the corresponding `.ebnf` file here in the same PR/commit.
