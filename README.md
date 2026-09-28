# games.boundlessfiction.com

The developer website for **Boundless Team**, and the home of the privacy policy, terms and
`app-ads.txt` that every one of our games points at.

Plain HTML and one stylesheet. No build step, no framework, no dependencies: what is in this
repository is exactly what gets served.

```text
games-site/
├── index.html              the landing page — this URL is the Developer Website on Play
├── app-ads.txt             ONE file for every game (see "app-ads.txt" below)
├── robots.txt
├── sitemap.xml
├── favicon.ico
├── assets/
│   ├── css/site.css        the whole design, one file
│   └── img/
│       ├── site/           studio icon, social image
│       └── selfbound/      per-game icon and screenshots
├── privacy-policy/
│   └── index.html          one policy, covers all games
├── terms/
│   └── index.html          one terms document, covers all games
└── games/
    ├── selfbound/index.html
    ├── game-2/index.html   the template to copy for the next game
    └── README.md           how to add a game
```

| URL | File |
|---|---|
| `https://games.boundlessfiction.com/` | `index.html` |
| `https://games.boundlessfiction.com/app-ads.txt` | `app-ads.txt` |
| `https://games.boundlessfiction.com/privacy-policy/` | `privacy-policy/index.html` |
| `https://games.boundlessfiction.com/terms/` | `terms/index.html` |
| `https://games.boundlessfiction.com/games/selfbound/` | `games/selfbound/index.html` |

---

## Before this goes live

Search the repository for `[` and fill in what is left. As it stands:

- **`games/selfbound/index.html`** — replace `[Google Play URL]` once the listing exists, and
  remove the `aria-disabled="true"` and the "Coming to Google Play" line.

`app-ads.txt` is already complete, with the real publisher ID in it.
- **`games/game-2/`** — leave it alone. It is the template, and every `[Placeholder]` in it is
  meant to stay until it is copied.

The contact address throughout is `bsoumik103@gmail.com`. To change it, find and replace
across the repository — it appears in the pages, the footers and the JSON-LD.

---

Two documents worth reading before touching anything:

- [docs/NEW-GAME-SETUP.md](docs/NEW-GAME-SETUP.md) — the playbook for adding a game, from this
  site through Play Console and AdMob.
- [docs/ACCESS-AND-DEPLOY.md](docs/ACCESS-AND-DEPLOY.md) — repository access, credentials, and
  how the site reaches the web.

## Working on it

### Clone

```bash
git clone YOUR_REPOSITORY_URL
cd games-site
```

### Open in VS Code

```bash
code .
```

Or File → Open Folder → `games-site`. Two extensions make this pleasant, neither required:

- **Live Server** — right-click `index.html` → *Open with Live Server*. It serves the folder
  on `http://127.0.0.1:5500`, which is the only way to test the absolute paths (`/assets/…`,
  `/privacy-policy/`) properly. Opening the file directly with `file://` makes every one of
  those links break, and that is the file protocol's fault, not the site's.
- **axe Accessibility Linter** — catches missing alt text and contrast problems as you type.

Any static server works if you would rather not install anything:

```bash
python -m http.server 5500
```

Then open `http://localhost:5500/`.

### What to change, and where

| You want to change | Edit |
|---|---|
| Studio name, intro, contact | `index.html` |
| The games list on the front page | `index.html`, the `<ul class="cards">` block |
| A game's text, features, Play link | `games/<game>/index.html` |
| Game icons and screenshots | `assets/img/<game>/`, then the `<img>` tags on that page |
| Colours, spacing, typography | `assets/css/site.css` — the `:root` variables at the top |
| Privacy policy | `privacy-policy/index.html` (and update the date at the top) |
| Terms | `terms/index.html` (and update the date at the top) |
| AdMob publisher ID | `app-ads.txt` |
| Adding a whole new game | `games/README.md` walks through it |

When you change either legal document in a way that materially affects players, also bump
`GameConst.legalVersion` in each game's own source, so the games ask people to accept the new
version.

### Push

```bash
git add .
git commit -m "Update game website"
git push
```

**GitHub is source control and history. Hostinger is production hosting.** Pushing to GitHub
does not by itself change the live site unless you have set up the Git deployment below.

---

## Hostinger

> hPanel's layout shifts between plans and redesigns. The names below are what the menus are
> called at the time of writing; if something has moved, the feature is still there under a
> similar name.

### 1. Create the subdomain

hPanel → your hosting plan → **Domains → Subdomains**. Create `games` under
`boundlessfiction.com`, which gives `games.boundlessfiction.com`.

Hostinger creates a folder for it and shows you the path — usually something like
`public_html/games` or `domains/games.boundlessfiction.com/public_html`. **Write that path
down.** It is the document root, and every step below depends on knowing it exactly.

If the subdomain is on a different hosting account from the main domain, add a DNS `A` record
for `games` pointing at that account's IP, in **Domains → DNS Zone**.

### 2. Put the files in the document root

The contents of this repository go *in* that folder, not in a `games-site` folder inside it.
When it is right, `index.html` sits directly in the document root:

```text
<document root>/index.html
<document root>/app-ads.txt
<document root>/assets/css/site.css
<document root>/privacy-policy/index.html
<document root>/terms/index.html
<document root>/games/selfbound/index.html
```

Three ways to get it there, best first.

**a. Git deployment (recommended).** hPanel → **Advanced → Git** → *Connect with GitHub*,
authorise the Hostinger GitHub app, pick this repository, pick the branch (`main`), and set
the deploy directory to the document root path from step 1. Then every `git push` deploys
automatically: GitHub notifies Hostinger, Hostinger pulls, the files are replaced. There is no
build step, which is exactly what a static site wants.

Two things that trip people up: **the deploy directory must be empty for the first
deployment**, and the deploy directory must be the subdomain's document root, not
`public_html`, or the site will appear on the wrong domain.

**b. File Manager.** hPanel → **Files → File Manager**, open the document root, and upload a
zip of the repository contents, then extract it there. Fine for the first upload and for the
occasional one-file fix. Do not forget the dot-less files: `app-ads.txt` and `robots.txt` must
land in the root.

**c. FTP/SFTP.** hPanel → **Files → FTP Accounts** for the credentials, then FileZilla or the
VS Code SFTP extension. Upload the repository contents into the document root. If you use an
extension that stores credentials in the project, keep that file out of Git — `.gitignore`
already excludes `sftp.json` and `.ftp-config`.

### 3. HTTPS

hPanel → **Security → SSL**. Issue a free Let's Encrypt certificate for
`games.boundlessfiction.com` and turn on **Force HTTPS**. A new certificate can take up to an
hour. Google Play will not accept an `http://` privacy policy URL, and ad-network crawlers
expect `https://` too.

### 4. Check it works

Open every one of these and confirm what should appear does:

```text
https://games.boundlessfiction.com/                     the landing page
https://games.boundlessfiction.com/app-ads.txt          plain text, starting with a # comment
https://games.boundlessfiction.com/privacy-policy/      the policy
https://games.boundlessfiction.com/terms/               the terms
https://games.boundlessfiction.com/games/selfbound/     the game page
```

Then check, in a browser:

- the padlock is present on each one, with no "mixed content" warning;
- `app-ads.txt` renders as **text**, not as a download and not as HTML;
- the pages look right at phone width (F12 → device toolbar);
- images load — a broken icon usually means a path missing its leading `/`.

From a terminal, the quickest proof:

```bash
curl -I https://games.boundlessfiction.com/app-ads.txt
```

`HTTP/2 200` and `content-type: text/plain` is what you want.

---

## app-ads.txt

**One file, at the root, for every game.** There is one AdMob account and one publisher ID, so
there is one `app-ads.txt`:

```text
https://games.boundlessfiction.com/app-ads.txt
```

Never create `games/selfbound/app-ads.txt` or any other per-game copy. The crawlers do not
look there. Each game's Play listing has
`https://games.boundlessfiction.com/` as its **Developer Website**, and the crawler takes that
domain and fetches `/app-ads.txt` from its root. A file anywhere else is invisible, and the
AdMob console will keep telling you it cannot find one.

This matters for money, not tidiness: without a valid `app-ads.txt`, a good share of
programmatic ad demand will not bid on your inventory.

After it is live, AdMob can take a day or so to re-crawl, and the warning in the console can
take a few days to clear. That is normal. Verify it yourself with the URL above rather than
waiting on the console.

---

## Play Console

For **every** game, the Developer Website is the root:

```text
https://games.boundlessfiction.com/
```

Not the game's own page. The root is what `app-ads.txt` is matched against, and it is what
makes the listing look like it belongs to a studio rather than to one app. Individual game
pages exist for players, for links in the store description, and for search.

The privacy policy URL on every listing is `https://games.boundlessfiction.com/privacy-policy/`.

---

## House rules

- **Common things stay common.** The policy, the terms and `app-ads.txt` exist once. A game
  page links to them; it never carries a copy.
- **One stylesheet.** New styling goes in `site.css` where the next game can reuse it, not in
  a `<style>` block on one page.
- **Absolute paths** (`/assets/css/site.css`) so a page can be moved between folder depths
  without rewriting every link.
- **Images carry `width` and `height`** so the page does not jump while they load, and real
  `alt` text so screen readers get something useful.
- **Keep it light.** The whole site is well under a megabyte. Screenshots are the only thing
  likely to change that: export them as JPEG, not PNG.
