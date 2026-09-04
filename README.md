# LONG HAUL

Nokia-era Snake, reskinned as a freight train.

You drive a locomotive around a 20×16 yard. Shipping containers appear one at a
time — box, open top, tank, reefer — and driving over one couples a flat wagon
to the back of your train with that exact container sitting on it. The train
only gets longer. Cross your own line and you derail.

One HTML file. No build step, no dependencies, no external assets, no network
calls. Open it and it runs.

---

## Controls

| | |
|---|---|
| **Arrow keys** / **WASD** | Steer |
| **Swipe** | Steer (touch) |
| **On-screen d-pad** | Steer (touch devices only) |
| **Space** / **Esc** | Pause and resume |
| **M** | Mute |
| **T** | Toggle tunnels (title screen) |
| **Any key** | Start, resume, restart |

The sound and pause icons in the header are tap targets, so everything is
reachable on a phone without a keyboard. They are pixel art like everything
else: a nine-pixel plate with its corner pixels left out, and a nine-pixel
glyph blitted through the same routine that draws the rolling stock.

**Tunnels** is the wrap-around mode. Off (default), the walls are fatal and the
board is drawn with a solid border. On, the edges wrap and the border is drawn
dashed, so you can always tell which rules you are playing under. The setting,
the mute state and your high score persist in `localStorage`.

## Scoring

- Container: **1 point**, **1 wagon**.
- Gold container (1 in 10): **3 points**, **2 wagons**.
- The train starts at 6 ticks/sec and speeds up 4% every 5 containers, capped at
  14 ticks/sec.

## Run it

```
open index.html
```

That is the whole thing — double-click the file, or drag it into a browser.
There is nothing to install and nothing to serve.

If you would rather serve it over HTTP (for `cleanUrls`, or to test the social
preview), any static server works:

```
python3 -m http.server 8000     # then visit http://localhost:8000
npx serve .
```

## Deploy

The repo is a plain static site, so any static host will do.

**Vercel**

```
npx vercel            # preview
npx vercel --prod     # production
```

`vercel.json` sets `cleanUrls`. No framework preset, no build command, no output
directory — Vercel serves the root as-is.

**Anything else** — Netlify, GitHub Pages, Cloudflare Pages, S3: point it at the
repo root. There is no build step to configure.

After deploying, one optional touch: `og:image` and `twitter:image` are relative
paths so the file also works when opened directly from disk. Most scrapers
resolve them against the page URL, but if you want to be certain, swap them for
the absolute `https://your-domain/og-image.png`.

## Files

```
index.html     the entire game — markup, CSS and JS inline
favicon.svg    locomotive icon
og-image.png   1200×630 social preview
vercel.json    { "cleanUrls": true }
```

## How it works

A few notes for anyone reading the source.

- **Two clocks.** The simulation is a fixed tick (6–14/sec); rendering is a
  single `requestAnimationFrame` loop at display rate. The renderer interpolates
  each car between its previous and current cell, so the train glides instead of
  snapping. Frame delta is clamped at 250 ms so a backgrounded tab cannot
  fast-forward the run on return.
- **The train is two arrays.** `cells` holds positions, head first. `wagons`
  holds colours indexed by position behind the locomotive — the *n*th container
  you haul becomes the *n*th wagon — so the colours stay put as the train moves.
- **Input buffer.** Turns queue two deep and apply one per tick, and a
  180° reversal is rejected against the last *queued* direction rather than the
  current one. A fast double-tap around a corner cannot kill you.
- **Wrap seams.** In tunnel mode a car crossing an edge is drawn twice — leaving
  one side and entering the other — so it slides through instead of teleporting.
- **Two canvases.** `#bg` is pinned to the viewport and painted only on resize:
  meadow, treeline, villages, and woodland closing in from every edge,
  generated from a fixed seed so a given screen always grows the same wood. The
  game canvas is transparent outside its own board, so the countryside is the
  whole background and the yard reads as a clearing cut into it — no seam.
- **Honest pixels.** The countryside is drawn on a small offscreen canvas — one
  art pixel is one pixel there, five to seven screen pixels wide — and then
  blown up with `imageSmoothingEnabled = false`. Trees, cottages, the church,
  fences, hay and sheep are sprites written as arrays of strings, one character
  per pixel, indexed into a shared palette; the foliage letters get repainted
  through two extra palettes so some of the wood is old and dark and a few
  trees are turning. There are no gradients: bands of green are stepped and the
  seams (and the vignette) are 4×4 ordered dither.
- **The yard is baked the same way.** The ground inside the fence is a second
  offscreen canvas, one art pixel to three logical ones, built once out of the
  same vocabulary as the wood outside it: mown greens in soft patches with the
  dither at their seams, grass flecks, a few wildflowers, ground worn bare at
  the edges, and sprite clutter in the four corners — those four cells are
  struck off the spawn list so cargo never lands inside a bush. It costs one
  `drawImage` a frame.
- **The permanent way, on the title screen only.** The idle train laps the
  border ring, so there is a closed loop of track under it: gravel bed,
  sleepers pitched by arc length, two rails on top. It is baked once into a
  third offscreen canvas the same way the yard is — a distance field to the
  ring's centreline, evaluated once per art pixel — rather than stroked as a
  polyline, so the ballast, the sleepers and the rails are made of the same
  square pixels as the grass, and the corners come out as honest pixel arcs
  instead of round line caps. The widths are picked to land on whole art-pixel
  rows: seven rows of bed with two more frayed out through the Bayer dither,
  sleepers across the same seven, and a rail on the second row in from each
  end of them, which leaves a sleeper showing outside each rail — the
  difference between track and a zip fastener. The sleeper pitch is trimmed to
  an exact divisor of the ring's perimeter so the loop closes on itself. The
  playfield has no track: during a run the train makes its own line, and rail
  under a snake that can turn anywhere would be a lie.
- **Eight headings, not four.** A car used to be drawn only square to the grid,
  so a ninety-degree corner was a single frame's flip — the hinge that made the
  turns look wrong. There is a diagonal for each quarter now: the art is
  rotated forty-five degrees at *art-pixel* resolution and blown up by whole
  pixels afterwards, and the alpha is snapped back to 0 or 255 because a
  rotated edge is the one thing canvas still antialiases with image smoothing
  off. A car rounding a corner therefore spends a beat on the diagonal, which
  is about a tick at base speed.
- **Steam.** A wisp off the chimney: a couple of puffs a tick, drifting back
  over the train and rising a little, gone in about two thirds of a second.
  It runs on the title screen too, because a train that is standing still in
  the corner of your eye should still be under steam.
- **One container routine.** The box standing in the yard and the box riding on
  a wagon are the same function, so what you drove over is visibly what you are
  now hauling. Every wagon is a flat: solebar, planked deck, two bogies, and a
  container on top — corner castings, fine corrugation across the roof, doors
  at one end. The two starter cars run empty, which is what an empty flat looks
  like.
- **Scaling.** The canvas is a fixed 500×438 logical surface. JS measures the
  container, sets the canvas CSS size to fit, and sizes the backing store to
  `devicePixelRatio` so it stays sharp on phones instead of resampling.
- **Audio** is a handful of WebAudio oscillators built on demand after the first
  input; there are no audio files. `localStorage` access is wrapped in
  `try/catch` and falls back to memory, because some browsers deny it on
  `file://` origins.
