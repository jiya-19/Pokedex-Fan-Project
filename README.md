# PokéDEX — Fan Project

> A fully functional, single-page Pokédex web app built with vanilla HTML, CSS, and JavaScript. No frameworks, no database, no build tools — just open the file and go.

![Fan Project](https://img.shields.io/badge/Fan%20Project-Not%20affiliated%20with%20Nintendo-red)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?logo=javascript&logoColor=black)
![PokéAPI](https://img.shields.io/badge/Data-PokéAPI-EF5350)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots](#screenshots)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [API Reference](#api-reference)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Legal Disclaimer](#legal-disclaimer)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This is a fan-made Pokédex that lets you browse, search, and filter Pokémon using live data from [PokéAPI](https://pokeapi.co). It covers all 9 generations (Pokémon #1 through #1025), includes Mega Evolutions, Gigantamax forms, and Primal Reversions, and presents everything in a polished single-page interface with smooth animations and a responsive layout.

The entire project lives in **one HTML file** — no npm install, no server, no build step.

---

## Features

### 🔍 Search
- Search by **exact name** (e.g. `pikachu`) or **Pokédex number** (e.g. `25`)
- **Partial name search** (e.g. `char` returns Charmander, Charmeleon, Charizard, and all Mega Charizard variants)
- Search results update in-place — no page reload

### 🎛️ Filters
- **Filter by Type** — all 18 types (Fire, Water, Grass, etc.)
- **Filter by Generation** — Gen I through Gen IX with region labels
- **Mega Evolutions filter** — dedicated category showing all Mega, Gigantamax, and Primal forms
- **Combined filters** — stack Type + Generation/Mega for precise results (e.g. Dragon-type Megas)

### ♾️ Infinite Scroll
- Home page displays 30 curated popular Pokémon
- Filter/search results load 24 cards at a time via `IntersectionObserver`
- Supports both numeric IDs (regular Pokémon) and slug strings (Mega forms)

### 📋 Pokémon Detail Modal
Clicking any card opens a **¾-screen modal** with blurred background containing:

| Tab | Contents |
|-----|----------|
| **Overview** | Flavor text, height, weight, catch rate, habitat, egg groups, abilities, weaknesses |
| **Base Stats** | Animated stat bars for all 6 stats with color coding and total |
| **Evolutions** | Full evolution chain with trigger labels (level, item, etc.) + Mega/Special forms section |
| **Moves** | Level-up moves and TM/HM moves |

- **✨ Shiny toggle** — swap the artwork to shiny variant instantly
- Click any Pokémon in the Evolution chain to navigate directly to their modal
- Press `Esc` or click outside the modal to close it

### 🎨 Themes
- **Dark mode** — deep space aesthetic with animated background orbs
- **Light mode** — clean, pastel-toned layout
- **System mode** — automatically follows your OS preference
- Theme preference is saved to `localStorage` and persists across sessions
- Theme toggle **hides** while a modal is open to avoid visual clutter

### 📱 Responsive Design
- Works on desktop, tablet, and mobile
- Cards reflow using CSS Grid with `auto-fill` columns
- Modal resizes to `96vw` on small screens

---

## Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Structure | HTML5 | Semantic single-file architecture |
| Styling | CSS3 + Custom Properties | Variables for theming, no preprocessor needed |
| Logic | Vanilla JavaScript (ES6+) | Fetch API, IntersectionObserver, no dependencies |
| Data | [PokéAPI](https://pokeapi.co) REST v2 | Free, comprehensive, no auth required |
| Fonts | Google Fonts (Nunito + Orbitron) | Loaded via CDN link tag |
| Hosting | _(see Deployment section)_ | Static file — any host works |

**Zero runtime dependencies. Zero npm packages. Zero build tools.**

---

## Getting Started

### Option 1 — Open Directly (Simplest)

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/pokedex-fan-project.git

# Open the file in your browser
open pokedex.html           # macOS
start pokedex.html          # Windows
xdg-open pokedex.html       # Linux
```

That's it. No server required for basic use.

### Option 2 — Serve Locally (Recommended for development)

Serving over HTTP avoids any potential browser restrictions on `file://` origins:

```bash
# Using Python (comes pre-installed on most systems)
python3 -m http.server 8080

# Using Node.js
npx serve .

# Using VS Code
# Install the "Live Server" extension and click "Go Live"
```

Then open `http://localhost:8080/pokedex.html` in your browser.

---

## Project Structure

```
pokedex-fan-project/
│
├── pokedex.html          # The entire application (HTML + CSS + JS)
├── README.md             # This file
└── LICENSE               # MIT License
```

Since the project is a single self-contained file, there are no `src/`, `dist/`, or `assets/` directories. All styles and scripts are inlined.

---

## How It Works

### Data Flow

```
User Input (search / filter)
        │
        ▼
  doSearch() builds an ID list
  (numeric IDs for regular Pokémon, slug strings for Megas)
        │
        ▼
  startDisplay(ids, label)
  resets the grid and infinite scroll state
        │
        ▼
  IntersectionObserver watches sentinel div
  → loadBatch() fetches 24 at a time from PokéAPI
  → makeCard() renders each result
        │
        ▼
  User clicks a card → openModal()
  → fetches species, evolution chain, moves in parallel
  → renders tabs on demand (lazy per-tab loading)
```

### Caching

All API responses are cached in a plain JavaScript object (`cache = {}`). This means:
- Repeat visits to the same Pokémon's modal are instant
- The full Pokémon list (`/pokemon?limit=10000`) is fetched at most once per session
- The Mega slug list is similarly fetched once and reused

### Mega Evolution Detection

Mega forms, Gigantamax forms, and Primal Reversions are detected by matching the slug pattern `-(mega|gmax|gigantamax|primal)` against the full Pokémon list from PokéAPI. Their IDs exceed 10,000 in the API, so they are stored and fetched by **name slug** rather than numeric ID throughout the codebase.

---

## API Reference

This project uses the following [PokéAPI](https://pokeapi.co/docs/v2) endpoints:

| Endpoint | Usage |
|---|---|
| `GET /pokemon/{id or name}` | Fetch Pokémon data (stats, types, moves, sprites) |
| `GET /pokemon?limit=10000` | Fetch full name+URL list for search and Mega detection |
| `GET /pokemon-species/{name}` | Fetch flavor text, genus, egg groups, habitat, varieties |
| `GET /evolution-chain/{id}` | Fetch full evolution chain tree |
| `GET /type/{name}` | Fetch all Pokémon belonging to a given type |

PokéAPI is free and open — no API key or authentication required. Please respect their [fair use guidelines](https://pokeapi.co/docs/v2#fairuse) — the in-app caching layer helps minimise redundant requests.

---

## Known Limitations

- **No offline support** — all data is fetched live from PokéAPI. An internet connection is required.
- **API rate limits** — very heavy usage in a short time could trigger PokéAPI's fair-use limits. The caching layer mitigates this significantly.
- **Alternate forms** — forms like regional variants (Alolan, Galarian, Hisuian), costume variants, and battle-only forms are not surfaced in the filter UI, though they exist in PokéAPI and can be searched by exact slug (e.g. `rattata-alola`).
- **Move details** — moves are listed by name only; damage, accuracy, and PP are not shown.
- **No persistence** — there is no favourites list, team builder, or user account system.

---

## Roadmap

Potential future improvements (contributions welcome):

- [ ] Favourite / bookmark Pokémon (saved to `localStorage`)
- [ ] Regional variant filter (Alolan, Galarian, Hisuian, Paldean)
- [ ] Move detail tooltips (power, accuracy, PP, type)
- [ ] Compare two Pokémon side-by-side
- [ ] Sound — play Pokémon cries using PokéAPI audio URLs
- [ ] PWA support (offline caching via Service Worker)
- [ ] Keyboard navigation for accessibility

---

## Legal Disclaimer

This is an **unofficial fan project** created for educational and personal purposes.

- Pokémon, Pokédex, and all related names, characters, and imagery are trademarks of **Nintendo**, **Game Freak**, and **The Pokémon Company**.
- Artwork and data are served via [PokéAPI](https://pokeapi.co), which aggregates publicly available Pokémon data under their open-use policy.
- This project is **not affiliated with, endorsed by, or sponsored by** Nintendo, Game Freak, or The Pokémon Company.
- This project is **non-commercial** — it generates no revenue and is not monetised in any form.
- No copyright infringement is intended.

---

## Contributing

Contributions, bug reports, and feature suggestions are welcome.

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes in `pokedex.html`
4. Test thoroughly in both light and dark mode, on desktop and mobile
5. Submit a pull request with a clear description of what changed and why

Please keep the single-file architecture intact — the goal is zero dependencies and zero build steps.

---

## License

This project is released under the [MIT License](LICENSE).

You are free to use, modify, and distribute this code for personal and educational purposes. Commercial use is discouraged out of respect for the Pokémon IP.

---
Built with ❤️ by a fan &nbsp;·&nbsp; Data from [PokéAPI](https://pokeapi.co) &nbsp;·&nbsp; Not affiliated with Nintendo

</div>
