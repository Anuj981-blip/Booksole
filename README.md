# BOOKSOLE

A single-file, browser-based reader and game-console shell for "Choose Your Own Adventure"–style branching gamebooks, themed as a retro game cartridge system. Opens with a pixel-art cinematic intro, boots into a console-style menu, and lets you import, play, save, and track achievements across multiple branching-fiction books.

Includes a bundled demo book, **"The BOOKSOLE,"** an original horror-comedy story that also serves as the framing story for the intro cutscene.

---

## What this actually is

BOOKSOLE is one HTML file (`booksole_v04-2.html`) with no build step and no server. Open it in a browser and it runs entirely client-side, using `localStorage` for persistence. It's built to feel like a game console: a boot screen, a "cartridge library" of books, achievements, and a save-slot system, wrapped around what is functionally an EPUB/gamebook reader.

The intent is to make reading branching fiction (à la classic Goosebumps-style "Give Yourself Goosebumps" gamebooks) feel like playing a retro game rather than reading a flat document.

## Features

**Cinematic pixel-art intro.** A seven-scene cutscene tells the in-universe origin story before the console boots: a jealous neighbor builds a device that traps people inside a horror author's unfinished stories, and the player's avatar gets sucked in. Click-to-advance dialogue, parallax scenes, glow/lighting effects, and an animated cast of original characters and creatures. Skippable at any time via the `SKIP INTRO` button.

**Boot sequence.** A faux system-boot animation (progress bar, status messages) before reaching the library, in keeping with the console conceit.

**Cartridge library.** A grid view of every book you've imported, each shown as a "cartridge" with cover art (if available), author, series badge, and a completion percentage based on endings found. New books get a `NEW` badge.

**Book importer.** Supports two input formats:
- **EPUB** — unpacks the archive, parses the OPF manifest and spine, extracts chapter HTML, detects "PAGE n" style links to build the choice graph, and pulls cover art if present.
- **Plain text / HTML** — looks for numbered page markers and "turn to page n" / "go to page n" phrasing to reconstruct the same page-and-choice structure from unstructured text.

Both formats produce the same internal book structure, so the rest of the app doesn't need to know which format a book came from.

**Branching reader.** Renders the current page's text and choices, tracks your path through the book, and lets you jump back to any earlier point in your run by clicking it in the path trail. Endings are auto-classified as good / bad / neutral based on keyword heuristics in the page text (for imported books) or set explicitly (for the bundled demo).

**Multiple save slots per book.** Start as many parallel runs through a book as you like, switch between them, delete the ones you don't want, and pick up exactly where you left off — including which page you're on and your full choice path so far.

**Achievements.** Nine built-in achievement types evaluated per-book (first ending found, three different endings, all endings, a "good" ending, all "bad" endings, a sub-3-choice speedrun, five completed playthroughs, ten distinct pages visited, one journal note written). Unlocking one shows a toast notification.

**Journal.** A separate notes view for jotting observations about a book as you play.

**Search.** In-reader text search across the current book's pages.

**Import / export saves.** Export your save state, achievements, and stats to a JSON file; re-import it later or on another device.

## Getting started

1. Download `booksole_v04-2.html`.
2. Open it directly in a browser (double-click, or `File → Open`). No server, no install, no dependencies to set up locally.
3. Watch the intro (or click `SKIP INTRO` in the bottom-right corner).
4. Click anywhere on the boot screen once it reaches 100%.
5. The bundled demo book, **"The BOOKSOLE (Demo),"** loads automatically the first time — click it to start reading.
6. To add your own book, click the `+ ADD BOOK` tile in the library and choose an `.epub`, `.txt`, or `.html` file.

There's nothing to install. The only external dependency loaded at runtime is [JSZip](https://stuk.github.io/jszip/) (via CDN) for unpacking EPUB archives, and two Google Fonts (`Press Start 2P`, `Courier Prime`) for the retro look — both loaded over the network, so an internet connection is needed for EPUB import and for fonts to render exactly as designed, but the app itself doesn't require a backend.

## Bringing your own books

### EPUB

Works best with EPUBs that already follow a "Choose Your Own Adventure" convention of page-numbered chapter files and in-text links phrased like "Turn to PAGE 42." The parser:

- Reads the `.opf` manifest and spine to get reading order and metadata (title, author, publisher, year, description).
- Matches chapter filenames against a `chNN.html`/`chNN.xhtml` pattern to assign page numbers.
- Scans each chapter for links containing "PAGE n" to build the choice graph, using the surrounding paragraph text as the choice label.
- Treats any page with zero detected choices as an ending, and guesses its tone (good/bad/neutral) from a short keyword list in the page text.
- Pulls cover art from the manifest's declared cover image, if one exists.

If your EPUB doesn't use this exact convention, it may import with missing choices or no detected pages at all — there's no manual page-mapping UI yet.

### Plain text / HTML

A looser fallback for books without proper EPUB structure. It looks for standalone numbers on their own line as page markers, then scans each page's text for phrases like "turn to page," "go to page," "see page," or "flip to page" to build choices automatically. Needs at least three detected page markers to accept the file; otherwise it reports failure and returns you to the library.

### Duplicate handling

If you re-import a file whose title matches a book already in your library, the importer updates that existing entry in place (refreshing cover art and page data) rather than creating a duplicate cartridge.

## The bundled demo: "The BOOKSOLE"

A short original horror-comedy gamebook, twelve pages, three endings (one good, one bad, one neutral/bittersweet). It doubles as the textual counterpart to the intro cutscene: the intro shows Max getting pulled into the BOOKSOLE device; the demo book is what happens once he's inside, told as a branching story you navigate by making choices, the same way any imported gamebook plays.

It loads automatically on first run and never overwrites a same-titled import, so it's safe to leave in your library permanently as a quick-start example of the expected page/choice format if you're curious how the engine represents a book internally.

## Data and privacy

Everything is stored in your browser's `localStorage` — your library list, each book's page data, your save slots, journal notes, found endings, and unlocked achievements. Nothing is sent to a server; nothing leaves your machine except the initial EPUB-parsing libraries and fonts loaded from their respective CDNs.

Because `localStorage` has a per-origin size limit, very large libraries (many big EPUBs with embedded cover art) can eventually hit that ceiling — if you see a "STORAGE FULL" notice, exporting and trimming old saves is the workaround until a more scalable storage layer exists.

Clearing your browser's site data for the page you're running this from will erase your library and saves. Use the export feature first if you want a backup.

## Project structure (single file)

Everything lives in `booksole_v04-2.html`, organized roughly top-to-bottom as:

| Section | What it does |
|---|---|
| `<style>` block | All visual styling for the boot/library/reader/journal screens, retro CRT scanline overlay |
| Intro IIFE (`<script>` near the top of `<body>`) | The seven-scene pixel-art cutscene: sprite-drawing functions, scene functions, the scene runner/click-to-advance loop |
| `bootStart()` | The faux boot progress animation |
| `DEMO` | The bundled demo book's page/choice data |
| Library functions (`libLoad`, `libSave`, `renderLibrary`, etc.) | Reading and writing the cartridge list and per-book metadata |
| Save/session functions (`newSave`, `makeChoice`, `goBack`, `jumpPath`, `persist`, `loadSession`) | Per-book save slots, choice tracking, path history |
| `buildAchs`, `checkAchs` | Achievement definitions and unlock checking |
| `parseEpub`, `parseTxt`, `handleFile` | The book import pipeline |
| Render functions (`renderStory`, `renderSaves`, `renderAchs`, `renderStats`, `renderDots`, `renderJournal`) | DOM updates for the reader UI |

There is no bundler, no package.json, and no build artifacts — the file you download is the file that runs.

## Known limitations

- The EPUB and plain-text parsers are heuristic, not a full CYOA-format spec implementation; books that deviate from the expected page/link conventions may import incompletely.
- No manual choice-graph editor — if the importer gets a page or link wrong, there's currently no in-app way to fix it short of re-importing a cleaned-up source file.
- `localStorage` is the only persistence layer, with the size and single-browser-profile limitations that implies.
- The intro cutscene's pacing is fixed (click-to-advance dialogue, auto-advance between scenes after a set duration); there's no scrub bar or chapter-select for it beyond the skip button.

## Credits

Original concept, demo story, and pixel-art intro built for this project. Goosebumps-style branching gamebooks are the genre inspiration; 
