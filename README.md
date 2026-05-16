# YIC Lost & Found Portal — CS382 Upgrade

**CS382 – Object Oriented Web Development**  
Yanbu Industrial College | Computer Science & Engineering Department  
Academic Year 1447–1448 / 2025–2026 | Second Semester

> This document covers all changes and additions made to upgrade the project
> from CS381 to meet CS382 requirements. The original CS381 README is in `README.md`.

---

## What Changed from CS381 to CS382

CS381 produced a working Lost & Found web application using vanilla PHP, CSS, and
plain JavaScript. CS382 takes the same application and upgrades it in three key areas
required by the new course specification:

| Requirement | CS381 | CS382 |
|---|---|---|
| JavaScript | Vanilla ES6 (no library) | **jQuery 3.7** |
| Async communication | None — full page reloads only | **AJAX via `$.getJSON`** |
| Styling discipline | Inline `style=""` and `<style>` blocks allowed | **External CSS only** |
| Database | MySQL (PDO) | MySQL (PDO) — unchanged |
| Authentication & roles | Student / Admin | Student / Admin — unchanged |

---

## Updated Tech Stack

| Layer | CS381 | CS382 |
|---|---|---|
| Frontend JS | Vanilla ES6 | **jQuery 3.7.1** (CDN) |
| Async | — | **AJAX (`$.getJSON`)** |
| CSS | External + scattered inline | **External only** (`css/style.css`) |
| Backend | PHP 8.0+ / PDO | PHP 8.0+ / PDO — unchanged |
| Database | MySQL / MariaDB | MySQL / MariaDB — unchanged |
| Icons | Font Awesome 6 | Font Awesome 6 — unchanged |
| Fonts | Google Fonts – Inter | Google Fonts – Inter — unchanged |

---

## CS382 Requirement Checklist

- [x] HTML controls on every page (forms, inputs, buttons, navigation)
- [x] All styling in external CSS — zero inline `style=""` attributes on key pages, no internal `<style>` blocks
- [x] Dynamic interface behavior implemented with **jQuery**
- [x] At least one **AJAX** feature (live item search)
- [x] Persistent data storage — MySQL database
- [x] Authentication & authorization with 2 roles (Student, Admin)

---

## Change 1 — jQuery (Dynamic Interface)

### What was replaced

`js/main.js` was completely rewritten from vanilla JavaScript to jQuery.
Every feature that was written using raw browser APIs (`document.getElementById`,
`addEventListener`, etc.) now uses jQuery equivalents (`$()`, `.on()`, `.find()`, etc.).

### Features now powered by jQuery

| Feature | How jQuery is used |
|---|---|
| Mobile navigation toggle | `.toggleClass('open')`, `.attr()` |
| Dark mode toggle | `$('html').attr('data-theme', ...)`, `localStorage` |
| Flash message auto-dismiss | `.fadeOut()` with callback |
| Flash close button | `$(document).on('click', '.flash-close', ...)` |
| Password show/hide | `.prev('input').attr('type', ...)` |
| Lost/Found type toggle | `.addClass()`, `.removeClass()`, `.prop('checked', ...)` |
| Image upload preview | `FileReader` triggered via jQuery `.on('change', ...)` |
| Form validation | `.on('submit', ...)`, `.find()`, `.val()`, `.css()` |
| Confirm dialogs | `$(document).on('click', '[data-confirm]', ...)` |
| Smooth scroll | `$('html, body').animate({ scrollTop: ... })` |
| Textarea character counter | `.each()`, `.on('input', ...)`, `.text()` |
| **AJAX live search** | `$.getJSON()`, `.done()`, `.fail()` |

---

## Change 2 — AJAX Feature (Live Item Search)

### What it does

On the **Browse Items** page, results update in real time as the user types or
changes any filter — without the page reloading.

### How it works (concept)

```
User types in search box
        ↓
jQuery waits 350ms (debounce — avoids firing on every keystroke)
        ↓
$.getJSON() sends a background request to ajax/search.php
        ↓
ajax/search.php queries the database and returns JSON
        ↓
jQuery receives the JSON and re-renders the results grid
        ↓
User sees updated cards — page never reloaded
```

The same AJAX call is also triggered immediately when any dropdown
(Type, Category, Status, Sort) changes.

### New file: `ajax/search.php`

| Property | Detail |
|---|---|
| Method | GET |
| Parameters | `search`, `type`, `category`, `status`, `sort` |
| Response | `Content-Type: application/json` |
| Response shape | `{ "total": int, "items": [ { id, title, type, status, ... } ] }` |

### Elements added to `items.php`

| Element | ID / Purpose |
|---|---|
| Search text input | `id="liveSearch"` — jQuery listens here |
| Results grid container | `id="resultsGrid"` — jQuery replaces its content |
| Item count label | `id="resultsCount"` — jQuery updates the number |

### Difference visible to the user

**CS381:** Type a search term → click Search → whole page reloads → results appear.  
**CS382:** Type a search term → results appear instantly, no reload, no browser flash.

You can prove this is AJAX by opening **Browser DevTools → Network tab** while
typing in the search box. You will see individual requests firing to `ajax/search.php`
with JSON responses — with no full document reload.

---

## Change 3 — External CSS Only

### The problem in CS381

Styles were scattered in three places:
- Inside HTML tags as `style=""` attributes (inline styles)
- Inside `<style>` blocks within PHP files (internal styles)
- In the external `css/style.css` file

### What CS382 enforces

All styling must be in `css/style.css`. Inline and internal styles are avoided
because they violate the principle of **Separation of Concerns** — HTML defines
structure, CSS defines appearance, JavaScript defines behavior. Mixing them makes
code harder to maintain, reuse, and debug.

### New CSS classes added to `style.css`

| Class | Replaces |
|---|---|
| `.container--padded` | `style="padding-top:28px;padding-bottom:48px"` |
| `.card-badges` | `style="display:flex;align-items:center;gap:8px"` |
| `.card-footer-meta` | `style="font-size:.78rem;color:var(--text-muted)"` |
| `.item-meta-list` | `style="display:flex;flex-direction:column;gap:4px..."` |
| `.stat-num--danger/success/accent` | `style="color:var(--danger/success/accent)"` |
| `.how-card`, `.how-icon`, `.how-title`, `.how-text` | "How It Works" section styles |
| `.user-avatar`, `.user-name-cell`, `.user-name-you` | Admin user table styles |
| `.role-badge`, `.role-admin`, `.role-student` | User role badge styles |
| `.promote-btn`, `.demote-btn` | Admin role action button styles |
| `.search-loading`, `.ajax-empty` | AJAX state indicators |

### Internal `<style>` block removed

`admin/users.php` had a `<style>` block embedded inside the PHP file.
This was removed entirely and those rules were moved to `css/style.css`.

---

## Additional Features (Built During CS382 Phase)

These features were added beyond the minimum CS382 requirements as improvements
to the application.

### Dark Mode Toggle

A button in the navbar lets users switch between light and dark themes.
- The preference is saved in `localStorage` so it persists across page loads.
- A tiny script in `<head>` reads localStorage and applies the theme before the
  page renders, preventing a white flash on reload.
- The Royal Commission color palette (navy, purple, cyan, gold) was applied to
  both light and dark themes.

### Admin Role Promotion / Demotion

An admin can promote any student to Admin, or remove admin privileges from
another admin, directly from the **Manage Users** page.

- Clicking "Make Admin" or "Remove Admin" shows a confirmation dialog (via jQuery).
- The action is sent as a POST form with a CSRF token.
- The promoted/demoted user receives an in-app notification automatically.
- Safety guards: cannot change your own role, cannot demote the last admin.

### Royal Commission Theme

The visual design was updated to reflect the Royal Commission for Jubail & Yanbu
color identity:

| Color | Source | Usage |
|---|---|---|
| Deep navy `#0A1A6E` | RC map shape | Primary color, navbar |
| Purple `#7B2FBE` | RC purple facet | Accent, hover states |
| Green `#16A34A` | RC green facet | Success states |
| Royal blue `#1D4ED8` | RC blue facet | Gradients |
| Cyan `#0BBADF` | RC cyan facet | Accent stripe, gradients |
| Gold `#C9A56A` | RC palm tree emblem | Accent buttons, highlights |

---

## New & Modified Files

### New Files (CS382)

```
lost-found/
└── ajax/
    └── search.php          ← AJAX endpoint (returns JSON search results)
```

### Modified Files (CS382)

```
lost-found/
├── css/
│   └── style.css           ← New utility classes, dark mode, RC theme
├── js/
│   └── main.js             ← Fully rewritten in jQuery 3.7
├── includes/
│   ├── header.php          ← jQuery CDN added, dark mode toggle button
│   └── footer.php          ← Updated course label to CS382
├── items.php               ← IDs for AJAX, inline styles removed
├── index.php               ← Inline styles replaced with CSS classes
└── admin/
    └── users.php           ← Internal <style> block removed, promote/demote feature
```

---

## How to Verify Each CS382 Requirement

### Verify jQuery
Open `js/main.js` — every function uses `$()`, `.on()`, `.find()`, `.val()` etc.
No `document.getElementById` or `addEventListener` anywhere.

### Verify AJAX
1. Open the site in a browser and go to **Browse Items**
2. Open **DevTools → Network tab** (F12)
3. Type anything in the search box
4. Watch requests appear to `ajax/search.php` with JSON responses
5. The page URL never changes, the browser never reloads

### Verify External CSS Only
1. Right-click any page → **View Page Source**
2. Search (`Ctrl+F`) for `style=` — none found on cleaned pages
3. Search for `<style>` — no internal style blocks
4. All styling references `css/style.css`

---

## Login Credentials (Demo — same database)

| Role | Email | Password |
|---|---|---|
| Admin | admin@yic.edu.sa | Password123 |
| Student | ahmed@yic.edu.sa | Password123 |
| Student | sara@yic.edu.sa | Password123 |

---

## AI Disclosure (CS382)

The CS382 upgrade was developed with assistance from Claude Code (Anthropic). AI was used to:
- Migrate vanilla JavaScript logic to jQuery equivalents
- Design and implement the AJAX live search feature
- Refactor inline styles into external CSS classes
- Implement the admin role promotion/demotion feature
- Apply the Royal Commission visual theme
- Write this documentation

All generated code was reviewed, tested, and understood by the student group before submission.
