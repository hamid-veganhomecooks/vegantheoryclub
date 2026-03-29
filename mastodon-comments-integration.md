# Mastodon Comments Integration
## Vegan Theory Club — Hugo + Blowfish

This documents how the Mastodon/fediverse comments system works on this site, how to maintain it, and how it was built.

Special thanks to https://github.com/louis-vgn for linking their mastodon comments integration in this closed issue for blowfish https://github.com/nunocoracao/blowfish/issues/1837#issuecomment-3341943246

---

## How it works

Each reading post can be linked to a Mastodon post (toot). When a reader visits the page and clicks "Load comments", a JavaScript web component fetches replies to that toot from the Mastodon API and renders them inline on the page.

- No database, no server, no cost
- Anyone on any fediverse instance (Mastodon, Pleroma, Akkoma, etc.) can reply
- Comments only appear when the reader clicks the button — nothing auto-loads
- If no `tootId` is set in the front matter, the comments section is hidden entirely

---

## File structure

These are the files that make the integration work. All are in your project root, not inside the theme.

```
vegantheoryclub/
  assets/
    js/
      mastodon-comments.js      ← Web component that fetches and renders replies
      purify-3.2.7.min.js       ← DOMPurify: sanitises HTML from Mastodon before rendering
  layouts/
    partials/
      comments.html             ← Hugo partial: loads the JS files and renders the component
```

### What each file does

**`mastodon-comments.js`**
A custom HTML web component (`<mastodon-comments>`) that calls the Mastodon API endpoint:
```
https://{instance}/api/v1/statuses/{tootId}/context
```
It renders the replies as a threaded comment section with dark/light mode support. Source: https://github.com/dpecos/mastodon-comments

**`purify-3.2.7.min.js`**
DOMPurify sanitisation library. Mastodon returns reply content as raw HTML. This library strips any potentially malicious content before it's rendered on the page. Source: https://github.com/cure53/DOMPurify

**`comments.html`**
The Hugo partial that wires everything together. It:
1. Guards against running on pages with no `tootId` front matter
2. Loads both JS files through Hugo's asset pipeline (with fingerprinting for cache-busting)
3. Renders the `<mastodon-comments>` web component with the instance/user/tootId values from front matter

---

## How Blowfish calls comments.html

Blowfish's article layout (`themes/blowfish/layouts/_default/single.html`) already has a comments slot built in. It calls `partials/comments.html` when two conditions are met:

1. The file `layouts/partials/comments.html` exists in your project
2. The page has `showComments: true` in its front matter

This means you don't need any custom hooks — Blowfish handles the wiring. Your `comments.html` just needs to exist in the right place.

---

## Adding comments to a reading post

### Step 1 — Post to Mastodon

Post about the week's reading from your `@vegantheoryclub@veganism.social` account. Include a link to the reading page on the site.

### Step 2 — Get the toot ID

The toot ID is the long number at the end of the post URL:

```
https://veganism.social/@vegantheoryclub/116292524854491317
                                         ^^^^^^^^^^^^^^^^^^
                                         this is the tootId
```

### Step 3 — Add to front matter

Open the reading's markdown file and add these fields:

```yaml
---
title: "Black Awakening in Capitalist America"
date: 2025-03-24
draft: false

author: "Robert L. Allen"
source_url: "https://archive.org/details/blackawakeningin00alle"
scheduled_date: "2025-03-31"
topics: ["black power", "african americans", "1969"]

showComments: true
instance: "veganism.social"
user: "vegantheoryclub"
tootId: "116292524854491317"
---
```

### Step 4 — Push

```bash
git add .
git commit -m "add week X reading with mastodon comments"
git push
```

Cloudflare rebuilds in ~1 minute. The comments section will appear at the bottom of the reading page.

---

## comments.html — full file contents

```html
{{ if .Params.tootId }}
{{- $jsDOMPurify := resources.Get "js/purify-3.2.7.min.js" -}}
{{- $jsDOMPurify = $jsDOMPurify | resources.ExecuteAsTemplate "js/purify-3.2.7.min.js" . | resources.Fingerprint (.Site.Params.fingerprintAlgorithm | default "sha512") -}}

{{- $jsMastodonComments := resources.Get "js/mastodon-comments.js" -}}
{{- $jsMastodonComments = $jsMastodonComments | resources.ExecuteAsTemplate "js/mastodon-comments.js" . | resources.Minify | resources.Fingerprint (.Site.Params.fingerprintAlgorithm | default "sha512") -}}

<script
    type="text/javascript"
    src="{{- $jsDOMPurify.RelPermalink -}}"
    integrity="{{- $jsDOMPurify.Data.Integrity -}}" defer>
</script>

<mastodon-comments host="{{- .Params.instance -}}" user="{{- .Params.user -}}" tootId="{{- .Params.tootId -}}"></mastodon-comments>
<script
    type="text/javascript"
    src="{{- $jsMastodonComments.RelPermalink -}}"
    integrity="{{- $jsMastodonComments.Data.Integrity -}}" defer>
</script>
{{ end }}
```

---

## Hugo asset pipeline — why files go where they do

This is important to understand when adding new images or scripts.

| Where you're linking from | File must go in | Example path in config/markdown |
|---|---|---|
| Config files (`.toml`) | `assets/` | `img/logo.png` |
| `comments.html` via `resources.Get` | `assets/` | `js/purify-3.2.7.min.js` |
| Markdown content `![](...)` | `static/` | `images/photo.jpg` |
| Direct browser URL | `static/` | `/images/photo.jpg` |

**`assets/`** — files processed by Hugo's pipeline (resized, fingerprinted, minified). Not directly URL-accessible. Used by config and templates.

**`static/`** — files copied as-is to the output. Directly URL-accessible. Used for images in markdown, favicons, downloads.

---

## Limitations and moderation

Since the site uses `@vegantheoryclub@veganism.social`, we do not have admin/moderator powers on that instance. Moderation options if a reply is objectionable:

1. **Ask a moderator at veganism.social** to delete the post — it will disappear from the comments on next load
2. **Block the user** This should prevent it from being displayed 
2. **Delete toot** — removes the thread entirely (loses good replies too, nuclear option)
3. **Remove `tootId` from the front matter** — hides the comments section for that post without deleting anything on Mastodon

### Future option
Running a single-user Akkoma instance on a cheap VPS (Hetzner around €4/month) would give full mod controls. Managed hosting options like Masto.host ( around $6-9/month) is an alternative with no server management required.

---

## Updating the JS dependencies

To update DOMPurify or the mastodon-comments component in future, always get the **raw file** not the GitHub webpage:

```
# DOMPurify latest
https://raw.githubusercontent.com/cure53/DOMPurify/main/dist/purify.min.js

# mastodon-comments
https://raw.githubusercontent.com/dpecos/mastodon-comments/master/mastodon-comments.js
```

After replacing the files, update the filename reference in `comments.html` if the version number in the filename changed (e.g. `purify-3.2.7.min.js` → `purify-3.3.0.min.js`).

---

## Troubleshooting

**Comments section not appearing at all**
- Check that `showComments: true` is in the front matter (YAML colon syntax, not `=`)
- Check that `tootId` is set in the front matter
- Check that `layouts/partials/comments.html` exists in your project root

**Build error: `type <nil> not supported in Resource transformations`**
- `resources.Get` returned nil — a JS file is missing or the filename doesn't match exactly
- Check `assets/js/` contains both files with names matching what's in `comments.html`

**Build error: `unexpected < in expression on line 1: <!doctype html>`**
- One of the JS files is actually an HTML page wrapper — you downloaded the GitHub webpage instead of the raw file
- Re-download using the raw URLs above

**Comments load locally but show CORS error in browser console**
- This is expected on localhost. The Mastodon API blocks requests from `localhost` for security reasons
- Test comments on the live site at `vegantheoryclub.org` instead

**Comments section appears but loads nothing**
- The toot may have no replies yet
- Double-check the `tootId` matches the number at the end of the toot URL exactly
