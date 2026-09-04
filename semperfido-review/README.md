# Semper Fido Sweepstakes — games page, restyled

A restyling of the live `page2.html`. **Double-click `index.html`** — it opens
straight from disk, no server and no build step. **Nothing on this page touches
the network.** Fonts, images and the hero video are all local.

This is a review artifact, not production code.

---

## Same content. Nothing added, nothing removed.

Every visible string on this page also appears on the live page:

| | |
|---|---|
| `Semper Fido Sweepstakes` | nav brand |
| `Home` `How It Works` `Sweepstakes` `Mobile Play` `Join Now` | nav |
| `Premier Vendor for Phantom Software` | hero |
| `Easily enter with just a few clicks.` | from the marquee's `data-linewords` |
| `Exciting prizes await every lucky winner!` | ditto |
| `Current Sweepstakes Offerings` | grid heading |
| `Join The Fun!` | footer |
| `© 2026 Semper Fido Sweepstakes. All rights reserved.` | footer |

That's the whole page. The one string that isn't on the live page is a
**"Skip to games" link**, invisible until you tab into it — a keyboard
accessibility affordance rather than content.

---

## The top bar is unchanged

Values lifted off the live page and reproduced exactly:

| | |
|---|---|
| Background | `rgba(255, 255, 255, 0.8)` |
| Border radius | `1440px` |
| Shadow | `rgba(27, 31, 10, 0.08) 0 30px 60px 0` |
| Max width / height / offset | `1320px` / `90px` / `16px` from top |
| Type | Brygada 1918, `19.2px`, `#232323` |
| Button | `#b90e2c`, `19.2px`, `100px` radius |

Measured on the rebuilt page: 1320×90 at top 16, all values matching.

**One addition:** `backdrop-filter: blur(10px)`, so the bar stays legible over
the game art while scrolling. Delete that line and it is pixel-identical.

At 390px the brand and menu button don't fit on one line and the bar wrapped to
136px. The mark is smaller and the wordmark runs to two lines on mobile,
bringing it back to 72px.

---

## The hero video

### What went wrong before

The live page embeds a YouTube copy of this clip and lets Mobirise size the
player. Two separate faults:

**1. The player box is the wrong shape, and differently wrong at every width.**

| Viewport | Player iframe | Aspect | Video renders at | Covers hero? |
|---|---|---|---|---|
| 1440 | 5130 × 461 | **11.13** | 819 × 461 | **No — pillarboxed** |
| 1600 | 5706 × 1382 | 4.13 | 2458 × 1382 | yes |
| 1920 | 2286 × 1382 | 1.65 | 2286 × 1286 | yes |

YouTube fits a video inside whatever box it's handed and fills the rest with
black. An 11:1 box collapses the picture to a strip in the middle.

**2. The embed says `autoplay=1&mute=0`.** No browser autoplays unmuted video,
so it frequently never started at all.

### The seam was in the video file

Separately from the layout faults, the uploaded clip had a hard vertical edge
about 76% across. Measuring the YouTube version's poster frame column by
column: a brightness discontinuity of **11.54** between adjacent columns, where
the next largest anywhere in the frame was **1.39**.

That was a composite seam baked into the footage, not a layout problem — the
result of reframing a **1600 × 2100 portrait** master into a 16:9 landscape by
extending the picture sideways to fill the width.

### How this version does it

Cropped straight out of the original master with ffmpeg — no reframing, no
stretching, nothing extended:

```
ffmpeg -i LobbyBackgroundDesktop.webm \
  -vf "crop=1600:900:0:1200" \
  -c:v libvpx-vp9 -crf 28 -b:v 0 -an out.webm
```

The `PHANTOM` logo occupies **y = 128–280** of the 2100px master. The crop
takes the bottom **1600 × 900** (y 1200–2100) — a true 16:9 region, 920px clear
of the logo, and the best-looking part of the frame: blue nebula left, warm
glow right.

Verified:

| | Original master | Broken YouTube version | This crop |
|---|---|---|---|
| Size | 1600 × 2100 | 1920 × 1080 | 1600 × 900 |
| Worst adjacent-column jump | 3.34 | **11.54** | **2.09** |
| Frames | 313 | — | **313** |
| Duration | 13.042s | — | **13.042s** |

2.09 is below the master's own natural variation of 3.34, so no edge was
introduced. Every frame is preserved.

In the page it's a plain `<video autoplay muted loop playsinline>` with
`object-fit: cover`. The browser crops it and never stretches it, so all the
aspect-ratio arithmetic the YouTube embed needed is gone — along with the
embed, the autoplay gate, the play-button fallback and the network request.
The poster is the clip's own first frame, so the hero looks identical before
the video decodes. `prefers-reduced-motion: reduce` pauses it and leaves the
poster.

### The fold

At 1440×780, roughly your monitor once Chrome's chrome is subtracted:

- 6 columns, 12 games, **exactly 2 rows** (`grid-template-rows: 275px 275px`)
- Row one occupies **477 → 752px, entirely above the fold**, 28px to spare

---

## What else is different

Presentation only.

| | Live | Here |
|---|---|---|
| Marquee | Scrolls past | Same two sentences, set once, still |
| Hero | Full height (~700px before any game) | 330px |
| Tiles | 4-across, full-bleed (~450×675 on a 1440px monitor) | 6-across, 190×285 |
| Type | Brygada 1918 at 80px for nearly everything | Brygada 1918 at 52 / 30 / 19.2 / 17 / 13 |

Same typeface the site already uses — the problem was never the face, it was
one size and one weight doing every job.

| Viewport | Columns |
|---|---|
| < 560px | 2 |
| ≥ 560px | 3 |
| ≥ 768px | 4 |
| ≥ 1024px | 5 |
| ≥ 1200px | 6 |

---

## Checks

Verified in a browser at 390, 768, 1024, 1440 and 1920px:

- No horizontal overflow at any width
- Hero video plays, loops, muted, covers the band exactly at every width
- Mobile menu opens and closes with correct `aria-expanded`; page content goes
  `inert` while open, so focus can't land behind the panel
- Menu targets 55–58px; toggle 48×48
- Console clean; **zero network requests**
- Every asset reference resolves

Tiles are bare art, as on the live page, each carrying the game name in `alt` —
the live tiles have no alt text at all, so they're invisible to screen readers
and search engines today.

### One non-visual bug on the live site

`page2.html` answers **`HTTP 405 Method Not Allowed` to a `HEAD` request**
(`GET` returns 200). Link checkers, crawlers and SEO tools probe with `HEAD`
first and will report the page as broken. Worth passing to whoever runs nginx.

### And one that's easy to miss

The marquee's markup still contains Mobirise's demo text — `☁️ Best offers
☁️ Free delivery ☁️ Vibes`. The real copy lives in `data-linewords` and only
swaps in once JS runs, so the demo text is what renders with JS disabled and
what a crawler reads.

---

## Files

```
index.html                    the page — open this
README.md                     this file
fonts.css                     @font-face rules pointing at assets/fonts/
                              (index.html inlines these as data URIs instead)
assets/
  hero.webm                   cropped background video, 1600×900, 13.042s
  hero-poster.jpg             its first frame, used as the poster
  sfslogo.png                 site logo
  games/*.png                 12 game tiles
  fonts/*.woff2               Brygada 1918, latin + latin-ext
```

Every link in the top bar is an absolute URL to semperfidosweepstakes.com, so
the navigation works wherever this is hosted rather than 404ing. All five were
checked and return 200, and the Join Now anchor exists on `aboutsfs.html`. The
12 game tiles point at `page2.html` on the live site. The two social icons are
still `href="#"` — they are `href="#"` on the live site too, and there are no
real profile URLs to use.

The page carries `<meta name="robots" content="noindex, nofollow">` so a public
review copy cannot be mistaken for the real site in search results. Remove it if
this ever becomes the real thing.

## Asset provenance

Logo and 12 game tiles downloaded from the live site's CDN
(`r.mobirisesite.com/3154168/assets/images/`), unmodified; filenames shortened.
`hero.webm` is cropped from your own `LobbyBackgroundDesktop.webm` master. All
of it belongs to whoever owns it.

Brygada 1918 is from Google Fonts under the SIL Open Font License 1.1 — the same
face the live site loads. Only latin and latin-ext are included, as one variable
face each covering weights 400–700: 55 KB rather than the 482 KB that Google's
four separate weight declarations would have embedded.

Facebook and Instagram marks are their owners' trademarks, drawn as inline SVG.

## Colors

| Token | Value | Source |
|---|---|---|
| `--brand-gold` | `#f8961e` | the live hero heading |
| `--brand-red` | `#b90e2c` | the live Join Now button |
| `--bar-bg` | `rgba(255,255,255,.8)` | the live top bar |
| `--bar-ink` | `#232323` | the live nav text |
| `--ink-900` | `#12100E` | page ground |
| `--ink-850` | `#17140F` | banner and footer bands |
| `--ink-800` | `#1C1917` | tile surface |
| `--ink-700` | `#2A2624` | borders |
| `--text-hi` | `#F5F2ED` | headings |
| `--text-lo` | `#A8A29B` | banner text |
| `--text-xlo` | `#8A837A` | copyright — 5.25:1 on `#0D0B09`, passes AA |

The brand gold and red are the live site's own values, not substitutes.

---

## Hosting on GitHub Pages

Push the **contents** of this folder to your repo (root, or a `/docs` folder,
then point Pages at it). Nothing needs building.

`.nojekyll` is included. Without it GitHub runs Jekyll over the site, which
silently skips any file or folder whose name starts with an underscore — a
classic way to lose assets on Pages.

Two things to know:

- **The three sibling nav links will 404** until those pages exist next to this
  one: `aboutsfs.html`, `page2.html`, `page3.html`. They keep the live site's
  real filenames so this drops straight into the real site. If you want the
  demo to be click-safe instead, point them at `#`.
- **Paths are all relative and lowercase**, so it works from a project subpath
  (`username.github.io/repo/`) and on a case-sensitive server. Don't rename the
  `assets/` folder or its files.
