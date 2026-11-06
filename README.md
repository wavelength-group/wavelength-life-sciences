# Wavelength Life Sciences — website repo & deployment handoff

**For:** John's company Claude / whoever sets up the GitHub + Netlify pipeline
**From:** the build of the Wavelength Life Sciences marketing site
**Goal:** move this site from manual drag‑and‑drop deploys onto **GitHub → Netlify continuous deployment**, the same way John's other websites are set up, on the **same Netlify account** that already hosts it.

---

## 1. What this site is

A single‑page marketing site for **Wavelength Life Sciences** (recruitment, coaching and talent for the life sciences; part of the Wavelength Group).

- It is a **static site** — plain HTML/CSS/JS. **No build step, no framework, no npm install, no bundler.**
- The whole site is one file, `index.html`, using **client‑side hash routing** (`#/services`, `#/roles`, `#/bioavailable`, etc.). Every "page" is a section inside that one file.
- There is one extra real page at `bioavailable/index.html` — a tiny redirect page that gives the Bioavailable register its own shareable URL with its own social preview (see §6).
- Live now at **https://wavelengthlifesciences.com** (and `www.`), currently served by a Netlify **drag‑and‑drop** deploy.

**The ask:** put these files in a GitHub repo and connect that repo to the existing Netlify site, so that from now on a `git push` publishes the site automatically — no more manual drops.

---

## 2. Current state (read before changing anything)

- **Hosting:** Netlify, on John's existing Netlify account/team. The site was last deployed by dragging a folder onto the Netlify "Deploys" drop zone. Its Netlify subdomain is **`wavelengthlifesciences.netlify.app`**.
- **Custom domain + HTTPS:** `wavelengthlifesciences.com` + `www` are already attached to that Netlify site, using **Netlify DNS**. The domain was bought at GoDaddy and its **nameservers already point to Netlify** (`dns1–dns4.p04.nsone.net`). This is done and working — **do not touch the GoDaddy nameservers or the DNS.**
- **Netlify Forms:** the contact form is wired for Netlify Forms. Form detection is enabled on the site and there is an email notification to **john@wavepeople.com**. This is a per‑site setting and will be preserved as long as the **existing site** is kept (see §5).
- **Analytics:** Google Analytics (gtag `G-8R7NH0EBYX`) is embedded in `index.html`.

---

## 3. What's in this repo

```
.
├── index.html              ← the entire website (built & production‑ready)
├── netlify.toml            ← Netlify config: publish ".", security headers
├── og-image.png            ← social share card for the homepage (1200×630)
├── og-bioavailable.png     ← social share card for the Bioavailable page (1200×630)
├── bioavailable/
│   └── index.html          ← shareable /bioavailable URL (redirects to #/bioavailable)
└── README.md               ← this file
```

That is the **complete** publishable site. The repo root **is** the publish directory. Nothing needs to be compiled.

> `index.html` is already the production build: the contact form carries the Netlify Forms attributes (`data-netlify="true"`, hidden `form-name`, honeypot) and the social/Open‑Graph meta tags are injected. Commit it as‑is.

---

## 4. Netlify build settings (important: there is no build)

When connecting the repo, set:

| Setting | Value |
|---|---|
| **Base directory** | *(empty)* |
| **Build command** | *(empty / leave blank)* |
| **Publish directory** | `.` (repo root) |
| **Functions directory** | *(empty)* |

`netlify.toml` already declares `publish = "."` and the security headers, so Netlify will pick these up automatically. If the UI insists on a build command, leave it blank — this is a "no‑build" static site.

---

## 5. Recommended setup — link the **existing** site to Git (keeps everything)

This is the least‑disruptive path and the one to use **if the site can stay on the Netlify team it is already on** (which is the same account John uses for his other sites). It keeps the site ID, the custom domain, the HTTPS certificate, and the Forms configuration untouched — it only swaps the deploy method from "manual drop" to "deploy from Git".

**Step 1 — create the GitHub repo and push these files.**

```bash
# from inside this folder
git init
git add .
git commit -m "Wavelength Life Sciences site — initial import for Netlify CI/CD"
gh repo create wavelength-life-sciences --private --source=. --remote=origin --push
# (use --public instead of --private if that matches how the other sites are set up)
```

**Step 2 — link the existing Netlify site to that repo.** In the Netlify dashboard:

1. Open the existing **wavelengthlifesciences** site (the one serving `wavelengthlifesciences.com`).
2. **Site configuration → Build & deploy → Continuous deployment → "Link repository"** (on a drop‑deployed site this button is sometimes labelled *"Link site to Git"* or *"Set up continuous deployment"*).
3. Authorise GitHub, pick the `wavelength-life-sciences` repo, branch `main`.
4. Build command: **blank**. Publish directory: **`.`**. Save.
5. Trigger one deploy (**Deploys → Trigger deploy → Deploy site**) to confirm the Git deploy succeeds.

Because this is the same site object, the custom domain, SSL and Forms all carry over with no further action.

> CLI alternative to Step 2, if preferred: `netlify link` (to the existing site) then connect the repo via `netlify sites:update` / the dashboard. The dashboard flow above is the reliable one.

From now on: **push to `main` → Netlify builds and publishes automatically.**

---

## 6. Alternative — new site from Git (only if it must move to a different team)

Use this **only** if the site has to live under a different Netlify team than where it currently is. It involves re‑attaching the domain and re‑enabling Forms, so there is a little more care required.

1. Push the files to GitHub as in §5 Step 1.
2. Netlify → **Add new site → Import an existing project → GitHub →** pick the repo. Build command blank, publish `.`.
3. Let it deploy to its temporary `*.netlify.app` URL and verify it looks right.
4. **Move the custom domain:** remove `wavelengthlifesciences.com` + `www` from the **old** site, then add them to the **new** site (**Domain management → Add domain**). Because the domain already uses Netlify DNS, this is an internal reassignment — **the GoDaddy nameservers still must not change.** Netlify will re‑issue the HTTPS certificate automatically.
5. **Re‑enable Netlify Forms** on the new site: **Forms → enable form detection**, then redeploy once so the form is registered, and re‑add the email notification to **john@wavepeople.com** (**Forms → Form notifications**).
6. Set the primary domain (apex vs `www`) to match the old site, and delete the old drop‑deployed site once the new one is confirmed live.

Option A (§5) avoids all of step 4–6, which is why it's preferred.

---

## 7. Things that must keep working (checklist after go‑live)

- [ ] `https://wavelengthlifesciences.com` loads with a valid padlock (HTTPS).
- [ ] `https://www.wavelengthlifesciences.com` loads (redirects to the primary, whichever John set).
- [ ] The **contact form** (`#/contact`) still submits and the submission appears under **Netlify → Forms**, with the email reaching **john@wavepeople.com**. (The form is named `contact`; detection must be ON.)
- [ ] `https://wavelengthlifesciences.com/og-image.png` and `/og-bioavailable.png` both load (these are the social share cards).
- [ ] `https://wavelengthlifesciences.com/bioavailable` loads and forwards to the Bioavailable register.
- [ ] The **GoDaddy nameservers are unchanged** (`dns1–dns4.p04.nsone.net`). DNS is managed in **Netlify**, not GoDaddy.

If social previews look stale after any change to the OG images, re‑scrape via the **LinkedIn Post Inspector** and Facebook **Sharing Debugger**.

---

## 8. Making content changes from now on

For ordinary text edits (new candidate on the Bioavailable register, a new role ad, copy tweaks), edit **`index.html`** directly, commit and push — Netlify redeploys. The content lives as plain HTML inside `index.html`:

- Bioavailable candidate cards are in the section with `data-page="bioavailable"` (a `.cand-index` list at the top + `.cand` article cards below). Reference codes follow the convention **CC‑LS‑YYMM{letter}** (CC = Ccentric, LS = Life Sciences practice, YYMM, incrementing suffix).
- The role ad is in the section with `data-page="roles"` (English + German `.role-post` blocks).

The social share images (`og-image.png`, `og-bioavailable.png`) are static PNGs — swap the files if the design changes. They were generated from HTML templates rendered at 1200×630 (IBM Plex typeface, the brand's dark theme and spectral‑wave motif); the generator scripts are not required to run the site and are not included here, but can be provided if you want to regenerate the cards.

---

## 9. Quick reference

| Item | Value |
|---|---|
| Production URL | https://wavelengthlifesciences.com |
| Netlify subdomain | wavelengthlifesciences.netlify.app |
| Domain registrar | GoDaddy (nameservers → Netlify DNS: `dns1–dns4.p04.nsone.net`) |
| DNS managed in | **Netlify** (not GoDaddy) |
| Deploy type wanted | GitHub → Netlify continuous deployment |
| Build command | *(none — static site)* |
| Publish directory | `.` (repo root) |
| Contact form | Netlify Forms, form name `contact`, notify `john@wavepeople.com` |
| Analytics | Google Analytics `G-8R7NH0EBYX` |
| Contact | Dr John Bethell — john@wavepeople.com |
