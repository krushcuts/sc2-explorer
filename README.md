# SC2 Explorer
### Star Control II — The Ur-Quan Masters Exploration Log

A mobile-first Progressive Web App (PWA) for logging and tracking star systems, planets, and moons as you explore the galaxy. Designed to run from your iPhone home screen while you play.

---

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
- [Tabs](#tabs)
- [Star Map](#star-map)
- [Filters & Sorting](#filters--sorting)
- [Visit Score](#visit-score)
- [Revisit Logic](#revisit-logic)
- [GitHub Sync](#github-sync)
- [Data Backup & Restore](#data-backup--restore)
- [Reference Tables](#reference-tables)
- [Data Structure](#data-structure)
- [Files in This Repo](#files-in-this-repo)
- [Technical Notes](#technical-notes)

---

## Features

### Exploration Logging
- Log **star systems** with name, coordinates (decimal precision, e.g. `290.8 : 26.9`), star color, and home faction/race
- Log **planets** per system with orbital position number (1 = closest to star)
- Log **moons** per planet with orbital position number (1 = closest to planet)
- Inline **moon entry** when adding a planet — log both at the same time
- Duplicate detection prevents entering the same star system or planet position twice

### Planet & Moon Data Fields
| Field | Details |
|---|---|
| Body type | 43 planet types (see [Reference Tables](#reference-tables)) |
| Position | Orbital order number |
| Weather | Hazard scale 0–8 |
| Tectonics | Hazard scale 0–8 |
| Temperature | Free-form degrees (supports negatives, e.g. `-400`) |
| Max resource | Resource quality — see [Reference Tables](#reference-tables) |
| Biological lifeforms | Yes / No |
| Alien homeworld | Yes / No |
| Notes | Free-form text |

### Visit Score
Every logged star system displays a **visit score (0–100)** next to its name. The score blends the best single-body score in the system (60%) with the system's aggregate mineral wealth (40%), so a system with four rich planets ranks higher than one with a single rich planet. Full formula details in the [Visit Score](#visit-score) section.

### Smart Revisit Detection
A body is flagged as **revisit-worthy** only if ALL of the following are true:
- Max resource value ≥ 5
- Weather + Tectonics combined < 8
- Temperature < 1000°C (or blank)

### Header Stats (tappable)
The **bio** and **revisit** counts in the header are tappable shortcuts that toggle those filters directly.

### Dark / Light Mode
Toggle in the header. Preference is saved between sessions. Defaults to dark.

---

## Getting Started

### What you need
- A free [GitHub account](https://github.com)
- Safari on iPhone (required for Add to Home Screen)

### Deploy to GitHub Pages

1. **Fork or upload** this repository to your GitHub account
2. Go to **Settings → Pages** in your repo
3. Set source to **main branch / root**
4. GitHub gives you a URL: `https://YOUR-USERNAME.github.io/sc2-explorer`

### Add to iPhone home screen

1. Open your GitHub Pages URL in **Safari** (must be Safari, not Chrome)
2. Tap the **Share** button (box with arrow pointing up)
3. Tap **Add to Home Screen**
4. The app icon appears on your home screen — tap it to launch full-screen

> **Updates:** After pushing new files to GitHub, hard-close and reopen the app. The service worker uses network-first delivery for HTML, so updates load automatically when connected.

> **Icon not refreshing?** Delete the home screen shortcut, clear Safari history and website data (back up your log first via GitHub Sync or Export JSON), then re-add to home screen.

---

## Tabs

The app has two main tabs — **Systems** and **Notes** — and a **References** dropdown at the top of every screen.

### Systems
The main exploration view. Search, filter, and sort all logged star systems. Each system shows its visit score next to the name. Expand any system to see its planets and moons with full data including individual body scores.

### Notes
Aggregates every note written across all systems, planets, and moons into a single searchable list. Each card shows the source system (tappable — jumps to that system) and body label. Keyword search highlights matches in amber.

### References (dropdown)
Tap **References ▾** to access three reference views:

**Planets Visited** — a flat sortable table of every planet and moon logged.

| Column | Description |
|---|---|
| Star System | System name — tap to jump to that system |
| Planet | Position number (e.g. `2` for planet 2, `2.1` for its first moon) |
| Type | Planet/moon type |
| Revisit | ✦ if revisit-worthy, blank otherwise |
| Max | Numeric resource value |
| Fuel Dist | Fuel units from Sol (distance ÷ 10) |

Tap any column header to sort; tap again to reverse.

**Calculator** — a standalone visit score calculator. Enter any combination of resource value, bio, weather, tectonics, temperature, and fuel distance to get a score and band recommendation, with a full breakdown of each contributing factor.

**Minerals** — a searchable reference of all 73 minerals with their category and RU value. Filter by mineral name or category.

---

## Star Map

Toggle the star map open below the systems list on the Systems tab. Sol is marked at coordinates **(175.2, 145.0)**.

- Tap any dot to select that system and scroll the list to it
- Y-axis is inverted to match the in-game map orientation

**Distance calculations:**
- **Distance from Sol** — straight-line Euclidean distance in map units
- **Fuel units** — Distance ÷ 10, shown in system cards and the Planets Visited table

---

## Filters & Sorting

Filters and sort options appear on two separate rows between the search bar and the system list. Multiple filters stack — a system must satisfy all active filters to appear.

| Filter | Activates when... |
|---|---|
| Revisit (C5+) | Any planet/moon is revisit-worthy (resource ≥ 5, combined hazard < 8, temp < 1000°) |
| Bio | Any planet/moon has biological lifeforms = Yes |
| Has Notes | System or any body has non-empty notes |
| Treasure World | Any planet has type = Treasure World |
| Homeworld | System has a home faction/race assigned |

Sort options: **A–Z** · **Distance from Sol** · **Cargo (best first)** · **Score (highest first)**

---

## Visit Score

Each star system displays a color-coded score (0–100) next to its name, e.g. **Alpha Giclas (64)**. The score is also used by the **Score** sort option.

### System score formula

```
System Score = best_body_score × 0.60 + aggregate_wealth × 0.40
```

**Best body score** — the highest individual planet/moon score in the system, accounting for fuel cost, hazards, and temperature (see below). This is the 60% component.

**Aggregate wealth** — the sum of all body mineral scores divided by a ceiling of 300, capped at 100. This component is fuel-independent (you're already paying to travel there). It rewards systems with multiple rich planets. Four Precious Metal bodies saturate this component.

### Individual body score formula

```
Score = mineral_score + bio_bonus − fuel_penalty − hazard_penalty − temp_penalty
```

| Component | Calculation |
|---|---|
| Mineral score | 0 (none) · 8 (Common) · 16 (Corrosive) · 25 (Base Metal) · 35 (Noble Gas) · 48 (Rare Earth) · 58 (Precious) · 68 (Radioactive) · 85 (Exotic) |
| Bio bonus | +15 if biological lifeforms present |
| Fuel penalty | fuel_distance × 0.5 |
| Hazard penalty | (weather + tectonics) × 4 |
| Temp penalty | 0 if blank or < 500° · 10 if 500–999° · 20 if 1000–1999° · 30 if ≥ 2000° |

### Score bands

| Score | Band |
|---|---|
| 70–100 | High Priority |
| 50–69 | Worth Visiting |
| 30–49 | Marginal |
| 10–29 | Low Priority |
| 0–9 | Skip |

---

## Revisit Logic

A planet or moon is flagged **revisit-worthy** (shown as ✦) only when all three conditions are met:

```
cargoValue >= 5
AND (weather + tectonics) < 8
AND temperature < 1000 (or blank)
```

A combined Weather + Tectonics score of **8 or higher** disqualifies a body — too unstable to land safely. A temperature of **1000°C or higher** also disqualifies — too hot to collect. The logic lives in the `isRevisitable()` function in `index.html`.

---

## GitHub Sync

Push your exploration log to a file in your GitHub repo for cloud backup that survives device changes.

### Setup
1. Go to [github.com](https://github.com) → your profile → **Settings**
2. **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. Click **Generate new token (classic)**
4. Name it `sc2-explorer`, set an expiration, check the **repo** scope
5. Copy the token — shown only once, starts with `ghp_`
6. In the app, tap **⇅ GitHub** at the bottom
7. Enter your token, GitHub username, repository name, and file path (default: `sc2-log.json`)

### Push & Pull
- **↑ Push** — writes your entire exploration log to the file in your repo
- **↓ Pull** — reads the file from your repo and replaces local data

The last push timestamp appears as a green banner at the bottom of the main screen. The GitHub modal shows last push and last pull times separately.

> **Security:** Your token is stored only in your phone's `localStorage` and sent only to the GitHub API. A `repo`-scoped token gives access only to your own repositories.

---

## Data Backup & Restore

For a manual backup without GitHub sync:

- **Export JSON** — downloads `sc2-log.json` to your device. Save to iCloud Drive for safekeeping.
- **Import JSON** — restores from any previously exported file.

> Export or push to GitHub before every play session. Data lives in Safari's `localStorage` — clearing browser data will erase it.

---

## Reference Tables

### Planet Types (43 total)
| | | |
|---|---|---|
| Unknown | Acid World | Airless Rock |
| Alkali World | Carbide World | Chlorine World |
| Chondrite World | Copper World | Crimson World |
| Dust World | Gas Giant | Green World |
| Halide World | Icy | Infrared World |
| Iodine World | Lanthanide World | Magma World |
| Magnetic World | Metal World | Noble World |
| Oolite World | Opalescent World | Organic World |
| Pellucid World | Plutonic World | Primordial World |
| Purple World | Quasi-Degenerate World | Radioactive World |
| Reducing | Redux World | Ruby World |
| Selenic World | Shattered | Super Dense World |
| Telluric World | Treasure World | Ultramarine World |
| Urea World | Water World | Xenolithic World |
| Yttric World | | |

*Unknown is the default type when logging a new planet.*

### Resource Values
| Value | Name | Notes |
|---|---|---|
| — | Unsurveyed | Not yet assessed |
| N/A | None | No minable resources |
| 1 | Common | |
| 2 | Corrosive | |
| 3 | Base Metal | |
| 4 | Noble Gas | |
| 5 | Rare Earth | Minimum revisit threshold |
| 6 | Precious Metal | ✦ |
| 8 | Radioactive | ✦ |
| 25 | Exotic | ✦✦✦ Rarest |

### Hazard Scale (Weather & Tectonics)
| Level | Indicator |
|---|---|
| 0 | None |
| 1–3 | Safe (teal pips) |
| 4–6 | Caution (amber pips) |
| 7 | Danger (red pips) |
| 8 | Danger — disqualifies from revisit |

### Star Colors
Yellow · Blue · Orange · Red · White · Brown Dwarf · Binary · Unknown

### Factions / Races
Androsynth · Arilou · Chenjesu · Druuge · Humans · Ilwrath · Melnorme · Mmrnmhrm · Mycon · Orz · Pkunk · Slylandro · Spathi · Supox · Syreen · Thraddash · Ur-Quan (Kohr-Ah) · Ur-Quan (Kzer-Za) · Utwig · VUX · Yehat · Zoq-Fot-Pik · Other

### Minerals (73 total)
Searchable in-app via **References → Minerals**. Categories and values:

| Category | RU Value | Example minerals |
|---|---|---|
| Common | 1 | Carbon, Hydrogen, Silicon, Chlorine, Hydrochloric Acid |
| Corrosive | 2 | Ammonia, Methane, Bromine, Iodine |
| Base Metal | 3 | Iron, Copper, Nickel, Lead, Titanium, Barium, Beryllium, Bismuth, Mercury + more |
| Noble Gas | 4 | Helium, Neon, Argon, Xenon |
| Rare Earth | 5 | Lanthanum, Neodymium, Holmium, Scandium, Yttrium |
| Precious Metal | 6 | Gold, Platinum, Palladium, Rhodium, Iridium |
| Radioactive | 8 | Uranium, Thorium, Plutonium, Radium |
| Exotic | 25 | Antimatter, Neutronium, Tzo Crystals, Quantum Crystals |

---

## Data Structure

Data is stored as JSON in the browser's `localStorage` under the key `sc2-v2`. Structure:

```json
{
  "systems": [
    {
      "id": "abc123",
      "name": "Vega",
      "x": 290.8,
      "y": 26.9,
      "starColor": "Yellow",
      "faction": "Spathi",
      "notes": "Quasispace portal nearby",
      "addedAt": "4/15/2026",
      "planets": [
        {
          "id": "def456",
          "order": 2,
          "type": "Telluric World",
          "weather": 3,
          "tectonics": 1,
          "temperature": "220",
          "cargoValue": 6,
          "bioLifeforms": true,
          "homeworld": false,
          "notes": "Dense vegetation, hostile fauna",
          "moons": [
            {
              "id": "ghi789",
              "order": 1,
              "type": "Airless Rock",
              "weather": 0,
              "tectonics": 2,
              "temperature": "-180",
              "cargoValue": 3,
              "bioLifeforms": false,
              "homeworld": false,
              "notes": ""
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Files in This Repo

| File | Purpose |
|---|---|
| `index.html` | The entire application — HTML, CSS, and JavaScript in one file |
| `manifest.json` | PWA manifest — app name, icons, display mode |
| `sw.js` | Service worker — network-first caching for offline support |
| `icon-192.png` | App icon (192×192) — Ilwrath Avenger |
| `icon-512.png` | App icon (512×512) — Ilwrath Avenger |
| `sc2-log.json` | Your exploration data (written by GitHub Sync — not committed manually) |
| `README.md` | This file |

---

## Technical Notes

- **Single-file app** — all logic lives in `index.html`. No build step, no npm, no dependencies.
- **Storage** — `localStorage` on-device. Keys: `sc2-v2` (data), `sc2-gh` (GitHub credentials), `sc2-theme` (dark/light preference).
- **Service worker** — HTML is fetched network-first so app updates propagate automatically when connected. Static assets (icons, manifest) are cache-first.
- **Icon** — the Ilwrath Avenger sprite is embedded as a base64 data URI in `index.html` — loads instantly with no extra HTTP request.
- **iOS PWA** — uses `apple-mobile-web-app-capable` and `apple-touch-icon` meta tags for proper home screen integration.
- **No tracking, no ads, no external requests** — the only outbound network calls are to the GitHub API when you explicitly trigger a sync.
