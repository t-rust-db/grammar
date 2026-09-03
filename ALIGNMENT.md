# Alignment with sqlite-rs

sqlite-rs is a separate, pre-existing project (github.com/iheitlager/
sqlite-rs, developed under a Schuberg Philis-owned context). t-rust-db is
a new, separate org. This note records what was checked to avoid future
collisions if/when the two ever need to interoperate (e.g. a shared
`RowExecutor`, a shared type, or a shared grammar file) — and what was
deliberately left unresolved rather than silently decided.

## Checked and fine

- **Crate namespacing.** t-rust-db's crates (`sql-types`, `sql-expr`,
  `sql-parser`, `sql-join`, `db-storage`, `db-parquet`, `db-cli`) don't
  collide with any sqlite-rs crate name — sqlite-rs is a single crate
  (`sqlite-rs`), not published as a set of libraries.
- **Grammar notation.** This repo's EBNF style (notation, V-block-less
  "not supported" section, maintenance rule) mirrors sqlite-rs's
  `.openspec/grammar/sqlite.ebnf` conventions, so the two read the same
  way side by side without actually merging into one file (they describe
  genuinely different languages — see `README.md`).

## Real collisions found — deliberately NOT resolved here

### 1. Three `Value` types exist across the ecosystem

| Type | Location | Variants |
|------|----------|----------|
| `sqlite_rs::record::value::Value` | sqlite-rs, `src/record/value.rs` | SQLite's own storage classes |
| `sql_types::Value` | t-rust-db/db-core, `sql-types/src/lib.rs` | `Int, Float, Str, Null` |
| `column_rs::vm::Value` | t-rust-db/column-rs, `src/vm.rs` | `Int, Float, Bool, Str(Cow), Null` |

`sql_types::Value` is currently **unused** by column-rs's own VM — it was
extracted speculatively during the September 2026 restructure (no runtime
`Value` existed in the original `sql.rs`, one was added "per spec"). The
VM still runs entirely on its own, richer `column_rs::vm::Value`.

**Not resolved:** which (if any) of these should become the one shared
`Value` type is an open design question (tracked in `projects/
database-rs/query-plan-comparison.md` and `architecture.md` outside this
repo), not something to decide inside a grammar/alignment pass. Building
`sql-join`'s `JoinHashTable` generic (see `db-core/sql-join`) rather than
hardcoding any one `Value` was a direct consequence of leaving this open.

### 2. Copyright header convention

Every sqlite-rs source file carries:
```rust
// Copyright 2026 Schuberg Philis
// SPDX-License-Identifier: Apache-2.0
```

t-rust-db repos currently carry **no per-file copyright header** — just
doc comments, with an `Apache-2.0` `LICENSE` file at the repo root and
`license.workspace = true` in `Cargo.toml`.

**Deliberately not copied.** t-rust-db is not a Schuberg Philis-owned
org; stamping "Copyright 2026 Schuberg Philis" on files that aren't part
of that codebase would be a false attribution, not an alignment. If
t-rust-db wants a per-file header at some point, it should name its own
actual owner/org, decided explicitly rather than inherited by copy-paste
from a different project's convention.

### 3. Opcode sets: reversed decision (2026-09-02, same day)

**Original call (superseded):** sqlite-rs's VDBE has ~65 SQLite-specific
opcodes (`src/vdbe/program.rs`); column-rs's VM had 10 columnar/vectorized
opcodes (in column-rs's own private `src/vm.rs` at the time). These were
evaluated for sharing during the initial architecture discussion (see
`projects/database-rs/unified-vm-vision.md` outside this repo) and kept
deliberately separate — `RowExecutor` inside sqlite-rs only, `BatchExecutor`
shared.

**Reversed same day.** column-rs's VM was extracted into `db-core/sql-vm`
as `sql_vm::batch` (the `BatchExecutor`), and `sql-vm` was structured with
room for all three executors — `batch` (implemented), `row` and `stream`
(stubs) — rather than splitting "the VM" across two repos by execution
strategy. `row` (the eventual `RowExecutor` home) is explicitly a
placeholder, not a port of sqlite-rs's actual 65-opcode VDBE — whether it
ends up sharing an opcode *set* with sqlite-rs or just the row-at-a-time
*execution strategy* while keeping distinct opcodes is still open (see
`sql-vm/src/row.rs`'s own doc comment).

**What this means for sqlite-rs specifically:** nothing has moved out of
sqlite-rs yet, and sqlite-rs's own VDBE is untouched — `db-core/sql-vm`
is a new, so-far-column-rs-only crate that happens to have room reserved
for a row executor. Re-check this section once `row.rs` gets real content
or sqlite-rs is asked to depend on it.

### 4. `JoinKind` vs sqlite-rs's `JoinOp` — closed (2026-09-03)

Checked as of 2026-09-02 while working issue #1: `sql_expr::JoinKind`
(`db-core/sql-expr/src/lib.rs`) had only `Inner`/`Left`, while
sqlite-rs's `JoinOp` also has `Cross`/`Right`/`Full` — the companion
db-core work to grow `sql-parser`/`sql-expr` with `SELECT *`, table
aliases, and `CROSS`/`RIGHT`/`FULL JOIN` was still in flight.

**Closed.** `sql_expr::JoinKind` now has `Inner`/`Left`/`Right`/`Full`/
`Cross` — the same five sqlite-rs's `JoinOp` has (`CROSS JOIN` requires
an accompanying top-level `LIMIT`, `db-core/sql-parser::column`'s own
DO-178C bounded-execution rule). No gap remains between the two engines'
`JOIN` vocabularies.

### 5. Parser convergence: `sql-parser` now houses sqlite-rs's own grammar too (2026-09-03)

Section 3's opcode-set reversal ("consolidate all three `sql-vm`
executors in one crate, not split by strategy across repos") has a
parser-layer counterpart: `db-core#19`/`#23` restructured `sql-parser`
into one crate with `column`/`row` Cargo-feature sections (`db-core` ADR
0002), migrating sqlite-rs's entire `src/parser/*` (tokenizer, AST,
recursive-descent parser, three-way parse outcome, AST printer) into
`sql_parser::row` as a mechanical port.

**What this means for sqlite-rs specifically:** same as section 3 —
nothing has moved *out* of sqlite-rs. `sqlite-rs/src/parser/*` is
untouched; `db-core/sql_parser::row` is an independent copy, not a
shared dependency sqlite-rs itself pulls in. Whether sqlite-rs ever
switches to depending on `sql_parser::row` instead of its own copy is a
separate, larger question (its own crate publishing/versioning story)
not decided by this convergence.

**What this means for this repo:** see `DECISIONS.md`'s entry
"Revisited after `db-core/sql-parser`'s row/column split — decision
holds" — `column-rs.ebnf` remains this repo's only `.ebnf` file.
`sql_parser::row` doesn't get one: it's a copy of sqlite-rs's grammar,
not an independent one, so sqlite-rs's own `.openspec/grammar/
sqlite.ebnf` already covers it.

## What to check again before any future integration

- If `sql-types::Value` ever gains real users (beyond the placeholder
  `Literal`-conversion it has today), re-diff it against both
  `column_rs::vm::Value` and sqlite-rs's `Value` before assuming it's
  "the" shared type.
- If t-rust-db repos ever move under an org that wants copyright headers,
  decide the actual copyright holder explicitly — don't default to
  sqlite-rs's.
