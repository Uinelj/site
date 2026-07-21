---
tags: [ai-authored, projects, cacaple]
---

# UI Components

## Component Tree

```
App
├── Header
│   ├── Title ("Poople" + 💩 logo)
│   ├── Puzzle number (#42: PICK)
│   ├── ℹ️ button → opens Intro modal
│   ├── 📊 button → opens Stats modal
│   └── 📅 button → opens Yesterday modal
├── GameArea
│   ├── RowContainer
│   │   └── Row[] (one per guess)
│   │       └── Box[] (4 per row — one per letter)
│   └── Keyboard
│       └── Key[]
├── Modal (generic wrapper)
│   ├── Intro (how to play)
│   ├── Stats
│   │   ├── Results (guess count, par, copy button)
│   │   ├── Totals (wins, avg, streaks)
│   │   ├── Histogram (guess distribution)
│   │   └── Countdown (time to next puzzle)
│   └── Yesterday
│       └── Row[] (showing optimal path)
├── EmojiRain (💩 falling animation on win)
├── Footer
├── Privacy (route: /privacy)
└── Support (route: /support)
```

## Key Components

### `Row`
Displays a 4-letter word as 4 `Box` components.

```jsx
function Row({ word, suppressHighlight }) {
    // For each letter position (0-3):
    // highlight = letter matches "poop" at that position
    return <div className="Row">{boxes}</div>;
}
```

### `Box`
A single letter tile.
- Normal: dark background
- **Highlighted** (class `highlight`): brown/poop-colored if the letter matches POOP at that position

```jsx
function Box({ letter, highlight }) {
    return (
        <div className={"Box" + (highlight ? " highlight" : "")}>
            {letter?.toUpperCase() ?? ""}
        </div>
    );
}
```

### `RowContainer`
Wraps all rows. Adds `jump` CSS class on game over (bounce animation).
Shows `ErrorMessage` for invalid words.

### `Keyboard`
On-screen keyboard with 3 rows of keys. Calls `onKeyPress(key)` on tap.

### `EmojiRain`
When the player wins, 💩 emojis rain down from the top of the screen.
Uses random positions, delays, and durations for each emoji.

## CSS Classes

| Class | Element |
|-------|---------|
| `.App` | Root container |
| `.Header` | Top bar |
| `.GameArea` | Game play area |
| `.RowContainer` / `.row-container` | All word rows |
| `.Row` | Single word row |
| `.Box` | Single letter tile |
| `.Box.highlight` | Letter matches target at that position (brown) |
| `.Keyboard` / `.KeyboardRow` / `.Key` | Keyboard |
| `.Modal` / `.ModalBackdrop` / `.ModalContent` | Modal system |
| `.Results` / `.ResultsCopyFeedback` | Win results panel |
| `.Histogram` / `.HistogramBar` | Stats chart |
| `.Countdown` | Next puzzle timer |
| `.EmojiRain` / `.EmojiRaindrop` | Win animation |
| `.jump` | Bounce animation on win |
| `.fade-in` | Modal entrance animation |
| `.hide` | Hidden modal |
| `.ErrorMessage` / `.error` | Invalid word feedback |

## Routes

| Path | Component | Description |
|------|-----------|-------------|
| `/` | `App` → `GameArea` | Main game |
| `/privacy` | `Privacy` | Privacy policy |
| `/support` | `Support` | Contact info |
| `/test` | `App` (test mode) | Debug: play any puzzle by index |

## Styling Notes

- Dark theme: background `#202020`
- Brown/poop color for highlighted letters
- Clean, minimal design
- Mobile-first with on-screen keyboard
- Smooth scroll to bottom as guesses are added
