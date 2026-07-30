# hawthorneracing.com.au

Static one-page site for Hawthorne Racing, hosted on GitHub Pages at
[hawthorneracing.com.au](https://hawthorneracing.com.au).

Hawthorne Racing is a demonstration brand. Content on the site is fictional.

## Contents

| File | Purpose |
|------|---------|
| `index.html` | The entire site: hero, a short team section, and the three partner cards. Fonts, the Hawthorne mark, and all three partner logos are base64-embedded, so the only external request the page makes is the hero photo. No build step. |
| `hawthorne-car-*.jpg` | Hero photo at five widths (480/700/1000/1400/1800), served via `srcset`. The one asset deliberately *not* inlined — see below. |
| `404.html` | Branded not-found page. GitHub Pages serves this automatically. |
| `CNAME` | Custom domain for Pages. Must contain exactly `hawthorneracing.com.au`. |
| `.nojekyll` | Tells Pages to serve files as-is instead of running them through Jekyll. |
| `robots.txt` | Allows all crawlers, points to the sitemap. |
| `sitemap.xml` | Single-URL sitemap. Update `lastmod` when the page changes. |

## First-time deploy

**1. Create the repo**

Create a repository named `hawthorneracing.com.au` on GitHub. Public is required
for Pages on a free plan; private works on Pro and above.

**2. Push these files to the repo root**

```bash
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin git@github.com:<your-account>/hawthorneracing.com.au.git
git push -u origin main
```

Or use **Add file → Upload files** in the GitHub web UI and drag the files in.
If you upload via the browser, note that `.nojekyll` is a dotfile and some
browsers hide it; if it does not appear in the repo, create it with
**Add file → Create new file**, name it `.nojekyll`, and leave it empty.

**3. Turn on Pages**

Repo **Settings → Pages**:

- Source: **Deploy from a branch**
- Branch: `main`, folder `/ (root)`
- Save

The first build takes a minute or two. The site will appear at
`https://<your-account>.github.io/hawthorneracing.com.au/` before the custom
domain is live.

**4. Set the custom domain**

Still in **Settings → Pages**, under **Custom domain**, enter
`hawthorneracing.com.au` and save. GitHub will commit or overwrite the `CNAME`
file to match, which is expected.

**5. Add the DNS records**

At the DNS provider for `hawthorneracing.com.au`, create four A records for the
apex (host `@`):

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Optionally add the matching AAAA records for IPv6:

```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

If the provider supports ALIAS or ANAME at the apex, point it to
`<your-account>.github.io` instead of using A records. That survives future
GitHub IP changes.

For the `www` variant, add a CNAME record: host `www`, value
`<your-account>.github.io`. Set up both apex and `www` before GitHub issues the
certificate, otherwise the certificate will only cover whichever was live first.

**6. Enforce HTTPS**

Once DNS resolves, return to **Settings → Pages** and tick **Enforce HTTPS**.
The certificate can take up to an hour to issue. If it stalls, remove the custom
domain, save, re-add it, and save again to force reissue.

## Making changes

Edit `index.html` and push. Pages redeploys on every push to `main`, usually
within a minute. Hard-refresh to get past browser caching.

The page is one file with the CSS in a `<style>` block at the top and a small
script at the bottom. Brand tokens live in the `:root` block, so palette changes
happen in one place.

## Notes

- `hawthorneracing.com.au` is canonical. The contact address on the site is
  `info@hawthorneracing.com.au`, matching the hosting domain rather than the
  `hawthorneracing.com` domain used elsewhere in the brand system.
- The header and the sign-off both use the icon mark plus the team name set in
  Barlow Condensed. The supplied horizontal lockup was dropped: it is a raster
  with a non-uniform dark background, so it never sat flush on the page and it
  went soft below roughly 70px tall. If an SVG lockup becomes available, either
  spot can use it directly at any size.

## Hero photo

The hero photo is an AI-generated image (Gemini), cropped 1.25:1 to keep the front
wing intact and re-encoded as JPEG. It carries all three partner decals.

It is a separate file rather than base64 in `:root` like the logos, on purpose. An
inline photo would sit in the critical path and block HTML parsing until it
finished downloading, and it could not be cached independently of the page. The
logos are small enough that inlining wins; a photo is not.

### Renditions

Five widths, all the same crop, generated from the original 2276x1856 source:

| Width | Quality | Size |
|-------|---------|------|
| 480 | 84 | 45KB |
| 700 | 84 | 82KB |
| 1000 | 82 | 133KB |
| 1400 | 82 | 218KB |
| 1800 | 82 | 318KB |

The two small ones use quality 84 because heavy downscaling hides JPEG artefacts
less effectively at small pixel dimensions.

Selection is `srcset` with width descriptors plus `sizes`, **not** `<picture>`
with `media`. Every rendition is the same crop, so this is resolution switching,
and `srcset` is what responds to device pixel ratio. `<picture>` is for art
direction — reach for it only if you want a genuinely different crop per
breakpoint. Verified picks: 480 at 485px/1x, 700 at 723px/1x, 1000 at 1403px/1x
and at 474px/2x, 1800 at 1403px/2x.

If you change the layout widths, update `sizes` to match or the browser will pick
the wrong rendition. It currently reads:

```
(max-width:420px) calc(100vw - 40px), (max-width:980px) calc(100vw - 56px), 55vw
```

### Two treatments, one <img>

A single `<img>` serves both so `srcset` applies to each. Above 980px
`.hero-figure` is lifted out of flow to become the angled right half; below 980px
it drops back into flow, stacked under the copy.

Four things that are easy to break:

- The figure is a child of `.hero`, **not** `.hero-inner`. It has to be, or being
  absolutely positioned inside the wrap would stop it bleeding to the section
  edge.
- It is inset to `left:45%`, **not** stretched across the whole hero. At full
  width, `object-fit:cover` scales the car across the entire section and the
  revealed slice starts behind the front wheel. Size the layer to the region you
  actually reveal.
- The angled cut is `clip-path:polygon(0 100%, 220px 0, 100% 0, 100% 100%)`. The
  lean is a fixed **220px**, not a percentage, so the cut holds the same angle at
  every viewport width. Percentages make it shallower as the window widens.
- Layering is explicit: figure `z-index:1`, `.motif` `z-index:2`, `.hero-inner`
  `z-index:3`. The motif needs its own z-index or it falls behind the photo. On
  mobile the figure switches to `position:relative` with `z-index:3` — z-index
  does nothing on a static element, and without it the motif paints over the
  photo.

The `::after` scrim darkens the photo toward the cut. Without it the faint
`#F2F4F5` racing line and the dot field disappear into the image, and the copy
gets close to the bright grandstand at narrow desktop widths. It is switched off
on mobile, where the photo is a bordered block instead.

The hero is a two-column grid above 980px and stacks below it. Both tracks are
`minmax(0,...)` and both children carry `min-width:0` — without that, the
`clamp()`-sized `h1` can push its `auto` grid minimum wider than the viewport and
cause horizontal scrolling on narrow phones. Above 980px the second track is
empty by design; it reserves the space the photo shows through.

## Partner logos

All three partners use their real logo, each embedded as a transparent PNG in
`:root` so the page still makes no external requests. No source file arrived with
an alpha channel, so each needed a different extraction:

- **Ketsudan** (`--ketsudan`) — from `2.png` in the Ketsudan brand folder, white
  lockup on flat `#2367B2`. Alpha was keyed off the **red** channel, which
  separates that blue from white across 35→255; green and blue give a much
  narrower range and a dirtier edge. Ignore `Ketsudan Logo - Full.PNG`: it is blue
  artwork on opaque white, and blue on this charcoal is about 2.6:1 contrast.
- **Arbitr** (`--arbitr`) — derived from `logo-white-horizontal` in the Arbitr
  brand assets. Those source files are JPEGs on a black background, so the white
  artwork was converted to luminance-as-alpha to sit cleanly on the dark section.
- **AIMSPIndex** (`--aimspi`) — the retro CRT mark. Supplied opaque on `#1A1A1A`,
  so the dark surround was flood-filled from the outside corners to transparent.
  Flood fill rather than a colour key, because the screen interior is also near
  black and must stay: the beige bezel encloses it, so the fill cannot reach it.

Ketsudan and Arbitr share the same brand blue, so both render as white line art
and the CRT is the row's only colour. If you swap one back to a colour treatment,
expect it to compete with the CRT.

Two layout constraints worth knowing before editing this section:

- `.partner-plate` is **112px** tall, sized to the CRT rather than to the two
  horizontal marks. The CRT is nearly square (420x336), and anything shorter
  renders its screen text too small to read; Ketsudan and Arbitr centre in the
  extra space. Shrinking the plate will make the AIMSPIndex logo illegible.
- The Hawthorne wordmark in the nav and sign-off cannot wrap, which is what the
  `max-width:420px` block in the CSS is for. Layout is verified free of
  horizontal overflow from 320px to 1400px.
