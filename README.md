# [Vegan Theory Club](https://vegantheoryclub.org)

A weekly reading group site built with [Hugo](https://gohugo.io/) and the [Blowfish theme](https://blowfish.page/), hosted on [Cloudflare Pages](https://pages.cloudflare.com/).

Special thanks to https://github.com/louis-vgn for linking their mastodon comments integration in this closed issue for blowfish https://github.com/nunocoracao/blowfish/issues/1837#issuecomment-3341943246

## Tools

These tools are hosted on the live site at `vegantheoryclub.org/tools/` and can be used from any browser without needing a local development environment.

- `static/tools/ics-generator.html` — open in any browser to generate and maintain `static/readings.ics`. Load the existing file to preserve past events, add new ones, and export the updated calendar.

- `static/tools/posting-tool.html` — must be opened from the live site (not as a local file) due to browser security restrictions. Paste your Mastodon access token from veganism.social → Settings → Development. The tool builds the toot, posts it to Mastodon, captures the toot ID, and generates the complete markdown file with all front matter pre-filled. Download the file and either push it via git or drag and drop it into GitHub to update the site.