# Throughput inputs

Two 4,000-statement files, and the command that reads them:

    npx tree-sitter parse -q -s test/corpus/bench/sign_literal.surql
    npx tree-sitter parse -q -s test/corpus/bench/ordinary.surql

`sign_literal.surql` is `RETURN -1 - -2 + -3 * -4;` repeated — four signed
literals per statement, which is the shape that provokes any ambiguity
around a leading sign. `ordinary.surql` is a plain `SELECT ... WHERE ...`
with no signs, as the control.

Both parse with zero ERROR nodes on this grammar and on `22feaab`, so the
two are directly comparable; an input that only one side can parse measures
error recovery rather than parsing.

Measured on this branch, best of three after two warm-up runs:

| input          | `22feaab` | this branch |
|----------------|-----------|-------------|
| `sign_literal` |    13,056 |      12,557 |
| `ordinary`     |    14,527 |      14,809 |

bytes/ms. With the `[$.Number]` conflict that the signed-number rule used to
need, `sign_literal` ran at 7,457 — about half.

Run every invocation with `TREE_SITTER_LIBDIR` pointed somewhere specific to
your checkout. The CLI caches the compiled library by the grammar's declared
*name*, so two checkouts of this grammar share one artifact and will hand
each other the wrong parser.
