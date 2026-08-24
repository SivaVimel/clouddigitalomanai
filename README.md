# clouddigitalomanai

**Oman Hosted Cloud AI** — the landing page for single-tenant LLM inference
hosted in Oman. A single-page, scroll-scrubbed film.

Open `index.html`, or serve this repo with GitHub Pages.

## Enabling Pages

Settings → Pages → Build and deployment → **Deploy from a branch** → `main` / `/ (root)`.
Nothing to build; the site is static and every path is relative, so it works on a
project page (`sivavimel.github.io/clouddigitalomanai/`) as well as a custom domain.

## What is here

```
index.html                 the whole page — markup, styles and the timeline engine
fonts/fonts-inline.css     Inter + JetBrains Mono, latin subset, embedded as data URIs
images/                    the mark, and six plates generated for this page
audit.py                   layout regression test (see below)
.nojekyll                  skip the Jekyll build step
```

11 files, ~1.1 MB. No dependencies, no build step, no server config.

## The story it tells

| Beat | Device |
|---|---|
| Title | the lockup holds, then blurs into the film |
| The bills | six months dealt out as a fan, no two the same |
| The two lines | a metered bill claws upward; ours is a straight line |
| Data residency | a boundary closes over the coast, packets pulled back inside |
| Shared vs yours | the frame splits — a crowd of tenants against one block |
| The price | 50 OMR, a month, flat |
| Drop-in | a diff: one line struck out, one line written in |
| "Unlimited" | what is uncapped, beside what is not |
| One endpoint | every client docks onto the same spine |
| Close | Contact Us, and a link to the desktop application |

The **"unlimited"** section is deliberately even-handed: no request cap, no token
cap, no metering — set beside *not* unlimited throughput, concurrent requests
queue, and capacity is one real machine rather than an autoscaling pool. If the
copy changes, keep both halves; the balance is the point.

## How it works

There is no video and no CSS animation. The page is one continuous timeline that
the reader scrubs with the scroll wheel:

```
T = section index + that section's progress      // one global clock
draw(T)  →  every stage sets its own inline styles
```

Each `<section>` is a spacer that supplies scroll distance; each `.stage` is
`position: fixed` and cross-dissolves with its neighbours, so sections hand over
the way a cut does. `draw()` is a pure function of scroll position, so the film
runs forwards, backwards, and at any speed it is flung.

Headlines arrive a word at a time while the line re-centres around them. Long
lines wrap into rows that each re-centre on their own, which is how a phone gets
the identical move rather than a shrunken one.

## Checking a change

```bash
python3 -m http.server 8899
AUDIT_PAGE=index.html python3 audit.py
```

Walks the whole reel at 15 viewports — 320 px to 2560 px, portrait and landscape —
firing the height-only resize a phone produces when its address bar slides, and
asserts that no two words of a line overlap and that no anchored text leaves the
frame. Exits non-zero on failure.

Three invariants it protects, each of which broke once:

1. **Never measure a hidden element.** A span inside a `display:none` stage
   measures as zero width, and zero widths put every word of a line on the same
   point. Lines re-measure lazily at draw time, when the stage is on screen by
   definition.
2. **A phone's address bar fires resize constantly.** Height does not affect text
   metrics, so a height-only change only redraws; width changes and rotations
   re-measure.
3. **Measure text, not its box.** Most headings are full-width absolutely
   positioned centring boxes, so their own rect is the frame.

Panels sized against a *pane* rather than the viewport follow the same rule — the
split-screen beat sizes its tiles from the half-frame they live in, or they
collapse to their floor on a phone.
