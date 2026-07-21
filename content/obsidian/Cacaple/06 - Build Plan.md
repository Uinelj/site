---
tags: [ai-authored, projects, cacaple]
---

# Build Plan

## Phase 1 — Data Preparation 🗂️

### 1.1 French 4-Letter Word List
- [ ] Source a French dictionary (e.g. Lexique.org, Hunspell fr_FR, or DICO)
- [ ] Filter to exactly 4-letter words
- [ ] Clean: no proper nouns, no hyphens, handle accented characters
- [ ] **Decision:** Do we allow accented letters (É, È, Ê, Ù, etc.) or normalize to ASCII?
    - Option A: Normalize (É→E, etc.) — simpler keyboard, more like Wordle FR
    - Option B: Keep accents — more authentically French, but harder UX

### 1.2 Choose Target Words
- [ ] Primary candidate: **CACA** (💩 in French, 4 letters, no accents)
- [ ] Additional targets: **PIPI** 🚽 and **VOMI** 🤢 — see [[07 - Multi-Target Strategy]]
- [ ] Verify each target has a large connected word graph via BFS
- [ ] Determine rotation strategy (fixed cycle vs. curated per day)

### 1.3 Precompute BFS Distances
- [ ] Write a BFS script (Python or Node.js):
  1. Start from target word ("CACA")
  2. Find all words 1 letter different
  3. Expand layer by layer
  4. Record distance for every reachable word
- [ ] Output: `wordDist` dictionary (word → distance)
- [ ] Report: total reachable words, max distance, graph connectivity

### 1.4 Word Frequency Data
- [ ] Source French word frequencies (Lexique.org has `freqlivres` and `freqfilms2`)
- [ ] Map to our 4-letter word list
- [ ] Output: `wordFrequency` dictionary (word → frequency score)

### 1.5 Curate Start Words
- [ ] Filter words with distance 4–8 from CACA
- [ ] Prefer common/recognizable words
- [ ] Generate 1,000+ start words
- [ ] Output: `startWords` list with distances

---

## Phase 2 — Core Game Logic 🎮

### 2.1 Project Setup
- [ ] Init Vite + React (or vanilla JS) project
- [ ] Configure for static deployment

### 2.2 Game Engine
- [ ] `daysSinceEpoch()` — pick our own epoch date
- [ ] `getStartWord()` — from curated list
- [ ] `isValidWord(word, prev)` — dictionary + 1-letter-different check
- [ ] `getDist(word)` — lookup precomputed distance
- [ ] `getAdjacentWords(word)` — all valid 1-letter neighbors
- [ ] `buildTree()` / `getShortestPath()` — for "Yesterday" feature

### 2.3 State Management
- [ ] localStorage helpers (getGuesses, setGuesses, etc.)
- [ ] Game initialization (new day detection, streak management)

---

## Phase 3 — UI 🎨

### 3.1 Core Components
- [ ] `Header` — titre, numéro du puzzle, boutons modales
- [ ] `GameArea` — zone de jeu principale
- [ ] `Row` / `Box` — affichage des mots
- [ ] `Keyboard` — clavier AZERTY français
- [ ] `RowContainer` — conteneur scrollable

### 3.2 Modals
- [ ] `Intro` — Comment jouer
- [ ] `Stats` — Résultats, Totaux, Histogramme
- [ ] `Yesterday` — La solution d'hier
- [ ] `Countdown` — Compte à rebours

### 3.3 Animations
- [ ] 💩 Emoji rain on win
- [ ] Row bounce on win
- [ ] Error shake on invalid word
- [ ] Modal fade-in/out

### 3.4 Styling
- [ ] Dark theme (background #202020)
- [ ] Brown/poop highlight color
- [ ] Mobile-responsive
- [ ] AZERTY keyboard layout

---

## Phase 4 — Polish & Deploy 🚀

### 4.1 Features
- [ ] Share / copy results
- [ ] Streak tracking
- [ ] Yesterday's solution display
- [ ] PWA support (offline play)

### 4.2 Deployment
- [ ] Build static bundle
- [ ] Deploy to GitHub Pages / Cloudflare Pages / Vercel
- [ ] Custom domain (cacaple.fr?)

### 4.3 Localization
- [ ] All UI text in French
- [ ] French error messages ("Pas dans le dictionnaire", "Pas une lettre de différence")
- [ ] French share text

---

## Open Questions

1. **Accents** — Normalize or preserve? (impacts keyboard, word list, UX)
2. **Target word** — Multiple targets planned: CACA, PIPI, VOMI. See [[07 - Multi-Target Strategy]]
3. **Keyboard** — AZERTY layout with or without accent keys?
4. **Name** — "Cacaple"? "Crotte"? "Proutdle"? 💩
5. **Word list source** — Which French dictionary/corpus to use?
