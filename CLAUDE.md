# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, build-less personal portfolio/CV for Mateo de la Cruz Méndez (Full Stack .NET Software Engineer). The site is bilingual (Spanish default / English) and consists of exactly two self-contained HTML files. There is no package manager, no build step, no tests, and no server — each file embeds all of its CSS and JavaScript inline.

- `index.html` — the portfolio landing page (hero, about, stack, experience timeline, projects, contact).
- `cv.html` — a print-optimized résumé (A4 layout, "Download PDF" triggers `window.print()`).
- `CV2.pdf`, `Mateo_PerfilStaffing.pdf`, `CV. Mateo De la Cruz (1).doc` — source/reference CV documents, not used by the site.

## Running / previewing

Just open the file in a browser — there is nothing to build or serve:

```powershell
Start-Process .\index.html
```

The only external dependency is **i18next** loaded from a CDN (`cdn.jsdelivr.net`). Both files degrade gracefully offline: if `i18next` fails to load, the Spanish content already present in the HTML is shown and the experience list still renders (see the `boot()` guard).

## Architecture (shared across both files)

Both files follow the same self-contained pattern, so changes usually need to be mirrored in both:

1. **`CONFIG` object** (top of the `<script>`) is the single source of truth for dynamic data. Edit only this to update real content:
   - `hireDate` / `careerStart` — ISO dates. `yearsSince()` computes tenure and total years of experience **live at page load**, so the "12+ years" / "5 years" figures are never hardcoded.
   - `jobs[]` — experience entries. The entry with `current:true` uses `since` to auto-compute its duration and renders the "Present" label; all others use fixed `dateES`/`dateEN` strings. Each job carries bilingual fields (`roleES`/`roleEN`, `descES`/`descEN`, etc.). Note the tech field differs by file: `index.html` uses `tech: [...]` (array → chips), `cv.html` uses `techES`/`techEN` (string).

2. **i18next translations** (`resources.es` / `resources.en`) drive all static copy. Markup carries `data-i18n="key"` attributes; `updateDOM()` walks every `[data-i18n]` node and sets `innerHTML` from the matching key. Interpolation vars (`{{total}}`, `{{tenure}}`, `{{year}}`) are passed in `updateDOM()`. `escapeValue:false` is set because many strings contain intentional `<b>` markup.

3. **Render + language toggle**: `renderExperience()` (index) / `renderJobs()` (cv) build the timeline HTML from `CONFIG.jobs`. The language button toggles `lang`, persists it to `localStorage` under key `lang`, and calls `i18next.changeLanguage(..., updateDOM)`. Language is shared by key but **not** synced between the two pages beyond `localStorage`.

## Editing conventions

- **Every user-visible string must exist in both `resources.es` and `resources.en`** with the same key, and the element must have a matching `data-i18n`. Adding text without a translation key breaks the language toggle for that element.
- When updating experience/skills, change `CONFIG` and the translation strings — do **not** hardcode computed years or duplicate job data into the markup.
- Keep content edits synchronized between `index.html` and `cv.html` where they describe the same facts (jobs, education, certs, contact info, stack).
- Contact numbers/links appear in multiple places (hero meta, floating buttons, contact cards). WhatsApp links use a `data-wa` attribute whose number is combined with the translated `wa_msg` in `updateDOM()`; update `data-wa`, not the `href`'s query string.
