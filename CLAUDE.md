# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**식단기록앱** — A single-file, offline-capable Korean meal tracking web app. No server, no frameworks, no external CDN. Everything lives in one file: `index_eat.html`.

## Running / Opening the App

```
# Open directly in browser (no server needed)
start index_eat.html

# Or serve locally if needed
python -m http.server 8080
# then open http://localhost:8080/index_eat.html
```

## Architecture

`index_eat.html` is the only deliverable — all HTML, CSS, and JavaScript inline in one file (target: ≤500KB).

### JavaScript Structure (inside `<script>`)

```
FOOD_DATABASE[]          → 35 built-in Korean foods, { name, calories } (kcal/100g)
state{}                  → { currentDate: "YYYY-MM-DD", entries: [] }
localStorage utilities   → loadFromStorage(date), saveToStorage(date, entries)
date utilities           → getTodayString(), formatDateKorean(), addDays()
calorie logic            → calcCalories(amount, per100g), calcDailyTotal(entries)
autocomplete             → searchFoods(query) → max 8 partial matches
render functions         → renderDateNavigator(), renderFoodList(), renderCalorySummary(), renderAll()
event handlers           → handleAddEntry(), handleDeleteEntry(id), handleDateChange(±1), handleGoToToday(), handleFoodNameInput()
init()                   → called on DOMContentLoaded
```

### Data Model (localStorage)

- Key: `diet_YYYY-MM-DD`
- Value: `{ date, entries: [{ id, name, amount, caloriesPer100g, totalCalories, createdAt }] }`
- ID format: `entry_${Date.now()}_${padded 3-digit random}`
- Calorie formula: `Math.round(amount × caloriesPer100g / 100)`

### Color System (CSS Custom Properties)

| Variable | Value | Use |
|---|---|---|
| `--color-primary` | `#FFD700` | Header, main buttons |
| `--color-primary-dark` | `#FFA500` | Button hover |
| `--color-secondary` | `#4CAF50` | Add button, calorie summary |
| `--color-secondary-dark` | `#2E7D32` | Secondary hover |
| `--color-danger` | `#F44336` | Delete buttons |
| `--color-bg` | `#FFFDE7` | Page background |

## Critical Constraints

- **No external dependencies** — no CDN, no npm, no frameworks. Must work fully offline.
- **XSS prevention** — always use `textContent` to insert user data into the DOM, never `innerHTML`.
- **localStorage error handling** — wrap all reads in try/catch (JSON.parse can throw); catch `QuotaExceededError` on writes.
- **Korean UI** — all user-facing text must be in Korean.
- **Single file** — do not split into separate CSS/JS files.

## Key UI Sections

```
<header>          → app title (yellow background)
.date-navigator   → ← prev | date display | next → | 오늘 button
.input-card       → food name (with autocomplete), amount(g), kcal/100g, live preview, 추가하기 button
.summary-card     → daily total calories (green highlight)
.food-list        → per-entry: name / amount / kcal/100g / total kcal / delete button
```

## Validation Rules

- Food name: required, 1–50 chars, trim whitespace
- Amount: required, number, 1–9999 (g)
- kcal/100g: required, number, 0–999

## Responsive Breakpoints

- Mobile: ≤767px (single column, full width)
- Tablet: 768–1023px (max-width 600px, centered)
- Desktop: ≥1024px (max-width 800px, centered)
