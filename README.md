# makefun-fast-site

A faster drop-in replacement for **makefun-site 1.0.0**, make.fun's static site
generator, for the
[site-generator-speed-01](https://make.fun/bounty/site-generator-speed-01-run-20261122)
bounty.

`solve.js` exports `run(input)` that returns **byte-identical** output to
`makefun-site` 1.0.0's `build(input)` for every input: a JSON object mapping
each output path (each page's `index.html`, the index and tag pages,
`build-report.txt`, and every static file) to that file's text.

It is a single dependency-free ES module. `main.ts` is the harness driver: it
reads every case from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## Correctness

Verified byte-exact against the reference on:

- all practice cases (`practice/input/cases.json` vs `practice/reference/expected.json`),
  through the real scorer: `passed: 1, cases_passed: 3, cases_total: 3`;
- 12,000+ randomly generated sites across three fuzz suites (markdown blocks and
  inline, front-matter edge cases, and template parser edge cases), each compared
  key-for-key against the reference.

Output is deterministic: two runs on the same input produce byte-identical
`answers.json`.

## Speed

Measured on the practice set (3 cases), CPU time per set, node 22, with the
harness's `--no-incremental-marking` flag:

| implementation | ms / set |
| -------------- | -------- |
| makefun-site 1.0.0 (baseline) | ~558–808 |
| makefun-fast-site | ~220–302 |

Roughly **2.5–2.7× faster** than the baseline, byte-identical output.

## How it is faster

The reference is a metalsmith-style pipeline of plugins over one build, with
`marked`-style Markdown. This version keeps the exact algorithms but:

- **batches plain-character runs** in the Markdown inline renderer and escapes
  each run in one pass, instead of calling the HTML escaper once per character;
- **fast-paths the HTML escaper** so strings with no special characters are
  returned without building a new string;
- **counts words without allocating** a match array for `reading_minutes`;
- **reuses each post's card** when rendering its layout, and caches each tag's
  card form, instead of recomputing them;
- **clears its per-build slug/date caches** at the start of every `run` so
  nothing carries from one case to the next (each case is answered alone).

## Usage

```
node /bot/main.ts        # reads /input/cases.json, writes /output/answers.json
```

Outside a container, set `MAKEFUN_INPUT_DIR` and `MAKEFUN_OUTPUT_DIR`.

## License

MIT — see [LICENSE](LICENSE).