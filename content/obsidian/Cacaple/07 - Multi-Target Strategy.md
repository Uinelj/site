---
tags: [ai-authored, projects, cacaple]
---

# Multi-Target Strategy

> **Addendum — July 2026**
> After initial analysis, CACA alone may have limited graph connectivity in French. We're planning to use **multiple target words** to keep puzzles varied and reachable.

## Target Words

| Target | Meaning | Why |
|--------|---------|-----|
| **CACA** 💩 | Poop | The brand — thematic equivalent of POOP |
| **PIPI** 🚽 | Pee | Same scatological family, probably strong connectivity (common letter patterns: P and I are frequent) |
| **VOMI** 🤢 | Vomit | Different letter distribution (V, O, M) — opens up different regions of the word graph |

## Why Multiple Targets?

In Poople, the only target is POOP. This works because the English 4-letter word graph is dense — most words can reach POOP in ≤11 steps.

French has fewer 4-letter words (especially without accents), so a single target like CACA may:
- Have a **small connected component** (C and K are rare in French)
- Leave many common words **unreachable**
- Result in **very long or repetitive** optimal paths

By rotating between CACA, PIPI, and VOMI, we:
1. **Cover more of the word graph** — each target reaches different word clusters
2. **Vary the difficulty** — some targets are easier, some harder
3. **Keep the theme** — all three are "gross body stuff" 💩🚽🤢
4. **Add variety** — players don't always head to the same ending

## Proposed Day Rotation

Option A — **Fixed cycle:**
```
Day 0: CACA
Day 1: PIPI
Day 2: VOMI
Day 3: CACA
Day 4: PIPI
...
```

Option B — **Curated per day** (each start word is paired to its best target):
```
startWords = [
    { word: "LUNE", target: "PIPI", distance: 5 },
    { word: "MARE", target: "CACA", distance: 4 },
    { word: "DOUX", target: "VOMI", distance: 6 },
    ...
]
```

Option B is better — it lets us pick the most fun puzzles regardless of which target they lead to.

## Impact on Architecture

| What changes | How |
|-------------|-----|
| **`wordDist`** | Need 3 separate BFS maps: `wordDistCaca`, `wordDistPipi`, `wordDistVomi` |
| **`startWords`** | Each entry includes `target` + `distance` |
| **`getStartWord()`** | Also returns today's target |
| **Highlight logic** | `Box.highlight` compares against the day's target, not a hardcoded word |
| **Share text** | Shows target: `Cacaple #42 → CACA 🎯 5/4` |
| **Yesterday modal** | Shows yesterday's target + optimal path |
| **Bundle size** | ~3× the word distance data (but still small — ~200KB total) |

## Precomputation Tasks

- [ ] Run BFS from **CACA** — record distances, count reachable words
- [ ] Run BFS from **PIPI** — record distances, count reachable words
- [ ] Run BFS from **VOMI** — record distances, count reachable words
- [ ] Compare coverage: which words are reachable from which targets?
- [ ] Union of all three → total playable word set
- [ ] For each reachable word, pick the best target (shortest distance, most paths)
- [ ] Curate start words with `(word, target, distance)` triples

## Fallback

If CACA turns out to have great connectivity after all, we can simplify back to a single target and keep PIPI/VOMI as occasional "special" puzzles (e.g. weekends, themed days).
