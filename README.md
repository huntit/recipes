# recipes

Recipe pages, nicely formatted for import into the MealBoard app.

**Live site:** https://huntit.github.io/recipes/

---

## Required setup

`_config.yml` must exclude this file from the Jekyll build:

```yaml
exclude:
  - README.md
```

Without it the build fails, because the Liquid examples below would be
executed instead of displayed. GitHub still renders this README on the repo
page — `exclude` only keeps Jekyll's hands off it.

---

## How it works

GitHub Pages serves this repo from `main`, root folder, with Jekyll enabled.

- One `.html` file per recipe, in the root, plus an optional `.jpg` photo of
  the same basename. Fonts come from the Google Fonts CDN; everything else
  (CSS, JSON-LD) is inline in the HTML.
- Filenames are kebab-case and become part of the URL:
  `slow-cooker-lamb-shoulder-rolls.html`
- `index.md` generates the recipe list automatically. Adding a recipe never
  requires editing the index.

## Adding a recipe

1. Write a complete standalone HTML document.
2. Include a `schema.org/Recipe` JSON-LD block in the `<head>` — this is what
   MealBoard reads on import. See an existing recipe for the shape.
3. Add the front matter block and Liquid guard (below).
4. Commit to `main`. The build takes a minute or two.

## Front matter and the Liquid guard

Every recipe file starts with this, then wraps the whole document:

```
---
layout: null
recipe: true
title: Slow Cooker Lamb Shoulder Rolls
blurb: One-line description shown on the index page.
meat: Lamb
serves: 6
---
{% raw %}<!DOCTYPE html>
...entire document...
</html>{% endraw %}
```

**`layout: null`** stops the theme wrapping the recipe in its own page shell,
which would produce nested `<html>` elements and break the standalone file.

**`recipe: true`** is what `index.md` filters on. Omit it and the recipe won't
be listed.

**The `raw` guard is not optional.** Front matter turns the file into a Jekyll
page, so Liquid parses the whole thing and consumes anything in `{{ }}` or
`{% %}`. Real example already in these files:

```css
html{-webkit-text-size-adjust:100%}
```

That trailing `%}` reads as a Liquid tag terminator and breaks the build. CSS
and JSON-LD are full of braces, so wrap the document and don't think about it.

## How index.md builds the list

Two loops, deliberately:

```liquid
{% assign cards = site.pages | where: "recipe", true | sort: "title" %}
{% assign legacy = site.static_files | where_exp: "f", "f.extname == '.html'" %}
```

- `cards` — files with front matter. Real titles and blurbs.
- `legacy` — plain `.html` files with no front matter, which Jekyll treats as
  static files. Listed with a title derived from the filename.

The second loop exists so older recipes stay visible until they're converted.
Once every file has front matter, delete it.

`relative_url` is required on every link. The site is served from `/recipes/`,
not the domain root, so a bare path generates a broken link.

## Recipe photos

Each recipe can have a `<basename>.jpg` sitting next to its `.html`, e.g.
`slow-cooker-lamb-shoulder-rolls.jpg`. Wire it up in three places:

- `og:image` meta tag in `<head>`, absolute URL.
- `"image"` field in the JSON-LD `Recipe` block, same absolute URL. This is
  what MealBoard actually reads.
- A `<figure class="hero">` at the top of `<body>`, with an `<img>` and a
  `<figcaption>` crediting the photographer and license.

Source photos from something with a clear, reusable license — Wikimedia
Commons (CC0 / CC-BY / CC-BY-SA) is the easiest to check and credit. Resize to
1600px wide and strip EXIF/GPS before committing:

```
magick source.jpg -auto-orient -strip -resize 1600x -quality 80 \
  -sampling-factor 4:2:0 -interlace JPEG <basename>.jpg
```

Use an absolute `https://huntit.github.io/recipes/...` URL, not a relative
path or a data URI — MealBoard (and most recipe importers) fetch the image
URL with a separate request rather than reading embedded image data.

## MealBoard import

Point MealBoard's web import at the GitHub Pages URL:

```
https://huntit.github.io/recipes/<filename>.html
```

Not the `raw.githubusercontent.com` URL — that serves as `text/plain` and the
JSON-LD block won't be parsed.

Schema has no clean way to express a cook split across two days. Where a recipe
is cooked one day and reheated the next, `cookTime` covers the first stage only
and the rest lives in the instruction steps, so an imported total may look
short.

## Conventions

- Australian English, metric, °C.
- Where a recipe has a low-sodium option, the pot is cooked unsalted and a
  portion split off before seasoning. Sodium figures quoted are typical
  Australian supermarket values.
- Recipe pages share one visual system: a two-day timeline rail, sticky
  ingredient column, and print styles. Copy an existing file as the starting
  point rather than writing fresh CSS.
