# Savanna static pages

Three Savanna learner pages, frozen from the **local dev app** into plain HTML + CSS so they can be
opened offline and used as a reference for front-end and accessibility work.

Open `index.html` (or any file in `pages/`) directly with `file://` — double-clicking works, no
server needed.

## What is here

| Page | Learner / course | What it shows |
|------|------------------|----------------|
| `pages/homepage-v1-paced.html` | Björn Sibanda — **WR 100: Fundamentals of Effective Communication**, cohort F26-SEM-16W (31 Aug – 20 Dec 2026), week 10 of 15 | Courses dropdown (3 courses), week strip with **every week preloaded**, "continue where you left off", immediate deliverables, period progress, overall score (59.56%, "slightly behind"), upcoming events |
| `pages/homepage-v1-self-paced.html` | Amara Nwosu — **VA-1: VA Foundations & Professional Identity** | Course-progression gauge (16% against the 70% graduation threshold), 1 of 3 modules complete, **working module switcher**, **preloaded programme modal**, pacing boost (11 days left), upcoming events |
| `pages/concept-with-navigation.html` | Amara Nwosu — concept *"The Future of the VA Industry"* | Curriculum navigation panel (read / current / unread states) beside a full authored concept: callouts, numbered card grid, two-column comparison, icon lists, "Mark as complete". The "Need a Fast Track?" block and the Padlet reflection block are removed on request. Includes an inline **AbolKit** `<abol-player>` lesson video. |
| `pages/abolkit-player-demo.html` | AbolKit integration demo | `@abolkit/player@0.1.1` via jsDelivr CDN. **Option 1 captions:** live generation against `http://16.16.124.190/api` on local HTTP; pre-generated VTT (`assets/media/hisys-story-*.vtt`, en/am/om/ti/aa) on GitHub Pages. |


## Layout

```
savanna_static_pages/
├── index.html                 landing page linking the three captures
├── pages/*.html               the frozen pages (relative links into ../assets)
├── assets/
   ├── css/                   the application's own stylesheets, unminified — application_alx.css 
   │                          (sprockets, themed), app.css (esbuild), Google Fonts, Typekit —
   │                          plus static-page.css, the only one written for these pages
   ├── fonts/                 76 font files, all referenced by the stylesheets above
   ├── img/                   51 images: logos, icons, concept artwork, substituted embeds
   └── js/                    popper.js + bootstrap.bundle.js (the exact files the live app
                              serves through sprockets) + static-page.js
```

Everything is shared: the three pages point at the same `assets/` tree, nothing is duplicated and
nothing is minified or gzipped. Captures are additive, so each run ends by dropping assets no
published page references any more — otherwise the Rive stills from retired week panels would
pile up run after run.

## What still works

These are live in the frozen pages, driven by `assets/js/static-page.js` against content that was
pre-rendered during the capture and parked in `<template>` elements (so nothing is duplicated in
the accessibility tree and no id appears twice):

| Control | Page | Behaviour |
|---|---|---|
| **Hamburger** | all | Collapses the sidebar to its 70px icon rail and back, remembered in `localStorage` under `sidebar-collapsed`. It moves the same three things the app does: `body.sidebar-collapsed`, `expanded` on `#sidebar`, and the labels (React wrote `display` inline on each, so `assets/css/static-page.css` overrides them from the body class). |
| **Week strip** | paced homepage | Every week of the course was rendered server-side at capture time; clicking one swaps the panel below it and moves the `active` state. The chevrons and the scroll-into-view of the selected week are reimplemented from the component's own sidecar JS. |
| **Score details modal** | paced homepage | Server-rendered per week, so it opens on its own — including the one that arrives with a week you switch to. |
| **My activities** | self-paced homepage | Each module's activity list was rendered at capture time; clicking a module swaps the panel beside it, as `HomepageSelfPacedActivities.tsx` does. |
| **Programme modal** | self-paced homepage | "Program details" normally fetches `/dashboards/program_structure.json` on first open — the capture opens it once so the fetched tree is baked in. Its course and module accordions expand and collapse: React renders only the body of an *open* section, so the capture opens each one to materialise its content, pairs every toggle with the element it controls (`data-static-collapse-toggle` / `data-static-collapse-body`) and closes again the ones the app had closed. |
| **Courses dropdown** | all | Server-rendered markup, so Bootstrap's dropdown opens it with no help: three courses across a programme group and "Other courses". |
| **Curriculum bar** | all | The bar fixed under the header pads the content by its own height and slides away on scroll down, as `CurrentCurriculumScrollBehavior.tsx` does. |
| Dropdowns, modals, accordions, collapse, tooltips | all | Bootstrap's own bundle. |

Everything else is inert.

## AbolKit player & captions (Option 1)

Integration uses the published CDN package `@abolkit/player@0.1.1` ([getting started](https://abolkit.vercel.app/getting-started/#player-cdn)).
The caption HTTP API is documented at [http://16.16.124.190/docs](http://16.16.124.190/docs) (routes under `/api`). The player attribute `caption-api` is the **API origin** (`http://16.16.124.190`); the component calls `/api/captions` and `/api/captions/generate` on that origin.

### Live caption generation (local HTTP)

```bash
python3 -m http.server 8000
# open http://localhost:8000/pages/abolkit-player-demo.html
```

On `http://localhost` / `http://127.0.0.1`, the demo sets:

- `caption-api="http://16.16.124.190"`
- `generate-captions="am om ti aa"` (English transcript via `lang="en"`, plus Amharic, Afaan Oromo, Tigrinya, Afar)

CORS on the API allows browser calls from localhost (`Access-Control-Allow-Origin: *`).

### GitHub Pages (HTTPS)

Live generation is **not** used on the deployed site. The API is HTTP-only; HTTPS pages cannot call it without mixed-content violations, and this project does not bypass that.

Pages uses pre-generated WebVTT files produced from the same video via the caption API and stored at:

- `assets/media/hisys-story-en.vtt`
- `assets/media/hisys-story-am.vtt`
- `assets/media/hisys-story-om.vtt`
- `assets/media/hisys-story-ti.vtt`
- `assets/media/hisys-story-aa.vtt`

**Live caption generation requires local HTTP testing until an HTTPS API endpoint is available.**


