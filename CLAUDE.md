# CLAUDE.md

> Guidelines for AI assistants working on this repository.

## Project Overview

**Taichung Family Trip 2026** — a static single-page website for a three-day, two-night family vacation to Taichung (2026/2/20-22). The trip covers sea-line tourism with 2 adults and 2 children (ages 4 and 7).

- **Live site:** https://taichung-family-trip.netlify.app
- **Repository:** https://github.com/sniperHK/taichung-family-trip

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Pure HTML5, CSS3, vanilla JavaScript (no frameworks, no bundler) |
| Fonts | Google Fonts — Noto Sans TC, Noto Serif TC, DM Serif Display |
| Client storage | `localStorage` (packing checklist persistence, key: `taichung-trip-packing`) |
| Maps | Google Maps Directions (external links) |
| Hosting | Netlify (auto-deploy on push to `main`) |
| Secrets | `age` encryption for environment variables |

## Repository Structure

```
.
├── index.html            # Main website (all HTML, CSS, JS in one file)
├── 行程.md               # Itinerary — SINGLE SOURCE OF TRUTH
├── 交通路線.md            # 12 driving routes with distances, times, parking
├── .shared-ai-context.md  # Project context, changelog, git history sync
├── netlify.toml           # Netlify config (publish dir: /)
├── .env.example           # Environment variable template
├── .env.age               # age-encrypted environment variables (committed)
├── .gitignore             # Ignores plaintext .env, age-key.txt
├── .claude/
│   └── setup.sh           # Cloud VM startup: auto-decrypts .env.age
├── secrets/
│   └── env.keys           # Environment variable allowlist
└── messageImage_*.jpg     # Trip reference image
```

## Critical Rule: Data Sync

**`行程.md` is the single source of truth for all itinerary data.**

Any content change must follow this workflow:

1. Update `行程.md` first
2. Sync the change into `index.html` (timeline cards, tables, details)
3. If transportation is affected, update `交通路線.md` as well
4. Commit all changed files together
5. Push to `main` triggers Netlify auto-deploy

Never update `index.html` itinerary content without updating `行程.md` first.

## index.html Architecture

The entire site lives in a single `index.html` file (~2000 lines) with embedded CSS and JS.

### CSS (~900 lines)

- Ocean vacation theme with CSS custom properties for colors
- Key color tokens: `--deep-navy`, `--ocean-blue`, `--sunset-coral`, `--warm-sand`, `--seafoam`
- Responsive design using `clamp()` for fluid typography, Flexbox, and CSS Grid
- Mobile-first with media queries for larger screens

### HTML Sections (in order)

1. **Navigation** — sticky nav bar with backdrop blur, scroll-based section highlighting
2. **Hero** — full-viewport animated scene (clouds, sun, gradient sky)
3. **Hotel Info** — accommodation details for the hotel
4. **Day 1 Timeline** — arrival + Gaomei Wetland sunset
5. **Day 2 Timeline** — Ocean Life Park, fishing port, cycling, night market
6. **Day 3 Timeline** — Natural Science Museum, suncake DIY, return trip
7. **Transportation** — route planner widget with Google Maps integration
8. **Practical Info** — weather, packing tips, emergency contacts
9. **Packing Checklist** — interactive checkboxes with localStorage persistence

### JavaScript (~140 lines)

- Scroll-based navigation highlighting
- `IntersectionObserver` for scroll-reveal animations
- Spot info toggle (expandable location cards with `data-*` attributes)
- Packing list with `localStorage` read/write
- Route planner: start/end selector, swap button, Google Maps link builder

## Development Workflow

### No build step required

This is a static site — edit files directly and open `index.html` in a browser. There is no `package.json`, no linter, no test suite, and no build tool.

### Deployment

- **Automatic:** Push to `main` on GitHub triggers Netlify deploy
- **Fallback:** Use `NETLIFY_BUILD_HOOK_URL` (in `.env.age`) to trigger a deploy via webhook when no GitHub PAT is available

### Secrets Management

Environment variables are encrypted with `age`:

- **Public key:** `age1sd7r54gpapjuvt67x4g4ednmzus7s5xqx93ltcv57axcrs6w7yyqktrceh`
- **Encrypted file:** `.env.age` (committed to git)
- **Decryption:** `.claude/setup.sh` runs automatically on Cloud VM startup, requires `AGE_SECRET_KEY` env var
- **Never commit** plaintext `.env` or `age-key.txt`

### Branching

- `main` — production branch, triggers Netlify deploy
- `master` — legacy default branch (not used for deploy)
- Feature branches follow the pattern `claude/<description>-<id>`

## Commit Conventions

Use conventional commit prefixes:

- `feat:` — new features or content additions
- `fix:` — bug fixes or content corrections
- `docs:` — documentation-only changes
- `chore:` — config, tooling, secrets management

Commit messages may be in English or Chinese depending on the change context. Keep messages concise.

## Code Conventions

- **File names:** Chinese for content files (`行程.md`, `交通路線.md`), English for config and code files
- **CSS:** Use existing CSS custom properties; keep the ocean theme consistent
- **HTML:** Semantic elements, proper `target="_blank"` with `rel="noopener"` on external links
- **JS:** Vanilla only — no external libraries or frameworks
- **IDs/Classes:** English, kebab-case (e.g., `route-planner`, `packing-list`)
- **Data attributes:** `data-key` for localStorage mapping on checklist items

## Updating .shared-ai-context.md

After completing work, update `.shared-ai-context.md`:

1. Run `git log --oneline --graph -15` and paste output into the Git History section
2. Add a changelog entry in the format: `[YYYY-MM-DD] [Agent Name]: Description of changes`
3. Update any sections that are no longer accurate

## Key Content Details

- **Hotel:** Janda Golden Tulip Hotel (Wuqi district, Taichung)
- **Trip dates:** Feb 20-22, 2026 (Fri-Sun)
- **Travelers:** 2 adults + 2 children (ages 4 and 7)
- **Focus area:** Taichung sea-line (Wuqi, Gaomei, Qingshui)
- **Ocean Park tickets:** Pre-purchased (Order #PN5727310576) — 1 adult + 2 child = NT$1,300, Session 2 (10:00-11:00)

## What NOT to Do

- Do not add npm/yarn dependencies or a build system
- Do not split `index.html` into multiple files (the single-file approach is intentional)
- Do not change the visual theme away from the ocean aesthetic
- Do not remove localStorage persistence from the packing checklist
- Do not commit plaintext `.env` or private keys
- Do not push directly to `main` without verifying content sync between `行程.md` and `index.html`
