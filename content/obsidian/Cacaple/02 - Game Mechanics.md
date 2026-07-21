---
tags: [ai-authored, projects, cacaple]
---

# Game Mechanics

## Core Rules

1. A **new 4-letter start word** is given each day
2. The player must reach the target word **"POOP"** (for us: **"CACA"**)
3. Each move: change **exactly one letter** to form a new valid word
4. Every intermediate word must be in the **word dictionary** (~7,400 words)
5. Goal: reach the target in as **few steps as possible**

## Day Numbering

```js
const UTC_GAME_CHANGE_HOUR = 8;

function daysSinceEpoch(e) {
    const a = new Date;
    a.setUTCFullYear(2025, 7, 15);     // Aug 15, 2025
    a.setUTCHours(UTC_GAME_CHANGE_HOUR, 0, 0, 0);
    let i = e ?? Date.now(), t = 0;
    for (; i > a.getTime();) {
        i = new Date(i).setDate(new Date(i).getDate() - 1);
        t += 1;
    }
    return t;
}
```

- **Epoch:** August 15, 2025 at 08:00 UTC
- Puzzle number = days since epoch
- New puzzle every day at 08:00 UTC (10:00 Paris time in summer)

## Start Word Selection

```js
function getStartWord(e) {
    const t = e ?? daysSinceEpoch();
    return startWords[t];   // simple index into curated list
}
```

- ~1,137 curated start words with known distances (e.g. `PICK,5`, `DAWN,5`, `AVID,7`)
- The distance number = minimum steps to POOP (the "par" for the puzzle)

## Word Validation

```js
function isValidWord(word, previousWord) {
    return isInWordList(word)              // must be in dictionary
        && word.length === 4               // must be 4 letters
        && oneLetterDifferent(word, previousWord);  // exactly 1 change
}

function oneLetterDifferent(a, b) {
    if (a.length !== b.length) return false;
    let diff = 0;
    for (let i = 0; i < a.length; i++) {
        if (a[i].toLowerCase() !== b[i].toLowerCase()) diff++;
    }
    return diff === 1;
}
```

## Distance Computation (Precomputed BFS)

Every word in the dictionary has a precomputed **minimum distance to "POOP"** stored in `wordDist`:

```
POOP,0
POOL,1
POLL,2
PILL,3
PICK,5
...
```

This is done offline via BFS from "POOP", radiating outward through all adjacent words.

## Optimal Path Algorithm

```js
function buildTree(startWord) {
    // Builds a tree of all shortest paths from POOP to startWord
    // Uses the precomputed distances to guide a layer-by-layer expansion
}

function getBestTraversal(tree, depth, word) {
    // Traverses the tree, preferring paths through 
    // higher-frequency (more common) words
    // Uses getLogFrequency(word) = Math.log(wordFrequencyDict[word])
}

function getShortestPath(startWord) {
    const tree = buildTree(startWord);
    return getBestTraversal(tree, tree.length - 1, startWord).path;
}
```

The shortest path is displayed in the **"Yesterday" modal** to show players the optimal solution.

## Game Flow

```
1. Page loads → getStartWord() picks today's word
2. Word goes into guesses[0]
3. Player types 4-letter words on keyboard
4. Each word validated: in dictionary + 1 letter different
5. If word === "POOP" → game over, show Results
6. Results show: your guesses / par (shortest path)
7. "Extra guesses" = your_guesses - par → stored in stats
```

## Win Condition

The game is won when the last guess is `"poop"`. There is no lose condition — you can keep guessing indefinitely.
