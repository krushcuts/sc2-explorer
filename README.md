# SC2 Explorer
### Star Control II — The Ur-Quan Masters Exploration Log

A mobile-first Progressive Web App (PWA) for logging and tracking star systems, planets, and moons as you explore the galaxy in *Star Control II: The Ur-Quan Masters*. Designed to run from your iPhone home screen while you play.

---

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
- [Installing on iPhone](#installing-on-iphone)
- [Data Structure](#data-structure)
- [Reference Tables](#reference-tables)
- [Filters & Sorting](#filters--sorting)
- [GitHub Sync](#github-sync)
- [Data Backup & Restore](#data-backup--restore)
- [Revisit Logic](#revisit-logic)
- [Notes Tab](#notes-tab)
- [Star Map](#star-map)
- [Files in This Repo](#files-in-this-repo)
- [Technical Notes](#technical-notes)

---

## Features

### Exploration Logging
- Log **star systems** with name, coordinates (decimal precision, e.g. `290.8 : 26.9`), star color, and home faction/race
- Log **planets** per system with orbital position number (1 = closest to star)
- Log **moons** per planet with orbital position number (1 = closest to planet)
- Inline **moon entry** when adding a planet — log both at the same time

### Planet & Moon Data Fields
| Field | Details |
|---|---|
| Body type | 19 planet types (see [Reference Tables](#reference-tables)) |
| Position | Orbital order number |
| Weather | Hazard scale 0–8 |
| Tectonics | Hazard scale 0–8 |
| Temperature | Free-form degrees (supports negatives, e.g. `-400`) |
| Max resource | Resource quality 0–25 (see [Reference Tables](#reference-tables)) |
| Biological lifeforms | Yes / No |
| Alien homeworld | Yes / No — mark as a known alien home planet |
| Notes | Free-form text |

### Smart Revisit Detection
A body is flagged as **revisit-worthy** only if ALL of the following are true:
- Max resource value ≥ 5
- Weather + Tectonics combined ≤ 8 (safe enough to land)
- Temperature ≤ 2000°C (not too hot to collect)

### Filters
- **Revisit (C5+)** — systems containing at least one revisit-worthy body
- **Bio** — systems with confirmed biological lifeforms
- **Has Notes** — systems where any body (or the system itself) has text in the notes field
- **Treasure World** — systems containing a Treasure World planet
- **Homeworld** — systems containing a designated alien homeworld

### Sorting
- **A–Z** — alphabetical by star system name
- **Distance** — nearest to Sol first
- **Cargo** — highest max resource value first

### Header Stats (tappable)
The **bio** and **revisit** counts in the header are tappable shortcuts that toggle those filters directly.

### Star Map
- Visual dot map of all logged systems
- Sol marked at its correct coordinates (175.2, 145.0)
- Tap any dot to select and jump to that system in the list
- Y-axis inverted to match the in-game map orientation

### Notes Tab
A dedicated tab aggregating every note across all systems, planets, and moons in one searchable list. Each note card shows the source system and body, with keyword highlighting as you type.

### Dark / Light Mode
Toggle in the header. Preference is saved between sessions. Defaults to dark.

---

## Getting Started

### Prerequisites
- A free [GitHub account](https://github.com)
- Safari on iPhone (required for Add to Home Screen)

### Deployment

1. **Fork or upload** this repository to your GitHub account
2. Go to **Settings → Pages** in your repo
3. Set source to **main branch / root**
4. GitHub will provide a URL: `https://YOUR-USERNAME.github.io/sc2-explorer`
5. Open that URL in Safari on your iPhone

---

## Installing on iPhone

1. Open your GitHub Pages URL in **Safari** (must be Safari, not Chrome)
2. Tap the **Share** button (box with arrow pointing up)
3. Tap **Add to Home Screen**
4. The app appears as an icon on your home screen

> **Note:** If you update the app files in GitHub, you may need to hard-close and reopen the app for changes to load. The service worker is configured for network-first delivery of HTML so updates come through automatically when connected.

---

## Data Structure

Data is stored as JSON in the browser's `localStorage` on your device under the key `sc2-v2`. The top-level structure is:

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
          "type": "Telluric",
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

## Reference Tables

### Planet Types
| Type | Type | Type |
|---|---|---|
| Acid | Airless Rock | Alkali |
| Carbon | Gas Giant | Icy |
| Metal World | Noble World | Oolite World |
| Quasi-Degenerate World | Radioactive | Reducing |
| Shattered | Super Dense | Telluric |
| Treasure World | Water World | Yttric World |
| Unknown | | |

### Resource Values
| Value | Name | Notes |
|---|---|---|
| N/A | None | No minable resources |
| 1 | Common | Lowest value, rarely worth collecting |
| 2 | Corrosive | |
| 3 | Base Metal | |
| 4 | Noble Gas | |
| 5 | Rare Earth | Minimum revisit threshold |
| 6 | Precious Metal | High value ✦ |
| 8 | Radioactive | High value ✦ |
| 25 | Exotic | Rarest and most valuable ✦✦✦ |

### Star Colors
Yellow · Blue · Orange · Red · White · Brown Dwarf · Binary · Unknown

### Hazard Scale (Weather & Tectonics)
| Level | Meaning |
|---|---|
| 0 | None |
| 1–3 | Safe (teal) |
| 4–6 | Caution (amber) |
| 7–8 | Danger (red) |

> Bodies where **Weather + Tectonics > 8** are excluded from the revisit list regardless of resource value.

### Factions / Races
Humans · Spathi · Ilwrath · Pkunk · Yehat · Melnorme · Druuge · Mycon · Slylandro · Orz · Utwig · Supox · Syreen · VUX · Androsynth · Ur-Quan (Kzer-Za) · Ur-Quan (Kohr-Ah) · Thraddash · Zoq-Fot-Pik · Arilou · Chenjesu · Mmrnmhrm · Other

---

## Filters & Sorting

The filter bar sits between the search box and the system list. Multiple filters stack — a system must satisfy all active filters to appear.

| Filter | Activates when... |
|---|---|
| Revisit (C5+) | Any planet/moon has resource ≥ 5, combined hazard ≤ 8, and temp ≤ 2000° |
| Bio | Any planet/moon has biological lifeforms = Yes |
| Has Notes | System notes or any body notes contain text |
| Treasure World | Any planet has type = Treasure World |
| Homeworld | Any planet/moon has homeworld = Yes |

Sort options: **A–Z** · **Distance from Sol** · **Cargo (best first)**

---

## GitHub Sync

The app can push your exploration log directly to a JSON file in your GitHub repo, giving you a cloud backup that survives device changes.

### Setup
1. Go to [github.com](https://github.com) → your profile → **Settings**
2. **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. Click **Generate new token (classic)**
4. Name it `sc2-explorer`, set an expiration, check the **repo** scope
5. Copy the token (shown only once, starts with `ghp_`)
6. In the app, tap **⇅ GitHub** at the bottom
7. Enter your token, GitHub username, repository name, and file path (default: `sc2-log.json`)

### Push & Pull
- **↑ Push** — writes your entire exploration log to the file in your repo
- **↓ Pull** — reads the file from your repo and replaces local data

The last push timestamp appears as a green banner at the bottom of the main screen.

> **Security note:** Your GitHub token is stored only in your phone's `localStorage`. It is never transmitted anywhere except directly to the GitHub API. Tokens with only the `repo` scope can only access your own repositories.

---

## Data Backup & Restore

Even without GitHub sync, you can back up manually:

- **Export JSON** — downloads `sc2-log.json` to your device (save to iCloud Drive for safekeeping)
- **Import JSON** — restores from a previously exported file

> Recommended: export before every play session. Data lives in Safari's `localStorage` — clearing browser data will erase it.

---

## Revisit Logic

A planet or moon is considered **revisit-worthy** and counted in the header stat only when all three conditions are met:

```
cargoValue >= 5
AND (weather + tectonics) <= 8
AND temperature <= 2000 (or blank)
```

This prevents high-value but inaccessible worlds (extreme weather, molten surface) from cluttering your revisit list. The threshold is encoded in the `isRevisitable()` function in `index.html`.

---

## Notes Tab

Tap **Notes** in the tab bar to see every note you've written — across all systems, planets, and moons — in a single scrollable list.

- Each card shows the source system (tappable — jumps to that system in the Systems tab) and the body label
- The search bar filters in real time across note text, system names, and body labels
- Matching keywords are highlighted in amber

---

## Star Map

The star map is toggled open below the systems list. Sol is marked at coordinates **(175.2, 145.0)**.

**Distance calculations:**
- **Distance from Sol** — straight-line Euclidean distance in map units
- **Fuel units** — Distance ÷ 10 (shown next to distance in system cards)

Tapping a dot on the map selects that system and scrolls the list to it.

---

## Files in This Repo

| File | Purpose |
|---|---|
| `index.html` | The entire application — HTML, CSS, and JavaScript in one file |
| `manifest.json` | PWA manifest — app name, icons, display mode |
| `sw.js` | Service worker — network-first caching for offline support |
| `icon-192.png` | App icon (192×192) — Ilwrath Avenger |
| `icon-512.png` | App icon (512×512) — Ilwrath Avenger |
| `sc2-log.json` | Your exploration data (created by GitHub Sync — not committed manually) |
| `README.md` | This file |

---

## Technical Notes

- **Single-file app** — all logic lives in `index.html`. No build step, no npm, no dependencies.
- **Storage** — `localStorage` on-device. Keys: `sc2-v2` (data), `sc2-gh` (GitHub credentials), `sc2-theme` (dark/light preference).
- **Service worker** — HTML is fetched network-first so app updates propagate automatically. Static assets (icons, manifest) are cache-first.
- **Icon** — the Ilwrath Avenger sprite is embedded as a base64 data URI directly in `index.html`, so it loads instantly without an additional HTTP request.
- **iOS PWA** — uses `apple-mobile-web-app-capable` and `apple-touch-icon` meta tags for proper home screen integration.
- **No tracking, no ads, no external requests** — the only outbound network calls are to the GitHub API when you explicitly trigger a sync.
