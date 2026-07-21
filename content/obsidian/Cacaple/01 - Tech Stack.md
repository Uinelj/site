---
tags: [ai-authored, projects, cacaple]
---

# Tech Stack

## Frontend Framework
- **React 19.1.1** — Single Page Application
- **Vite** — Build tool (evidenced by hashed asset filenames like `index-BNI_6U_Q.js`)
- **React Router** — Client-side routing (HashRouter / createBrowserRouter)

## Analytics
- **Mixpanel** — Event tracking (game start, finish, copy results, etc.)
- **Plausible** — Privacy-friendly web analytics (`plausible.io/js/script.js`)

## Hosting
- **Fully static** — No backend API calls for game logic
- All game data (word list, distances, frequencies) is **embedded in the JS bundle**
- The site could be hosted on any static host (Netlify, Vercel, GitHub Pages, Cloudflare Pages)

## State Management
- **React `useState` / `useCallback`** — No external state library
- **`localStorage`** — Persistence of game state, streaks, stats

## Other Libraries Found in Bundle
- **PostCSS** — CSS processing (in the build pipeline, not runtime)
- **PDF.js** — Appears to be an artifact/unused dependency (react-pdf-viewer references found)

## For Our Clone
We can simplify:
- Vite + React (or even **vanilla JS/HTML/CSS** since the logic is simple)
- Drop Mixpanel (or add a privacy-friendly alternative like Plausible)
- Host on GitHub Pages or Cloudflare Pages for free
