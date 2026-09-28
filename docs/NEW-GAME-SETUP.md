# Setting up a new game, the Boundless Team way

The playbook for every game after the first. Written for whoever picks this up next, including
a fresh Claude session with no memory of how any of it was decided.

Read this before starting a new game. Most of it is decisions already made — following them
costs nothing, and re-deciding them tends to break something that was working.

---

## The shape of things

One studio, one website, many games:

```text
games.boundlessfiction.com/          Developer Website for EVERY game on Play
├── app-ads.txt                      ONE file, root only, every game, one AdMob account
├── privacy-policy/                  ONE policy, covers all games present and future
├── terms/                           ONE terms document, same
└── games/<slug>/                    one page per game
```

Three rules that follow from that, and that get broken by people trying to be helpful:

1. **The Developer Website on every Play listing is the root**, `https://games.boundlessfiction.com/` — never a game's own page. AdMob matches `app-ads.txt` against that domain.
2. **`app-ads.txt` exists once, at the root.** Never a per-game copy. Crawlers do not look in subfolders.
3. **The policy and terms are shared.** A new game does not get its own. If a new game does something the policy does not cover, extend the shared policy.

---

## Part 1 — The website

1. `cp -r games/game-2 games/<slug>` — short, lowercase, hyphenated. It becomes the URL and
   cannot be changed later without breaking links.
2. Art into `assets/img/<slug>/`: `icon-256.png`, `icon-96.png`, 4–6 screenshots as JPEG,
   and ideally `og.jpg` at 1200×630.
3. Fill in every `[Placeholder]` in the copied page, and update its `canonical` and `og:url`.
4. Add a card to the front page (`index.html`, the list under *Published games*) and a `<url>`
   block to `sitemap.xml`.
5. Push. Hostinger deploys from `main` automatically.

Detail lives in [games/README.md](../games/README.md).

---

## Part 2 — The game project itself

Conventions the first game (Selfbound) established. They are in its repository; copy them
rather than reinventing.

### Identity

- **Package name** `com.selfbound.game`-style: pick once, it is permanent after the first
  upload to Play. A new ID means a new listing with zero installs and zero reviews.
- Store the package id in one constant, and build the Play URL from it.

### Legal, in the app

- `GameConst.privacyPolicyUrl` → `https://games.boundlessfiction.com/privacy-policy/`
- `GameConst.termsUrl` → `https://games.boundlessfiction.com/terms/`
- `GameConst.legalVersion` → an int. Onboarding shows a **never pre-ticked** checkbox linking
  both; nothing starts until it is ticked. Bumping the version asks existing players once more,
  on a sheet they cannot dismiss.
- Settings links both documents too. Play expects the policy reachable in-app.

### Ads

- Eleven AdMob units per app (interstitial, app-open, native, eight rewarded). Test units by
  default; live units only behind `--dart-define=TEST_ADS=false`. Never tap your own live ads.
- Every rewarded unit: reward "1 item". The game decides what a video is worth, not AdMob.
- `app-ads.txt` needs nothing per game — one AdMob account, one publisher ID, already live.
- Publish the GDPR and US-state messages in AdMob, or the in-app consent flow does nothing.

### Purchases

- Product id `pack_<something>`, non-consumable, created in Play Console **after** a build with
  the billing library has been uploaded, then **activated**.
- Google Play is the source of truth for ownership; the local flag is only an offline cache.
  A refund must remove access on the next launch.
- Acknowledge every purchase, or Play refunds it automatically after three days.
- **Price at $0.99 for a first release.** Content may be worth more; an unknown developer's
  content is not. Raising later is allowed and keeps existing owners. Cutting later teaches
  people to wait for a sale.

### Layout

- Wrap the app above the Navigator in a max-600px centred column (`AppFrame` in Selfbound).
  One change makes every screen and sheet behave on tablets, including screens written later.

### Art pipeline

- One piece of key art → `python tool/generate_app_icon.py art.png` → launcher icons, the Play
  512, and the 1024×500 feature graphic. Android shows only the middle ~67% of an adaptive
  icon, so compose for that crop; the script already does.
- The themed (monochrome) layer must be a flat silhouette. A photograph cannot be one.

### Release build

- Upload key in `android/key.properties` (gitignored). **Back up the keystore** — losing it
  means Google must reset your upload key before you can ship an update.
- R8 on, resources shrunk, bundle split by language/density/ABI.
- `flutter build appbundle --release --dart-define=TEST_ADS=false --obfuscate --split-debug-info=build/symbols`,
  then upload the symbols to Play or crash reports are unreadable.

---

## Part 3 — Play Console, per game

| Field | Value |
|---|---|
| Developer Website | `https://games.boundlessfiction.com/` (the root, every time) |
| Privacy Policy | `https://games.boundlessfiction.com/privacy-policy/` |
| Target audience | 13–17 and 18+. "Children" means under 13; staying 13+ avoids Families policy |
| Restrict minor access | Off, unless the content truly requires it |
| Content rating | Answer the questionnaire honestly — you do not pick the rating |
| Data safety | No data collected by us; AdMob collects an advertising ID and device data; Play Games saves are the player's own |
| Ads declaration | Yes, contains ads |

**On ratings.** The questionnaire generates the rating; each authority applies its own rules to
your answers. Answering to reach a target rating is misrepresentation, and the penalty is
removal. A higher rating costs a little discovery (parental-control accounts, and minors in the
EEA/UK/AU/BR/SG/CH), and nothing else. Declare drug references, death, abuse and especially
**suicide or self-harm themes** if they are in there.

Title and description: **never state how much content there is.** "200 questions" ages into a
lie the moment you add more. Describe by kind.

---

## Part 4 — Order of operations

Two steps have waiting built into them, so start them first.

1. Play Console account, if new — a personal account opened after Nov 2023 must run a **closed
   test, 12 testers, 14 days** before production.
2. Payments profile. Bangladesh supports merchant registration; developer currency is USD, so
   payouts need a bank account that accepts an international USD transfer.
3. AdMob app and its eleven units; publish the consent messages.
4. Keystore, then a signed bundle to internal testing.
5. Create and activate the in-app product; test with a licence-tester account, installed
   **from Play** (sideloaded builds cannot buy).
6. Game page on this site; push; verify it is live.
7. Closed test → open testing (this is "early access": public listing, badge, and reviews that
   do not touch your public rating) → production.

---

## Part 5 — Verifying, not assuming

```bash
curl -I https://games.boundlessfiction.com/app-ads.txt     # 200 and text/plain, not HTML
curl -I https://games.boundlessfiction.com/games/<slug>/   # 200
```

Then, in a browser: the padlock, the page at phone width, and every link in the footer.

Hostinger notes that bite:

- The Git deploy directory **must be empty for the first deployment**, and Hostinger leaves a
  "Default page" placeholder in new subdomain folders. Delete it or the deploy refuses.
- The deploy directory must be the **subdomain's** document root. Check what hPanel reports
  when the subdomain is created.
- Issue SSL **for the subdomain**. A certificate for the apex does not always cover it.
- A `*.domain` certificate covers one level only: `games.boundlessfiction.com` yes,
  `a.b.boundlessfiction.com` no.

---

## Non-negotiables

- **No credential in any repository**, ever. This one is public. Tokens go in the OS credential
  store; a doc may say where they live, never what they are.
- **Test ads by default.** Live units only via an explicit build flag.
- **The path through a game is never behind an ad or a purchase.** Ads are opt-in trades;
  purchases add content and never remove friction that shouldn't exist.
- **Honest store listings**: honest rating, honest data safety, honest description. Every one of
  those is a removal risk, and removal costs more than any of them buy.
