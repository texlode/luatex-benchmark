# luatex-benchmark

Reproducible benchmarks accompanying the paper **"Real-Time LuaTeX:
Recompiling Large Documents in 1 ms"** (TUGboat, 2026).

These scripts measure how long **vanilla LuaLaTeX** (TeX Live 2025, no engine
patches) needs to typeset a single paragraph, and compare that against
**Typst** in watch mode. They time *public engines doing public operations* —
there is no `texlode` code here. The point is narrow and verifiable: a
paragraph compiles in roughly 1 ms inside a live LuaTeX session, constant in
document size.

## Requirements

- TeX Live 2025 (provides `lualatex`), with the `microtype`, `amsmath`, and
  `lipsum` packages
- Typst 0.14+ (only for the comparison in `typst-comparison/`)
- A POSIX shell for the Typst scripts (Git Bash on Windows is fine)

## What reproduces which table

| Script | Paper section / table |
| --- | --- |
| `paragraph-benchmark.tex` | §3, the self-contained one-paragraph timing MWE |
| `systematic-benchmark.tex` | §3, the short/medium/long + inline/display-math table (median / P5 / P95) |
| `stability-benchmark.tex` | §"Why one paragraph is enough", the 500-compile no-degradation table |
| `typst-comparison/bench.mjs` | §"Comparison with Typst", the Typst column of the ratio table |

> **What this repo measures, and what it does not.** The LuaLaTeX scripts here
> are a clean-room, engine-independent demonstration of the paper's core
> claim: that Knuth–Plass line breaking for one paragraph costs ~1 ms on stock
> TeX Live, constant in document size. They time line breaking inside a
> `\vbox` using `\directlua`. The paper's headline table is measured inside
> texlode's resident engine (line break **+** node traversal **+** IPC), so
> these vanilla numbers corroborate the paper's *pipeline breakdown* (the
> ~80 % attributed to "LuaTeX line break + traversal") rather than reproducing
> every product-pipeline cell exactly. No texlode code is included or required.

## Running

```sh
# single-paragraph sanity check — prints one line to the terminal
lualatex paragraph-benchmark.tex

# systematic table (median / P5 / P95 per category)
lualatex systematic-benchmark.tex   # summary printed to terminal + .log

# stability across 500 consecutive in-session compiles
lualatex stability-benchmark.tex

# Typst comparison (drives the `typst` CLI only — no texlode involved)
cd typst-comparison
node bench.mjs 10  30
node bench.mjs 100 30
node bench.mjs 300 30
```

## Methodology notes

- **In-session, not cold start.** The paragraph is typeset inside a running
  LuaLaTeX process. A cold `lualatex` invocation costs ~1 s of preamble/font
  startup, which is irrelevant to interactive editing and is *not* measured.
- **Line breaking is isolated from page breaking.** Each timed paragraph is
  built inside a `\vbox`, so the cost measured is Knuth–Plass line breaking,
  not the page builder / output routine. This matches the paper's claim that
  line breaking is the dominant, paragraph-local cost.
- **Warmup.** The first `WARMUP` iterations per category are discarded before
  computing statistics.
- **Hardware-dependent.** The paper's figures are from an Intel Core
  i7-1355U. Absolute numbers will differ on other machines; the *shape*
  (constant in document size, dominated by line count) should not.

## License

MIT — see `LICENSE`.
