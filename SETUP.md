# saurabh-pant.com — Blog Setup

This folder is the source for the blog at https://saurabh-pant.com/.
It generates a static site with Hugo + the PaperMod theme, hosted on
GitHub Pages via the existing `sto-nith/sto-nith.github.io` repo.

Setup date: 2026-06-13.

---

## Tools Installed

| Tool | Version installed | Where it lives | How to remove |
|---|---|---|---|
| Hugo (extended) | v0.163.1 | `/usr/local/bin/hugo` | `brew uninstall hugo` |

Nothing else was installed at the system level. No Node, no Ruby, no
Docker, no language runtimes. The entire blog toolchain is one binary.

### Architecture note (cleanup item, not a blocker)

This Mac is Apple Silicon (M3 Pro, `arm64`), but Homebrew is the Intel
build at `/usr/local/Cellar`, so Hugo runs under Rosetta. It works fine
— Hugo is fast enough that the translation overhead is invisible. To
clean this up later (out of scope here), reinstall Homebrew natively
into `/opt/homebrew` and re-`brew install hugo`.

---

## Folder Layout

```
blog/
├── archetypes/          Templates for `hugo new content`
├── assets/
│   └── css/extended/
│       └── custom.css   Editorial overrides (fonts, accent, post cards)
├── content/             Markdown — this is what you'll edit
│   ├── archives.md      Renders the /archives/ page
│   └── posts/
│       └── hello-world/
│           ├── index.md  ← the post
│           └── cover.svg ← its cover image (page bundle)
├── data/                Structured data files (empty)
├── i18n/                Translations (empty)
├── layouts/
│   └── partials/
│       └── extend_head.html   Loads Lora + Inter from Google Fonts
├── static/              Files copied verbatim into the site root
│   ├── CNAME            ⚠ Tells GitHub Pages our domain. Do not delete.
│   └── avatar.svg       Placeholder monogram avatar (replace with photo)
├── themes/
│   └── PaperMod/        Git submodule — do not edit directly
├── public/              Generated output. Git-ignored. Never edit.
├── resources/           Hugo's build cache. Git-ignored.
├── .github/workflows/
│   └── hugo.yml         CI: builds and deploys on push to main
├── .gitignore
├── .gitmodules
├── hugo.toml            Site config
├── prompts.txt          (your file, untouched)
└── SETUP.md             this file
```

---

## Commands That Were Run (in order)

```bash
# 1. Install Hugo
brew install hugo

# 2. Scaffold the site into this directory
cd "<this directory>"
hugo new site . --force --format=toml

# 3. Initialize git and add the theme as a submodule
git init -b main
git submodule add --depth=1 \
  https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 4. Wrote hugo.toml, static/CNAME, .gitignore, content/archives.md

# 5. Created the first post stub (still marked draft=true)
hugo new content posts/hello-world.md
# Then replaced the body with real content.

# 6. Test build (offline, no network)
hugo --buildDrafts

# 7. Live preview (drafts visible)
hugo server --buildDrafts
```

---

## Theme Customization (the "editorial" look)

PaperMod ships minimal by default. The look has been pushed to a more
editorial direction with three small, all-overridable files:

| File | What it does |
|---|---|
| `assets/css/extended/custom.css` | Custom fonts, warm-paper light theme, deep-ink dark theme, amber accent (`#b8862e`), post cards with hover lift, pill-shaped tags, animated underlines on nav links. PaperMod auto-loads any CSS in `assets/css/extended/`. |
| `layouts/partials/extend_head.html` | Loads Lora (serif headings) + Inter (sans body) from Google Fonts via PaperMod's `extend_head` hook. |
| `static/avatar.svg` | Placeholder "SP" monogram on dark background with amber ring. Used by ProfileMode home. |

### Replace the placeholder avatar with your photo

1. Drop a square JPG or PNG (recommended ~400x400) at `static/avatar.jpg`.
2. In `hugo.toml`, change `imageUrl = "/avatar.svg"` → `imageUrl = "/avatar.jpg"`.
3. Optionally delete `static/avatar.svg`.

### Change the accent color

Edit `assets/css/extended/custom.css` — there are two `--accent` lines
(one under `:root`, one under `.dark`). Pick a hex color and replace
both. Live preview reloads instantly.

### Add a cover image to a new post

Use a page bundle — a folder with `index.md` plus the cover image:
```
content/posts/my-post/
├── index.md
└── cover.png         (or .jpg, or .svg)
```
And in the post's front-matter:
```toml
[cover]
image = 'cover.png'
alt = 'short description'
relative = true
```

---

## Daily Workflow — Writing a Post

```bash
cd "<this directory>"

# Create the post (front-matter is auto-filled from archetypes/default.md)
hugo new content posts/my-new-post.md

# Edit content/posts/my-new-post.md in any editor.
# Front-matter starts with draft = true. Flip to false (or delete) to publish.

# Preview locally — auto-reloads on save
hugo server --buildDrafts

# When happy:
git add content/posts/my-new-post.md
git commit -m "post: short title"
git push                    # CI takes ~30s, site updates at saurabh-pant.com
```

That's the whole loop. Three commands per post.

---

## How Deploys Work

```
local edit → git push origin main
                  ↓
       GitHub Actions (.github/workflows/hugo.yml)
       1. Checks out repo + theme submodule
       2. Installs Hugo 0.163.1 on the runner
       3. Runs `hugo --minify`
       4. Uploads ./public/ as a Pages artifact
       5. Deploys to GitHub Pages
                  ↓
           https://saurabh-pant.com
```

The CNAME file in `static/` is copied into `public/` on every build, so
GitHub Pages always knows the custom domain.

---

## How to Go Live (NOT YET DONE)

Local preview is verified. The site has not been pushed to GitHub. When
ready:

1. **Set the GitHub remote.** The existing repo is `sto-nith/sto-nith.github.io`.
   Its current contents (the old "Saurabh was here!" `index.html`) will be
   overwritten by the Hugo deploy. Back up first if you want the original:
   ```bash
   git remote add origin https://github.com/sto-nith/sto-nith.github.io.git
   git fetch origin
   # Optional: tag the old state for posterity
   # git fetch origin main && git tag pre-hugo origin/main
   ```

2. **Switch GitHub Pages source to "GitHub Actions".**
   Go to: repo Settings → Pages → Build and deployment → Source → "GitHub Actions".

3. **Push.**
   ```bash
   git add -A
   git commit -m "blog: initial Hugo + PaperMod setup"
   git push -u origin main --force   # --force only because we're replacing the old single-file site
   ```

4. **Re-attach the custom domain.**
   Settings → Pages → Custom domain → enter `saurabh-pant.com`. Tick
   "Enforce HTTPS" once the cert provisions (~10 min).

5. **Flip the first post to non-draft when you want it visible.** Edit
   `content/posts/hello-world.md`, change `draft = true` to `draft = false`,
   commit, push.

---

## Risks & Rollback

| If this breaks | What it looks like | Fix |
|---|---|---|
| `static/CNAME` deleted | Custom domain detaches from Pages after next deploy | Recreate the file with `saurabh-pant.com` on a single line, push. |
| Theme submodule missing on fresh clone | `hugo` errors with "module not found" | `git submodule update --init --recursive` |
| Hugo upgraded locally but CI pinned to 0.163.1 | Local renders differently from prod | Bump `HUGO_VERSION` in `.github/workflows/hugo.yml` to match. |
| Want to undo a deploy | Old version live | `git revert <bad-commit> && git push` — CI redeploys the prior state. |
| Want to nuke the whole blog | Don't want this anymore | `brew uninstall hugo`, delete this folder, in GitHub Pages Settings remove the deploy workflow. The domain stays whatever you point it at next. |

---

## What's Pending (Outside This Setup)

- **Renew the domain.** Expires 2026-06-13. Auto-renew off.
- **Decide the fate of `contactme@saurabh-pant.com`** on Zoho — currently
  no MX records pointed at it, so no incoming mail.
- **Add GA4** if you want analytics. The old `UA-65628725-1` Universal
  Analytics tag from the previous `index.html` is dead — Google retired
  Universal Analytics in 2023. Drop a GA4 measurement ID into
  `params.googleAnalytics` in `hugo.toml` when ready.
