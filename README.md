# Vegan Theory Club

A weekly reading group site built with [Hugo](https://gohugo.io/) and the [Blowfish theme](https://blowfish.page/), hosted on [Cloudflare Pages](https://pages.cloudflare.com/).

---

## First-time setup

### Prerequisites

- [Hugo (extended version)](https://gohugo.io/installation/) — install via `brew install hugo` (macOS) or see the Hugo docs for your OS
- [Git](https://git-scm.com/)
- A [GitHub account](https://github.com/)
- A [Cloudflare account](https://cloudflare.com/) (free tier is fine)

### 1. Clone this repo

```bash
git clone --recurse-submodules https://github.com/YOUR_USERNAME/vegan-theory-club.git
cd vegan-theory-club
```

> **Important:** The `--recurse-submodules` flag is required — it pulls in the Blowfish theme.  
> If you forget it, run: `git submodule update --init --recursive`

### 2. Install the Blowfish theme as a submodule

If you're setting this up fresh (not cloning an existing repo):

```bash
git init
git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish
```

### 3. Preview locally

```bash
hugo server
```

Open [http://localhost:1313](http://localhost:1313) in your browser. The site hot-reloads as you edit files.

---

## Connecting to Cloudflare Pages

### 1. Push your repo to GitHub

```bash
git add .
git commit -m "initial setup"
git push origin main
```

### 2. Create a Cloudflare Pages project

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Go to **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
3. Authorise GitHub and select your `vegan-theory-club` repository
4. Use these build settings:

| Setting | Value |
|---|---|
| Framework preset | Hugo |
| Build command | `hugo --gc --minify` |
| Build output directory | `public` |
| Environment variable | `HUGO_VERSION` = `0.145.0` (or latest) |

> Setting `HUGO_VERSION` explicitly prevents Cloudflare from using an outdated Hugo version.

5. Click **Save and Deploy**

That's it. Every time you push to `main`, Cloudflare rebuilds and redeploys the site automatically.

### 3. (Optional) Add a custom domain

In your Cloudflare Pages project → **Custom domains** → add your domain. If your domain's DNS is also on Cloudflare, it'll configure automatically.

---

## Adding a new reading

### Using Hugo's CLI (quickest)

```bash
hugo new readings/2025-04-07-care-work-federici.md
```

This creates a new file from the `archetypes/readings.md` template. Fill in the front matter fields and write your description, then set `draft: false` when ready to publish.

### Or just copy an existing file

Copy any file from `content/readings/`, rename it with the new date and slug, and edit the front matter.

### Front matter fields

```yaml
---
title: "Title of the text"
date: 2025-04-07          # When you're adding this post
draft: false

author: "Author Name"
source_url: "https://link-to-full-text.com"
scheduled_date: "2025-04-07"   # Week this is scheduled for

topics: ["ethics", "ecofeminism"]  # Used for tag/browse pages
---
```

### Publish

```bash
git add .
git commit -m "add week 5 reading: Care Work - Federici"
git push
```

Cloudflare will pick up the push and redeploy within ~1 minute.

---

## Customising the theme

The main config files are all in `config/_default/`:

| File | What it controls |
|---|---|
| `hugo.toml` | Base URL, site title, theme |
| `params.toml` | Colour scheme, layout, feature toggles |
| `languages.en.toml` | Site description, author, date format |
| `menus.en.toml` | Navigation links |

### Changing the colour scheme

Edit `colorScheme` in `config/_default/params.toml`. Options include:
`avocado`, `ocean`, `fire`, `neon`, `slate`, `concrete`, `blowfish`

Full Blowfish config reference: https://blowfish.page/docs/configuration/

---

## Keeping the theme up to date

```bash
git submodule update --remote --merge
git commit -am "update blowfish theme"
git push
```

---

## Disable Rocket Loader (important!)

Cloudflare's Rocket Loader feature breaks Blowfish's dark/light mode switcher.  
Go to **Cloudflare Dashboard** → your domain → **Speed** → **Optimization** → **Rocket Loader** → turn it **OFF**.
