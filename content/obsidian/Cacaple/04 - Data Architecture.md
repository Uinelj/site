---
tags: [ai-authored, projects, cacaple]
---

# Data Architecture

## Overview

**All game data is embedded directly in the JavaScript bundle.** There are no API calls. This makes the game fully static and offline-capable.

## Three Data Sets

### 1. `wordDist` — Word Distance Dictionary (~7,417 entries)

Every valid 4-letter word with its **minimum number of steps to reach "POOP"** via BFS.

```
POOP,0
POOS,1
POOD,1
POOR,1
POOL,1
POPS,1
PODS,2
POLL,2
...
PICK,5
DAWN,5
...
AUGH,11     (furthest words)
```

**How it's computed** (offline, before build):
1. Start BFS from "POOP" (distance 0)
2. Find all words 1 letter different → distance 1
3. From those, find all new words 1 letter different → distance 2
4. Repeat until all reachable words are mapped

This also implicitly defines the **word list** — if a word has a distance, it's valid.

### 2. `wordFrequency` — Word Frequency Scores (~7,417 entries)

Popularity/commonality scores for each word. Used to prefer "nicer" words when showing the optimal path.

```
THAT,1587193
THIS,1234567
POOP,1587193
...
BAJU,15125
MOZZ,13997
```

Used in `getLogFrequency(word) = Math.log(wordFrequencyDict[word])` to pick the "best" shortest path (through more common words).

### 3. `startWords` — Daily Start Words (~1,137 entries)

Curated list of start words with their distance (par):

```
PICK,5
DAWN,5
OPTS,6
JIVE,6
HALE,5
AVID,7
FARM,6
...
```

Selected by index: `startWords[daysSinceEpoch()]`

## localStorage Schema

| Key | Type | Description |
|-----|------|-------------|
| `guesses` | `string[]` (JSON) | Current game's word chain: `["pick", "peck", "peek", "peep", "poop"]` |
| `games` | `object` (JSON) | Win history: `{ "0": 3, "1": 5, "2": 2 }` → keys are "extra guesses", values are count |
| `streak` | `number` (JSON) | Current consecutive daily wins |
| `bestStreak` | `number` (JSON) | All-time best streak |
| `dateLastPlayed` | `number` (JSON) | Timestamp (ms) of last game started |
| `dateLastWon` | `number` (JSON) | Timestamp (ms) of last game won |

### Game Initialization Logic

```
On page load:
  if (not played today OR guesses empty OR guesses[0] ≠ today's start word):
      reset guesses to [startWord]
  if (not won yesterday AND not won today):
      reset streak to 0
  set dateLastPlayed = now
```

### Win Logic

```js
function winGame(extraGuesses) {
    const games = getGames();
    games[extraGuesses] = (games[extraGuesses] ?? 0) + 1;
    localStorage.setItem("games", JSON.stringify(games));
    
    const streak = getStreak() + 1;
    setStreak(streak);
    if (streak > getBestStreak()) setBestStreak(streak);
    setDateLastWon(Date.now());
}
```

`extraGuesses = total_guesses - 1 - par` (minus 1 because the start word counts as a "guess" in the array but isn't a player move)

## For Our Clone

We need to precompute:
1. **French 4-letter word BFS distances to "CACA"** (or chosen target)
2. **French word frequencies** from a corpus
3. **Curated daily start words** at varied difficulty levels (distance 4-8)
