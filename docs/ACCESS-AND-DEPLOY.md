# Access and deployment

Where this project lives, who can push to it, and how it reaches the web. Written down so it
survives a new laptop or a six-month gap.

**No credential is written in this file, and none should ever be.** Anything committed to Git
ends up on GitHub, and GitHub's secret scanning revokes tokens it finds there — usually within
minutes. Where each secret actually lives is below.

---

## The repository

| | |
|---|---|
| GitHub account | `Soumik766` |
| Repository | [`Soumik766/games`](https://github.com/Soumik766/games) (private) |
| Default branch | `main` |
| Local working copy | `D:\games-site` |

```bash
git clone https://github.com/Soumik766/games.git games-site
cd games-site
```

## How pushing is authenticated

The GitHub token is in **Windows Credential Manager**, under host `github.com`, saved through
Git Credential Manager (which ships with Git for Windows). It is encrypted by Windows for your
user account.

```bash
git push        # just works; the helper supplies the credential
```

To see what is stored, change it, or remove it:

- **Windows:** Start → *Credential Manager* → *Windows Credentials* → entries beginning
  `git:https://github.com`.
- **From a terminal:**
  ```bash
  printf "protocol=https\nhost=github.com\n\n" | git credential fill      # show
  printf "protocol=https\nhost=github.com\n\n" | git credential reject    # delete
  ```
  `fill` prints the token in clear text, so do not run it while sharing a screen.

A backup copy of the token is kept on the development machine, outside every repository and
outside anything Git tracks, so it can be recovered if Credential Manager is lost in a
reinstall. A password manager is a better home for it, and generating a fresh token takes a
minute either way.

### Replacing the token

Tokens expire, and any token that has been pasted into a chat, an email or a screenshot should
be replaced.

1. [github.com/settings/tokens](https://github.com/settings/tokens) → revoke the old one.
2. Generate a new fine-grained token: repository access limited to `Soumik766/games`,
   permission **Contents: Read and write**. Nothing else is needed to push.
3. Store it:
   ```bash
   printf "protocol=https\nhost=github.com\nusername=x-access-token\npassword=NEW_TOKEN\n\n" | git credential approve
   ```
4. Update the backup copy on your machine, if you keep one.

---

## How the site reaches the web

**GitHub stores history. Hostinger serves the site.** Pushing to GitHub does not by itself
change what visitors see, unless the Hostinger Git deployment below is connected.

### Current state, last checked 26 September 2026

| Host | Status |
|---|---|
| `boundlessfiction.com` | Live on Hostinger (LiteSpeed). Serves *Boundless Fiction — Bangla Manga, Comics & Novels*. |
| `www.boundlessfiction.com` | CNAME to the root. |
| `games.boundlessfiction.com` | **Does not exist yet** — no DNS record, and there is no wildcard. |

Nothing about this domain touches GitHub Pages. The subdomain has to be created in Hostinger
before any of it resolves.

### Bringing the subdomain up

1. **hPanel → Domains → Subdomains** — create `games` under `boundlessfiction.com`. Note the
   document root path it reports; everything below depends on it.
2. **hPanel → Advanced → Git** — *Connect with GitHub*, authorise, choose `Soumik766/games`,
   branch `main`, and set the deploy directory to **the subdomain's own `public_html`**.

   Two ways this goes wrong, both of which cost an afternoon:

   - **The wrong `public_html`.** The main site `boundlessfiction.com` has one too, and it
     holds the manga site. Hostinger's Git deploy *replaces* files in its target, so deploying
     there would drop this site's `index.html` on top of that homepage. Check the path with
     *Change* on the Git page and make it match what **Domains → Subdomains** reports.
   - **A directory that is not empty.** The first deployment refuses unless the target is
     empty, and Hostinger leaves a "Default page" placeholder in every new subdomain folder.
     Delete it first, including any hidden `index.php` or `default.php`.

   The first deployment has to be started by hand with the **Deploy** button. Pushing to
   `main` only deploys automatically once that first run has succeeded.
3. **hPanel → Security → SSL** — issue a certificate **for the subdomain**. A certificate
   covering `boundlessfiction.com` does not automatically cover `games.boundlessfiction.com`.
   Then turn on Force HTTPS.

After that, every `git push` deploys: GitHub notifies Hostinger, Hostinger pulls, files are
replaced. There is no build step.

### Verify

```bash
curl -I https://games.boundlessfiction.com/app-ads.txt   # expect 200 and text/plain
```

Then open each of these and confirm the padlock:

```text
https://games.boundlessfiction.com/
https://games.boundlessfiction.com/app-ads.txt
https://games.boundlessfiction.com/privacy-policy/
https://games.boundlessfiction.com/terms/
https://games.boundlessfiction.com/games/selfbound/
```

---

## What depends on this site being up

- **Google Play listings.** The Developer Website for every game is
  `https://games.boundlessfiction.com/` — the root, never a game page. The privacy policy URL
  is `https://games.boundlessfiction.com/privacy-policy/`. A policy URL that 404s means a
  rejected review.
- **AdMob.** `app-ads.txt` at the root carries publisher ID `pub-2623266578899875`. AdMob
  matches it against the Developer Website domain from the Play listing, re-crawls about
  daily, and the console warning lags a few days behind reality. Verify with the URL, not the
  console.
- **The games themselves.** Selfbound links to the policy and terms from its onboarding and
  Settings, through `GameConst.privacyPolicyUrl` and `GameConst.termsUrl` in
  `lib/app/constants.dart` of the game repository.
