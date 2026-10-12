# DeepSeek V4 Flash Base: quantization comparison (static page)

Static, dark, mobile-friendly page comparing 5 GGUF variants of DeepSeek-V4-Flash-Base on 12 prompts:
Q8_0, protected Q4 and Q4_K_M (10 samples per prompt, run 2026-10-11 JST), plus two imatrix builds added
2026-10-12 JST with 5 samples per prompt (s0-s4, same seeds and settings):
- Protected Q4 + imatrix (C1): same protected layout as Protected Q4, quantized with an importance matrix (175.67 GB, 4.94 BPW).
- Q5_K key layers + imatrix (C3): protected layout plus Q5_K for all ffn_down experts and ffn_gate/up experts in layers 0-2 and 33-42 (182.56 GB, 5.14 BPW).
FP8 was not run. For s5-s9 the imatrix columns show "not run", and blind view only shuffles variants that have the selected sample.

## Files
- `index.html`: the page (HTML + CSS + JS, no dependencies). The data is embedded in a
  `<script type="application/json">` block, so it works from `file://` (browsers block `fetch` of local files) and on GitHub Pages.
- `data.json`: the same data as a standalone file (linked from the page footer; handy for other tools).
- `template.html`, `build.py`: how index.html/data.json are generated. Only needed to rebuild.

## How it was built
`python3 build.py` reads `/workspace/ds4-research/compare/results/{A,B,D}_P{01..12}_s{00..09}.json` and `{C1,C3}_P{01..12}_s{00..04}.json`,
`prompts.json` and `{A,B,C1,C3,D}_SUMMARY.json`, keeps per sample: text, completion tokens, finish reason,
per-stream tokens/sec (llama.cpp `timings.predicted_per_second`), wall seconds, seed. It writes `data.json`
and injects the same JSON into `template.html` (with `<` escaped as `\u003c`) to produce `index.html`.
Total ~0.63 MB.

All statistics are computed in the browser from the raw samples (nothing is hand-entered):
mean/median tokens, stop vs length counts, distinct-2 (unique word bigrams / total, split on whitespace;
"within" = mean over single samples, "pooled" = across the 10 samples), shared leading characters with
the Q8_0 output of the same seed, median per-stream tok/s. These are rough surface metrics, not quality scores;
the page states that judgement is left to the reader.

Features: prompt tabs with full prompt text; sample switcher s0..s9 (also arrow keys / buttons) that changes all three
variants together; per-variant "Expand all 10"; blind view (variant names hidden, order shuffled per prompt+sample with a
deterministic seeded shuffle, stats hidden, "Reveal labels" button); URL hash `#P03/s2` (`/blind` suffix) restores state.
All model text is inserted via `textContent` inside `white-space: pre-wrap` boxes (prompt shown dim, continuation bright).

## Test
Served locally with `python3 -m http.server` and via `file://`; rendered with headless Google Chrome (screenshots at 390 px and
~1100-1280 px) and driven by a script in an iframe (prompt/sample switching, expand, blind, reveal, stats tables).
Published at https://imaginalnika.github.io/ds4-quant-compare/ .

## Run details (from the run's notes)
2x RTX PRO 6000 (96 GB), llama.cpp `aa94f20`, `/v1/completions`, 12 parallel requests, temperature 0.85 (P01-P06) / 1.0 (P07-P12),
top_p 0.95, max_tokens 300, other settings llama.cpp defaults (top_k 40, min_p 0.05). Seed = 1000*prompt# + sample#.
Q8_0 ran with about half of the routed experts on CPU (speed only), so tokens/sec are not comparable across variants.
