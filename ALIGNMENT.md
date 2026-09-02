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

### 3. No shared opcode set (by design, not oversight)

sqlite-rs's VDBE has ~65 SQLite-specific opcodes (`src/vdbe/program.rs`);
column-rs's VM has 10 columnar/vectorized opcodes (`src/vm.rs`). These
were evaluated for sharing during the September 2026 architecture
discussion (see `projects/database-rs/unified-vm-vision.md` outside this
repo) and deliberately kept separate — one is row/cursor-oriented and
B-tree-coupled, the other is batch/columnar and Parquet-oriented. Nothing
to reconcile here; recorded so a future reader doesn't rediscover the
same question.

## What to check again before any future integration

- If `sql-types::Value` ever gains real users (beyond the placeholder
  `Literal`-conversion it has today), re-diff it against both
  `column_rs::vm::Value` and sqlite-rs's `Value` before assuming it's
  "the" shared type.
- If t-rust-db repos ever move under an org that wants copyright headers,
  decide the actual copyright holder explicitly — don't default to
  sqlite-rs's.
