# nicasa site-rules

Community-maintained image extraction rules for the [Nicasa](https://apps.apple.com/us/app/nicasa/id6756032669) browser extension. The extension fetches this file periodically to keep its built-in site support up to date without requiring an app update.

## How it works

`rules.json` is fetched from:

```
https://raw.githubusercontent.com/nicasa-app/site-rules/refs/heads/main/rules.json
```

The extension caches the result in `chrome.storage.local` and refreshes it:
- Once on install / update
- Daily via an alarm
- On demand via the **"↻ Check for updates"** button in the Options page

---

## `rules.json` schema

```jsonc
{
  "version": "1.0.0",         // semver — bump on every meaningful change
  "updatedAt": "2026-04-10T00:00:00Z",
  "categories": [
    {
      "name": "Category Label",
      "rules": [ /* SiteRule[] */ ]
    }
  ]
}
```

### `SiteRule` fields

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | ✓ | Stable identifier, prefix `builtin_`. Never change after publish. |
| `name` | `string` | ✓ | Display name shown in the Options UI. |
| `hostPattern` | `string` | ✓ | Glob pattern matched against `location.hostname`. Use `*.example.com` to include subdomains. |
| `enabled` | `boolean` | ✓ | Default enabled state when the rule is first activated. |
| `imageSelector` | `string` | ✓ | CSS selector(s) (comma-separated) used to find `<img>` elements on the page. |
| `minWidth` | `number` | ✓ | Minimum image width in pixels to accept (0 = no limit). |
| `minHeight` | `number` | ✓ | Minimum image height in pixels to accept (0 = no limit). |
| `excludePatterns` | `string[]` | ✓ | URL substrings — images whose `src` contains any of these are discarded (avatars, icons, etc.). |
| `urlTransformPipeline` | `string` | — | A pipeline of transform steps (separated by ` \| `) applied in order to every extracted image URL. See [URL transform pipeline](#url-transform-pipeline) below. |
| `urlExtFallbacks` | `string[]` | — | If the (transformed) image URL fails to load with a 4xx error, the viewer retries with each extension in order. Example: `["png", "webp"]` tries `.png` then `.webp`. No pre-flight requests are issued — retries happen only on actual load failure. |
| `urlTransforms` | `{ from, to }[]` | — | **Deprecated.** Array of plain-string find & replace pairs. Superseded by `urlTransformPipeline`. |
| `urlTransformFrom` | `string` | — | **Deprecated.** Single find substring. Superseded by `urlTransformPipeline`. |
| `urlTransformTo` | `string` | — | **Deprecated.** Single replacement string. Superseded by `urlTransformPipeline`. |

---

## Supported sites

### Art & Illustration
| Site | Host pattern |
|---|---|
| Pixiv | `*.pixiv.net` |
| ArtStation | `*.artstation.com` |
| DeviantArt | `*.deviantart.com` |
| Behance | `*.behance.net` |

### Photography
| Site | Host pattern |
|---|---|
| Flickr | `*.flickr.com` |
| 500px | `*.500px.com` |
| Unsplash | `unsplash.com` |
| Pexels | `*.pexels.com` |
| VSCO | `vsco.co` |

### Social Media
| Site | Host pattern |
|---|---|
| X (x.com) | `x.com` |
| Reddit | `*.reddit.com` |
| Tumblr | `*.tumblr.com` |

### Image Boards & Galleries
| Site | Host pattern |
|---|---|
| Pinterest | `*.pinterest.com` |
| Imgur | `imgur.com` |
| Zerochan | `www.zerochan.net` |
| Wallhaven | `wallhaven.cc` |

### Anime & Manga
| Site | Host pattern |
|---|---|
| MyAnimeList | `myanimelist.net` |
| AniList | `anilist.co` |
| Niconico Seiga | `seiga.nicovideo.jp` |

### Stock & Microstock
| Site | Host pattern |
|---|---|
| Shutterstock | `*.shutterstock.com` |
| Getty Images | `*.gettyimages.com` |

---

## Adding or updating a rule

1. Fork this repository.
2. Edit `rules.json`:
   - Increment `version` (e.g. `1.0.0` → `1.0.1` for a patch, `1.1.0` for a new site).
   - Update `updatedAt` to the current ISO-8601 date.
   - Add your rule object to the appropriate category (or create a new category).
3. Open a pull request with a brief description of the site and why the selector works.

### Tips for writing selectors

- Prefer **stable structural selectors** (`figure img`, `.gallery img`) over generated class names.
- Use `excludePatterns` to filter out avatars, icons, and thumbnails — this prevents clutter in the Nicasa picker.
- Test `minWidth` / `minHeight` against the site's actual content images; `0` means no size filter.
- For sites that serve thumbnails and swap to full-size via URL path differences, use `urlTransformPipeline` (see below).

---

## URL transform pipeline

`urlTransformPipeline` is a plain string of function calls separated by ` | `. Steps are applied left-to-right to the image URL.

### Available functions

| Function | Syntax | Description |
|---|---|---|
| `replace` | `replace('old', 'new')` | Plain-string find & replace (all occurrences). |
| `re` | `re('pattern', 'replacement')` | Regex find & replace. `$1`, `$2` … capture groups are supported. |
| `rm` | `rm('substring')` | Remove every occurrence of a substring. |
| `lastafter` | `lastafter('sep', 'prepend')` | Take the last segment after `sep`, optionally prefix it, and return the part of the URL up to and including `sep` with the new filename. |
| `param` | `param('key', 'value')` | Add or update a URL query parameter. |

### Examples

```
# Wallhaven — swap thumbnail domain/path, then prefix the filename
# (full images may be .png/.webp; pair with urlExtFallbacks for load-time retry)
replace('th.wallhaven.cc/small','w.wallhaven.cc/full') | lastafter('/', 'wallhaven-')

# Generic thumbnail → full-size via regex
re('_[0-9]+x[0-9]+\\.', '.')

# Add or override a quality parameter
param('quality', '100')
```

**Wallhaven full rule example** (pipeline + extension fallbacks):

```jsonc
{
  "urlTransformPipeline": "replace('th.wallhaven.cc/small','w.wallhaven.cc/full') | lastafter('/', 'wallhaven-')",
  "urlExtFallbacks": ["png", "webp"]
}
```

---

## Versioning

`version` follows [semver](https://semver.org):

| Change | Version bump |
|---|---|
| Fix a broken selector | patch (x.y.**Z**) |
| Add a new site | minor (x.**Y**.0) |
| Rename a field / breaking schema change | major (**X**.0.0) |

---

## License

[MIT](LICENSE)
