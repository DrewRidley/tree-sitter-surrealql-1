# Throughput inputs

Two 4,000-statement files, and the command that reads them:

    npx tree-sitter parse -q -s bench/sign_literal.surql
    npx tree-sitter parse -q -s bench/ordinary.surql

`sign_literal.surql` is `RETURN -1 - -2 + -3 * -4;` repeated — four signed
literals per statement, which is the shape that provokes any ambiguity
around a leading sign. `ordinary.surql` is a plain `SELECT ... WHERE ...`
with no signs, as the control.

Both parse with zero ERROR nodes on this grammar and on `22feaab`, so the
two are directly comparable; an input that only one side can parse measures
error recovery rather than parsing.

## Method

**Interleave the revisions and take the median.** Run one sample of each
revision per round, round-robin, for 50 rounds. Do not run all of one
revision's samples and then the next.

This matters more than it sounds. Machine state drifts over a run, so
measuring revisions in sequence attributes the drift to the revision. Taking
best-of-N makes it worse, because the best sample is the one most sensitive
to that drift. Best-of-three, measured sequentially, reports this branch as
*faster* than `22feaab` on `sign_literal`; interleaved medians over the same
binaries report it as slower. The interleaved figure is the one to trust.

Run every invocation with `TREE_SITTER_LIBDIR` pointed somewhere specific to
your checkout. The CLI caches the compiled library by the grammar's declared
*name*, so two checkouts of this grammar share one artifact and will hand
each other the wrong parser.

## Numbers

Interleaved, n=50, median bytes/ms:

| input          | `22feaab` | with the `[$.Number]` conflict | this branch |
|----------------|-----------|--------------------------------|-------------|
| `sign_literal` |     9,647 |                          5,095 |       8,363 |
| `ordinary`     |    12,157 |                         11,969 |      11,853 |

On `sign_literal` the conflict cost **-47%** against `22feaab`. Dropping it
recovers **+64%** against the conflict, which still leaves this branch about
**13% below** `22feaab`. The regression is mostly, not entirely, paid back.

On `ordinary` the three are indistinguishable: the interquartile ranges
overlap across the whole spread (roughly 10,600–13,900 on every revision),
so that row shows no effect in either direction.

Absolute numbers are machine-specific; the ratios are the portable part.
