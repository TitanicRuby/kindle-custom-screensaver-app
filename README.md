# E-Reader Screensaver Converter

A static, browser-only tool for converting your own photos into screensaver images for Kindle, Kobo, and reMarkable devices — no Python, no SSH, no install. It shares the same core idea as [`batch_convert.py`](https://github.com/TitanicRuby/kindle-custom-screensaver/blob/main/batch_convert.py) from my [kindle-custom-screensaver](https://github.com/TitanicRuby/kindle-custom-screensaver) project (resize/crop to your device's exact resolution, convert to grayscale, map onto a 16-level palette), but usable by anyone with a browser — no editing a script, no command line.

> **Note:** hobby project, not a maintained/audited tool — use at your own risk, and feel free to open an issue or PR if you spot something wrong. This tool only handles the *image conversion* step; you still need to get the resulting PNGs onto your device yourself (see [kindle-custom-screensaver](https://github.com/TitanicRuby/kindle-custom-screensaver) for the Kindle jailbreak + SSH workflow, or your device's own screensaver-folder instructions).

**100% client-side.** Your photos are processed entirely in your browser using the Canvas API — nothing is uploaded anywhere, there's no server, and no accounts. You can even disconnect from the internet after the page loads and it'll keep working.

**Single file.** `index.html` is fully self-contained — all CSS and JS (including the vendored libraries) are inlined into it, so there's nothing else it needs to load. That's deliberate: sandboxed/restricted browsers (Flatpak or Snap builds of Firefox, some locked-down corporate setups) often grant a locally-opened HTML file access to itself but *not* to sibling files sitting next to it in the same folder, which silently breaks a multi-file version (page loads, but nothing JS-driven works — no error dialog, just an empty dropdown). A single file has no sibling files to lose access to.

---

## Using it

No install, no account, no command line. Two steps: get the file, open it.

### 1. Get `index.html`

Only that **one file** is needed — you don't need to clone the repo or install anything.

- On GitHub, click on `index.html` in this repo, then click the **download icon** (⬇, near the top-right of the file view — it may be labeled "Download raw file"). Save it wherever you like (Desktop, Downloads, doesn't matter).
- If your browser doesn't show a download icon there: click **Raw** instead, then use your browser's "Save Page As" — Ctrl+S on Windows/Linux, Cmd+S on Mac — and make sure it saves as `index.html`, not `.txt`.
- Alternative if the above is confusing: on the repo's main page, click the green **Code** button → **Download ZIP**, extract it anywhere, and find `index.html` at the top level of the extracted folder.

### 2. Open it

- **Windows:** double-click `index.html` in File Explorer.
- **macOS:** double-click `index.html` in Finder.
- **Linux:** double-click it in your file manager (Files/Nautilus, Dolphin, etc.).

It should open straight into your default browser (Chrome, Firefox, Edge, Safari, Brave — any modern one works) and you're ready to go.

> **If it opens in a text/code editor instead of a browser** — this happens if `.html` files on your system are set to open in something like VS Code or Notepad rather than a browser — right-click the file → **Open with** → pick your browser instead. Or open your browser first and drag the `index.html` file into the browser window.

> **On a phone or tablet:** opening a locally-downloaded HTML file isn't as reliable on mobile (especially iOS) as it is on a desktop. If this repo has GitHub Pages enabled (check for a live link at the top of the repo page), just visiting that link in your mobile browser is the more reliable path there.

That's it — no server, no other files needed alongside it, and once the page has loaded you can even go offline; it keeps working.

### The flow, once it's open

Upload photo(s) → pick your device's screen size → pick a crop mode → optionally remove a flat background → adjust the crop if using manual mode → pick dithering on/off → set a naming pattern if converting a batch → convert → download.

### Supported devices

| Device | Resolution |
|---|---|
| Kindle (11th Gen) | 1072×1448 |
| Kindle Paperwhite (10th Gen) | 1072×1448 |
| Kindle Paperwhite (11th Gen) | 1236×1648 |
| Kindle Paperwhite (12th Gen) | 1264×1680 |
| Kindle Oasis (3rd Gen) | 1264×1680 |
| Kindle Colorsoft | 1264×1680 |
| Kindle Scribe | 1860×2480 |
| Kobo Nia | 758×1024 |
| Kobo Clara BW / Clara 2E / Clara Colour | 1072×1448 |
| Kobo Libra 2 / Libra Colour | 1264×1680 |
| Kobo Sage | 1440×1920 |
| Kobo Elipsa 2E | 1404×1872 |
| reMarkable 2 | 1404×1872 |
| reMarkable Paper Pro | 1620×2160 |

Not on the list, or a resolution changed in a firmware update? Use **Custom size** in the device dropdown. PRs adding/correcting devices are welcome.

### Crop modes

- **Center-crop to fill** — crops the edges to match your device's aspect ratio exactly, no distortion, no padding.
- **Fit with padding** — the whole photo stays visible, letterboxed to fill the target size. Padding can be white, black, or transparent (transparent is experimental — e-ink devices generally don't composite alpha, so on-device behavior is untested and may vary by model/firmware).
- **Manual position** — drag and zoom a crop frame locked to your device's aspect ratio. For batches, you step through each photo one at a time. The crop is stored as ratios and re-applied to the full-resolution original at export time, so preview downscaling never limits output quality.

### Remove background (optional, any crop mode)

A separate "Remove flat background (experimental)" toggle, independent of the padding-color option above — it acts on the photo's own content, in any crop mode, not on padding.

Static guidance is always shown under the checkbox, regardless of whether it's enabled:
- **Works best on:** plain, single-color backgrounds (studio backdrops, solid paper, flat vector art/illustrations) behind matte, opaque subjects.
- **May work poorly on:** subjects with glossy, translucent, or reflective parts (plastic wrap, glass, glare) — especially if they touch the image edge — as well as busy, textured, or multi-color backgrounds. Always check the preview before downloading.

Separately, once the toggle is enabled with photo(s) uploaded, a *dynamic* status line reports what fraction of this specific photo's border actually matches its detected background color — a rough "background solidity" hint (e.g. *"Background looks solid (96% of the border matches)"* vs *"Background doesn't look uniform (62% of the border matches) — results may be poor"*), recalculated live as the tolerance slider moves. It's advisory only — it won't block conversion either way. The two messages are independent: the static text sets expectations about subject/background type in general, the dynamic note reflects the actual uploaded image.

Under the hood: it detects the most common color along the photo's own border (bucketed to survive JPEG noise — real photos rarely have two pixels that are *exactly* identical, so a naive "most common exact color" wouldn't find a meaningful mode), then flood-fills inward from the border, turning matching pixels (within the tolerance slider) transparent. Same on-device caveat as transparent padding: e-ink devices generally don't composite alpha.

### Quality

- **No dither** (default) — flat, direct mapping to the nearest of 16 evenly-spaced grayscale levels.
- **Dithered** — Floyd-Steinberg error diffusion, reduces visible banding on gradients at the cost of a slightly noisier look.

---

## Third-party code (vendored, not CDN-loaded)

Both are inlined into `index.html` (see [Single file](#e-reader-screensaver-converter) above) so the tool has zero runtime network dependencies. Unmodified original copies are kept under `vendor/` purely for license/attribution reference — they aren't loaded by the page:

- [Cropper.js](https://github.com/fengyuanchen/cropperjs) v1.6.2 — MIT license — powers the manual crop UI.
- [JSZip](https://github.com/Stuk/jszip) v3.10.1 — MIT/GPLv3 dual license (used here under MIT) — bundles batch conversions into a single `.zip` download.

## License

MIT — see [`LICENSE`](LICENSE).
