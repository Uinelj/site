---
tags: [ai-authored, projects, cacaple]
---

# Share & Social

## Share Text Format

When a player wins and taps "Copy results", this text is copied to clipboard:

```
Poople #42 6/5
🟫⬜⬜⬜
⬜⬜⬜⬜
⬜⬜🟫⬜
⬜🟫⬜🟫
⬜🟫🟫🟫
🟫🟫🟫🟫

https://poople.io/
```

### Format Breakdown

```
{Game Name} #{puzzle_number} {your_moves}/{par}
{grid}

{url}
```

### Grid Generation

```js
for (const word of guesses) {
    for (let i = 0; i < 4; i++) {
        if (word.toLowerCase()[i] === "poop"[i]) {
            text += "🟫";   // brown square = letter matches target position
        } else {
            text += "⬜";   // white square = no match
        }
    }
    text += "\n";
}
```

Only two emoji states:
- 🟫 Brown = correct letter at correct position (matches POOP)
- ⬜ White = different letter

This is simpler than Wordle (no yellow/misplaced state) — it's purely positional matching against the target.

## Stats Display

### Totals Panel
- **Wins** — Total games won
- **Avg. Extra Guesses** — Mean of (your moves - par) across all games
- **Current Streak** — Consecutive daily wins
- **Best Streak** — All-time record

### Histogram
Bar chart showing distribution of extra guesses:
- X axis: extra guesses (0, 1, 2, 3, ...)
- Y axis: number of games

### Results Panel
> You used **6 guesses.**
> The best solution was **5 guesses.**
> [Copy results]

## Yesterday Modal

Shows:
- Yesterday's start word
- The shortest distance (par)
- How many paths existed ("was only one way" / "were a few ways" / "were several ways")
- The optimal path displayed as Row components

```js
// Tree width determines the message:
const width = getTreeWidth(yesterdayWord);
if (width === 1) "was only one way"
else if (width < 5) "were a few ways"
else "were several ways"
```

## For Our Clone (Cacaple)

Share text would be:
```
Cacaple #1 5/4
🟫⬜⬜⬜
⬜⬜⬜🟫
⬜⬜🟫🟫
⬜🟫🟫🟫
🟫🟫🟫🟫

https://cacaple.fr/
```
