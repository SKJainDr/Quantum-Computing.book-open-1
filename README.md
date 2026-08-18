# Quantum Computers — Online Reader

A self-contained GitHub Pages site for **Quantum Computers: Principles, Qubits, Circuits, Algorithms and Hardware** (Q.C. Series, Vol. I) by Dr. S. K. Jain.

## Latest fixes (this version)

- **Watermark made twice as visible**: font size doubled and opacity increased, tuned separately for light/dark theme (previously the same low opacity was used for both, so it was especially faint in dark mode) so it now reads clearly in both instead of being barely visible.
- **Visitor counter** added to the bottom of the sidebar (👁 icon, always visible).
- **Like button** added next to the read-aloud controls — a red heart with a live count. Click once to like; it turns solid red and the count increments. There's no "unlike" — see the note in Features below on why.
- Fixed a bug where the visitor counter and like button could silently fail to initialize if chapter content failed to load — they're now fully independent of each other.

## Earlier fixes

- **Fixed the root cause of missing images** (including the dedication-page photo): chapter content is loaded via JavaScript and inserted into `index.html`, so image paths needed to be `content/images/...`, not `images/...`, to resolve correctly once hosted. All ~104 images now load.
- **Typography now matches the book**: Georgia throughout (confirmed as the document's actual font), heading sizes/weights/colors pulled directly from the source's Heading 1–4 styles, body paragraphs justified (matching the book's own alignment).
- **Box colors now match the book exactly**: sampled directly from the real cell-shading fills used for each box type in the source document (Anecdote teal `#007B7F`, Key Concept purple `#4A148C`, Warning red-orange `#BF360C`, etc.), rather than an invented palette.
- **Light theme is now the default** — it's the book's actual appearance (white page, dark text). Dark mode is still available via the toggle. The watermark remains visible in both.
- **Chapter/section pager buttons** are now a solid colored gradient with white text, clearly visible.
- **Fixed a heading-hierarchy bug**: the source document inconsistently tags some major in-chapter sections (e.g. "3.1 Single-Qubit Gates") with the same style as chapter titles. These were rendering as duplicate giant chapter banners; they're now correctly demoted to section-level headings.

## What's inside

- `index.html` — the reader shell (sidebar TOC, topbar controls, reading pane, on-page TOC)
- `assets/css/style.css` — dark/light theme (CSS variables, toggle persists via `localStorage`)
- `assets/js/app.js` — chapter loading & routing, on-page TOC generation, read-aloud
- `content/*.md` — the book itself, one Markdown file per chapter, generated from your `.docx` source
- `content/manifest.json` — the chapter list that drives the sidebar (edit titles/order here)

## Features

- **Dark / light theme** — toggle in the top bar, remembers your choice
- **Read aloud** — uses the browser's built-in Web Speech API (no external service, works offline once loaded). Play/pause, stop, and a speed selector (0.8×–1.75×). The paragraph currently being read is highlighted and auto-scrolled into view, and it automatically advances to the next chapter when one finishes.
- **Clickable navigation** — every chapter link, every subsection in the on-page TOC, and Prev/Next chapter buttons are deep-linkable (`#/03-chapter-3.md#3-2-multi-qubit-gates`), so you can share a link straight to a section
- **Filter box** in the sidebar to quickly jump to a chapter
- **Visitor counter** (sidebar footer) and **like button** (top bar, red heart) — see "Visitor counter & like button" below for how these work and how to configure them
- Responsive: collapses to a slide-out sidebar on mobile

## Cross-linking the series

The sidebar has a "More in this series" section linking to the sibling volume — but since each book is a separate GitHub Pages site, this repo has no way to know the sibling's URL automatically. The link points to `#` (inert) until you fill it in.

**After you've deployed both volumes**, open `assets/js/app.js`, find the `SERIES_LINKS` constant near the top of the visitor-counter/like-button section, and replace the placeholder `url: "#"` with the sibling's real GitHub Pages URL, e.g.:

```js
const SERIES_LINKS = [
  { label: "Volume II — Quantum Algorithms & Complexity", url: "https://your-username.github.io/quantum-algorithms-book-site/" },
];
```

If you add more volumes to the series later, add more entries to this same array — each renders as its own link.

## Visitor counter & like button

Both are backed by [Abacus](https://abacus.jasoncameron.dev), a free counting API that needs no signup or API key — just a GET request. This was chosen deliberately: the older standard for this (CountAPI) shut down on August 7, 2026, so I verified Abacus was actually live before wiring it in rather than assuming.

- **Visitor counter**: increments once per page load (not once per chapter navigated within the app) and shows the running total in the sidebar footer.
- **Like button**: shows the current total like count on load. Clicking it increments the count once, turns the heart solid red, and remembers that you've liked it (via `localStorage`) so it can't be clicked repeatedly. There's no "unlike" — Abacus only allows anonymous increments, not decrements, without an admin key, so removing a like isn't possible without adding real authentication.

**Before you rely on these, check they actually work once deployed.** I built and tested this entirely in a sandboxed environment with no outbound internet access, so I could verify the code runs and fails gracefully (shows "—" if the request fails) but could **not** verify the live API calls actually succeed. Open your deployed GitHub Pages site, click the like button, and refresh — the count should persist and go up. If it doesn't, the free service may have changed or gone down (this happens with free API services, as the CountAPI shutdown above shows) — check `assets/js/app.js` for the `ABACUS_BASE` constant and swap in a replacement if needed.

**Namespace collisions**: both counters share the namespace `qc-series-vol1-skjain` (set near the top of `assets/js/app.js` as `COUNTER_NAMESPACE`). Abacus counters are "unlisted" but not private — anyone who knows the exact namespace/key string could read or increment them. This name is specific enough to be very unlikely to collide with someone else's counter, but if you want a guarantee, change `COUNTER_NAMESPACE` to something including your own domain or GitHub username before publishing.

## Content protection

The reader includes deterrents against casual downloading/copying, plus a visible watermark:

- Right-click, text selection, copy/cut, drag, and the usual save/print/devtools shortcuts (Ctrl/Cmd+C, S, P, U, Shift+I/J/C, F12) are blocked
- Printing shows a "not available for printing" message instead of the book
- A tiled watermark (`© Dr. S. K. Jain · <today's date>`) is rendered over every page, sized and shaded to be clearly legible in both themes, so it's captured in any screenshot or screen recording. The date updates automatically to the viewer's current date.

**Be aware of the actual limits of this:** these are deterrents, not real protection. Anyone who opens their browser's dev tools, disables JavaScript, or views page source can still read and copy the text — that's true of any website, and no client-side JavaScript can prevent it. This setup stops casual copy-paste, right-click saving, and printing, and it guarantees your name and a date are baked into any screenshot. It will not stop someone determined to extract the text. If you need real access control, that requires a login wall and server-side delivery, which is a different (and larger) project than a static GitHub Pages site — let me know if you want to go that route instead.

To change the watermark name, edit `WATERMARK_NAME` near the top of `assets/js/app.js`. To adjust its visibility further, edit `--watermark-opacity` (per theme) and the `font-size` in the `.watermark-layer` rules in `assets/css/style.css`.

## Publishing to GitHub Pages

1. Create a new GitHub repository (e.g. `quantum-computers-book`).
2. Copy everything in this folder into the repo root and push:
   ```bash
   git init
   git add .
   git commit -m "Quantum Computers — online reader"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```
3. In the repo on GitHub: **Settings → Pages → Source → Deploy from a branch → `main` / `(root)`** → Save.
4. Your book will be live at `https://<your-username>.github.io/<repo-name>/` within a minute or two.

### Testing locally before you push

Opening `index.html` directly by double-clicking it will **not** work — browsers block `fetch()` of local files for security reasons. Serve the folder instead:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## About the text conversion

This version was converted from your `.docx` source (not the PDF), which made a much more faithful conversion possible:

- **Figures** — all ~90 figures are extracted directly from the Word file and placed inline next to their original captions, compressed for fast web loading (13.4MB → 4.3MB) with no visible quality loss.
- **Equations** — the book's embedded equation objects (Equation Editor/MathType) are rendered as cropped, high-clarity images placed inline or inside dedicated "Equation" boxes, matching how they appear in the book itself.
- **Code blocks** — Qiskit/Python code samples are detected by their source formatting (monospace font) and rendered as proper fenced code blocks.
- **Pedagogical boxes** — the book's own six information-box types (📜 Anecdote, 🔑 Key Concept, 🌍 Real World, ⚠ Warning, 🧮 Mathematics, plus Equation/Example/Solved-Problem boxes) are detected from the document's table structure and styled distinctly, matching the design described in the book's own "How This Textbook Is Structured" section.
- **Data tables** (glossary/index, comparison charts, transpilation option tables, etc.) render as real Markdown tables.

This was built with an automated converter (Python + `python-docx`), not by hand-transcribing 387 pages, so it's worth a skim rather than treated as pixel-perfect — box classification is pattern-based and a rare box may be mis-typed (cosmetic only, content is preserved either way). If you spot anything off in a specific chapter, flag it and I can fix that section directly in the corresponding `content/NN-chapter-N.md` file.
