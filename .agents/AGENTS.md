# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## What this is

A static, no-build Thai food recommendation site ("อร่อยไทยวันนี้"). Two plain HTML pages plus one shared vanilla-JS data file — no framework, no bundler, no package manager, no tests. Everything runs directly in the browser from `file://` or any static file server.

## Running it

There is no build/lint/test tooling. To view the site, open `index.html` directly in a browser, or serve the folder with any static server, e.g.:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000/index.html`.

## Architecture

- `dishes.js` — the single source of truth for content: the `DISHES` array (id, name, emoji, optional `image` path, description, category, ingredients) and the `CATEGORIES` list used for filter chips. Both `index.html` and `detail.html` load this file via `<script src="dishes.js">` and read the globals directly (no modules/imports).
- `index.html` — the main page. Contains its own `<style>` block and an IIFE `<script>` that owns all page state and rendering (no separate JS file). Key pieces:
  - **Dish of the Day**: picked once per calendar day (persisted in `localStorage` under `thaiFood.dishOfDay` as `{ date, dishId }`), re-picked automatically when the stored date is stale. The reroll button forces a re-pick that always excludes the currently-shown dish.
  - **Favorites**: an array of dish ids in `localStorage` under `thaiFood.favorites`. Toggled from both the gallery grid and the favorites chip list.
  - **Stale-data handling**: on load, both the stored Dish of the Day and stored favorites are filtered/validated against the current `DISHES` array (via `findDish`), so ids removed from `dishes.js` are dropped rather than causing broken UI.
  - Rendering is a manual `render()` that re-draws the spotlight, gallery, and favorites list by rebuilding `innerHTML`; there is no virtual DOM or diffing.
  - Search (`search-input`) and category chips filter the gallery client-side via `matchesFilter()`, matching against name/desc/id.
  - Clicking a dish card or the spotlight navigates to `detail.html?id=<dishId>`.
- `detail.html` — dish detail page. Reads `id` from the URL query string, looks it up in `DISHES`, and renders it standalone; also reads/writes the same `thaiFood.favorites` localStorage key so favorite state stays in sync with `index.html`.
- `image/` — dish photos referenced by a dish's `image` field in `dishes.js`; dishes without an `image` fall back to their `emoji` field.

## Domain language

See `CONTEXT.md` for the canonical terms and their exact meaning — use these terms (not the "avoid" synonyms listed there) in code, comments, and commit messages:

- **Dish** — one entry in `DISHES`, identified by `id`.
- **Dish of the Day** — the single dish spotlighted for today; stable for the calendar day, changed only by reroll (which never repicks the displaced dish) or by rollover to a new day.
- **Favorites** — the visitor's set of liked dishes, independent of Dish of the Day.

## Editing the menu

When adding/removing dishes in `dishes.js`, keep in mind both pages read `DISHES` by array reference at load time with no reactivity — changes require a page reload. Also check that any `localStorage`-stored dish ids removed from the array are still handled by the existing stale-data filtering in `index.html` (don't reintroduce unguarded lookups).
