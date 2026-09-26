# Adding a game

Every game gets one folder here and one card on the front page. Nothing else changes.

```text
games/
├── selfbound/      a published game
├── game-2/         the template — copy this, never edit it in place
└── README.md       this file
```

## The steps

1. **Copy the template.** `games/game-2/` → `games/your-game/`. Use a short, lowercase,
   hyphenated folder name: it becomes the URL,
   `https://games.boundlessfiction.com/games/your-game/`. Pick it once — changing it later
   breaks any link anyone has shared.

2. **Add the art.** Make `assets/img/your-game/` and put in it:
   - `icon-256.png` and `icon-96.png` — the app icon, square.
   - 4–6 screenshots as `.jpg`, straight from a phone. Name them for what they show
     (`the-path.jpg`), not `screenshot-1.jpg`.
   - `og.jpg` at 1200×630 for link previews, if you have one. Otherwise leave the page
     pointing at `/assets/img/site/og.jpg`.

3. **Fill in the page.** Open the copied `index.html` and work top to bottom, replacing every
   `[Placeholder]`:

   | Placeholder | What goes there |
   |---|---|
   | `[Game Name]` | the title, in `<title>`, `<h1>`, the metadata and the JSON-LD |
   | `[Game Description]` | one or two plain sentences, reused in the meta description |
   | `[Genre]` | Narrative, Puzzle, Arcade… |
   | `[Google Play URL]` | `https://play.google.com/store/apps/details?id=your.package.name` |
   | `[Feature]` | four short things the game actually does |

   Also update the two `https://games.boundlessfiction.com/games/game-2/` URLs in the
   `<link rel="canonical">` and `og:url` to your folder, and point the `<img>` tags at your
   art. When the game goes live, remove the `aria-disabled="true"` from the Play button and
   delete the "In development" line.

4. **Add the card.** In the site root `index.html`, find the list under *Published games* and
   copy the marked card. Drop `class="soon"` once it is published.

5. **Add it to `sitemap.xml`** — one `<url>` block, copied from the one above it.

## What never goes in here

- **`app-ads.txt`** lives only at the site root. One AdMob account means one file, and the
  crawlers only read the root of the developer website.
- **Privacy policy and terms** live at `/privacy-policy/` and `/terms/` and cover every game.
  Game pages link to them; they never carry their own copy.
- **`site.css`** is shared. Style a game page with the existing classes; if something is
  genuinely new, add the class to the shared stylesheet so the next game can use it too.

## Before you push

Open the page from `index.html` in a browser and check, in this order: the title in the tab,
every link, every image, the page at phone width, and that the Privacy and Terms links in the
footer still work from this folder depth (they are absolute paths, so they should).
