# Decisions

## Reference sqlite-rs's grammar file, don't mirror it (2026-09-02) — SUPERSEDED

**Superseded 2026-09-03** by "sqlite-rs is being absorbed into
`t-rust-db` — promote the grammar file after all" below. Kept here for
history, not because it's still in effect.

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

## Revisited after `db-core/sql-parser`'s row/column split — decision holds (2026-09-03) — SUPERSEDED

**Superseded later the same day** once it became clear sqlite-rs itself
is being absorbed into `t-rust-db` (not just its parser code) — see
"sqlite-rs is being absorbed into `t-rust-db`" below. Both entries'
premise ("sqlite-rs is a separate, pre-existing project" with its own
canonical grammar home) no longer holds; kept here for history.

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

## sqlite-rs is being absorbed into `t-rust-db` — promote the grammar file after all (2026-09-03)

Both decisions above reasoned from "sqlite-rs is a separate,
pre-existing project" with its own canonical grammar home and its own
maintenance process — that was the entire basis for referencing rather
than mirroring. That premise is gone: sqlite-rs is being absorbed into
the `t-rust-db` org (`t-rust-db/sqlite-rs` already exists as the
destination repo). Once sqlite-rs is a `t-rust-db` repo like any other,
"its grammar file lives in a different org's repo, don't duplicate
across org boundaries" no longer applies — it's the same org's grammar,
same as `column-rs.ebnf` already is.

**Decision: promote sqlite-rs's grammar file here, as `sqlite-rs.ebnf`.**
Copied verbatim from `sqlite-rs/.openspec/grammar/sqlite.ebnf` (V-block
annotations, `parse.y` line refs, and all — that provenance metadata is
still accurate and still useful, it just now lives in a second place
too). The header notes the promotion and points back at the canonical
source for now.

**What still needs deciding, not resolved by this entry:**
- Whether `sqlite-rs.ebnf` here becomes the *only* copy (with sqlite-rs's
  own `.openspec/grammar/sqlite.ebnf` removed once absorption completes)
  or a synced copy stays in both places long-term. Depends on how the
  broader sqlite-rs -> `t-rust-db/sqlite-rs` absorption is actually
  carried out (a straight repo move vs. an ongoing two-repo period) —
  outside this repo's scope to decide alone.
- Until that's settled, the Maintenance rule (`README.md`) treats this
  as a synced copy: any change to sqlite-rs's parser grammar (wherever
  that repo currently lives) MUST update `sqlite-rs.ebnf` here in the
  same PR/commit, same obligation `column-rs.ebnf` already carries for
  `db-core/sql-parser::column`.
