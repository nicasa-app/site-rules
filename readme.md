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
| `urlTransformFrom` | `string` | — | Substring to find in extracted image URLs. |
| `urlTransformTo` | `string` | — | Replacement string for `urlTransformFrom` (e.g. swap a thumbnail path segment for the full-size one). |

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
- For sites that serve thumbnails and swap to full-size via URL path differences, use `urlTransformFrom` / `urlTransformTo`.

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
