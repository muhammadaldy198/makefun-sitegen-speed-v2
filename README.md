# makefun-sitegen-speed-v2

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/muhammadaldy198/makefun-sitegen-speed-v2/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/muhammadaldy198/makefun-sitegen-speed-v2/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A fast drop-in replacement for makefun-site 1.0.0: it builds the same static site output tree, byte for byte, in a fraction of the CPU time. Dependency-free ES module (`solve.js`) plus a small Node driver (`main.ts`). This is a total rebuild (v2) of the v1 entry — new architecture throughout, not a patch on it.

## How it works

`run(input)` in `solve.js` reimplements the whole makefun-site build from scratch — front matter, Markdown, templates, collections, permalinks, the report — with the same semantics, tuned for speed:

- **Templates are compiled to JavaScript functions** (`new Function`) instead of interpreted: text nodes become string literals, path lookups become inline-guarded expressions, and loops become plain `for` loops over a reused loop object on one flat mutable scope.
- **Static fragments are rendered once per case.** A template whose reads are only `site`, `tags` and `recent` (nav, header, footer) is rendered once and reused by every page, with guards that disable the cache when a case's templates could make it wrong (include cycles, deep nesting, a loop shadowing a static root).
- **The answers JSON is built directly.** `runJson(input)` emits JSON text as the build runs — the 7 MB object graph and the `JSON.stringify` over it never exist in the graded path — and `main.ts` joins the per-case texts into the answers array.
- **Markdown is one index-based pass.** No `split('\n')`, no token objects: blocks are scanned by line ranges with first-character dispatch, inline markup jumps between structural characters with `indexOf`, and fence bodies are escaped in a single call over the whole slice.
- **No per-page view objects.** The view fields (title, url, date, summary, excerpt, reading time, tags, content, neighbours) are attached to the source objects themselves — the source *is* the page and the card — and link resolution is cached per directory.
- **Front matter is scanned by index**, mirroring the reference's error strings and first-error order exactly.

Measured on the practice set (3 cases, Node 22.23.4 on this VM, cold end-to-end CPU of a fresh `node main.ts` process, combined median of two independent measurement runs): v2 **678 ms** against v1's 989 ms and the house baseline's 1,421 ms; with the null entry's ~194 ms of startup subtracted, v2 is **485 ms** against v1's 795 ms — **~1.6× faster than v1, ~2.5× faster than the baseline**. First-run `run()` CPU (the cold metric, startup excluded): v2 337 ms vs v1 553 ms. Output is byte-identical on the practice cases, 90 fresh generator draws and a set of adversarial inputs (backslash runs, `\>` escapes, unclosed front matter, template include cycles, loop variables named `loop`, CRLF, unicode slugs), and `grade-local --driver process` passes 3/3.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=output node main.ts
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=3p0axm2p&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
