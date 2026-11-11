# Fast Path spec (v1)

Status: approved by the owner on 2026-11-11 (real 3D like pinball, defaults for the rest).
Red-teamed twice with Fable (findings and fixes at the end). Nothing is built yet: builds take
their turn in the owner's one-at-a-time queue.

## What it is

A single-player 3D racing game for `fast-path.prignano.ai`. You drive a small car on a real 3D
track, seen by a camera that follows behind you, against five computer rivals over three laps.
There is no online play: every race is you against the bots, on your own device, offline.

Pick a track and a level (Easy, Normal or Hard), race, then **Race again** or **Next track**.
Your best race time and best lap on each track are kept on the device.

It's plain JavaScript with [three.js](https://github.com/mrdoob/three.js) (MIT, vendored the
way pinball does) and no build step, served as static files. It installs as an app (PWA) on
iPhone, iPad and Android from day one, plays with touch or a keyboard, and is made for all
ages: kids will play it.

## Decisions so far

| # | Question | Answer |
| --- | --- | --- |
| 1 | Kind of game | A racing game against bots only, no online play, single player (owner, 2026-11-10, via the game queue) |
| 2 | The view | **Real 3D like pinball**: three.js with WebGL 2, low-poly models drawn in code (owner, 2026-11-11) |
| 3 | Name | **Fast Path**, repo `nickprignano/fast-path` (owner took the default, 2026-11-11; he created the repo the same day). Checked 2026-11-11: no racing game by that name on the stores or itch.io; a small itch.io game titled "Fast Path" (genre not stated); the one "FASTPATH" US trademark checked is cancelled and was for logistics software. A close match in the same genre goes back to the owner, with **Lookahead** as the fallback |
| 4 | Twist | Rivals named after their driving rule (owner took the default, 2026-11-11) |
| 5 | Touch | Screen halves: hold left or right to steer, both to brake, auto-gas (owner took the default, 2026-11-11) |
| 6 | Name, art and sound | Original. No names, cars, tracks, art, music or sounds from any commercial racing game. The cars are not modeled on any real car or any game's car: no badges or grilles |
| 7 | Home | Its own public repo under `nickprignano`, served at a `prignano.ai` subdomain, deployed by Workers Builds on each merge to `main`. The owner creates the repo and adds the domain. Launch ends with the `content/games.json` PR in `prignanodotai-web` |
| 8 | Players' data | Stays on the device. No accounts, names, analytics or network calls beyond the pinned theme header |
| 9 | Physics | Our own small sim in track coordinates (distance along the track's centre line and offset across it), as pinball keeps its 2D physics under the 3D view. A car's speed is its **true ground speed**: off the centre line, its distance along the track advances by speed × time ÷ (1 − offset × curvature), so the inside line really is shorter. Flat cross-section (no banking); heights are drawn from the track's shape, not simulated |
| - | License | MIT. Our own code, art and sound. three.js keeps its MIT notice in `vendor/three/LICENSE`. `tools/sw-version.mjs` and the PWA recipe come from Vibes (MIT). From pinball (GPL-3.0-only) only `tools/vendor-three.mjs`, `tools/check-3d.mjs` and the three.js renderer setup in `src/three/view.js` come across: they're generic three.js and Playwright code with no Vector Pinball code in them, written for the owner. A small pinball PR records in its README that those pieces are the owner's original work, also offered under MIT, so the two repos don't contradict each other. javascript-racer's MIT notice ships in `THIRD_PARTY_NOTICES.md` anyway (see Prior art) |

## Prior art and licenses

Checked 2026-11-10 and 2026-11-11 (web search, GitHub, and our own KB prior-art script, which
found nothing in the KB).

| Source | License | What we take |
| --- | --- | --- |
| [three.js](https://github.com/mrdoob/three.js) r187 (`three@0.187.0`) | MIT | The renderer (`three.core.js`, `three.module.js`). No add-ons |
| Pinball's 3D upgrade (`docs/SPEC-3D.md`, `tools/vendor-three.mjs`, `tools/check-3d.mjs`, the renderer setup in `src/three/view.js`) | ours | The vendoring tool and its pinned hash, the pixel-ratio cap, `compileAsync`, dispose on change, the context-lost rule, the software-WebGL browser check, and its lessons (below). Not its post-processing files, camera fit, table build or palette code |
| Vibes | ours, MIT | `tools/sw-version.mjs` and its PWA files |
| The pseudo-3D road technique and Louis Gorenfeld's "Lou's Pseudo 3d Page" | article | Track as sections with curve and climb; the outward push in curves. Credited in About |
| [jakesgordon/javascript-racer](https://github.com/jakesgordon/javascript-racer) | MIT (code) | Ideas only, now fewer since we draw in real 3D. Its sprites are placeholders its README says were borrowed from a commercial console racer, and its music is licensed to that project only: we take neither. We ship its MIT notice anyway |
| Other open-source browser racers (three.js and top-down; many unlicensed) | mixed | Nothing |
| "Race Condition", a retro racer on Steam (2022) | commercial | Not a source: it's why that name was out |
| Infinite Recursion and the ai-brain page "Tuning an agent-built game by simulation" | ours | Seeded, pure simulation; fairness tests with scripted drivers and a band floor; simulate the numbers before building on them |
| prignano-theme `art/flash.js` and `tools/flash-scan.mjs` | ours, MIT | The flash counter (the caller bins frames into cells) and the frame-stepping scan approach |

Lessons from pinball's 3D build that this spec builds in: never hide an instanced mesh by
scaling it to zero (NaN normals blank the frame; move it away); a disposed view ignores its own
context-lost event; vendor with relative imports and a pinned npm hash; headless Chromium with
software WebGL is good enough for blank-frame checks.

## How a race looks and feels

### The look

- Low-poly and flat-shaded, in each track's own palette built from theme hues: a smooth
  gradient sky, soft haze toward the horizon, and rolling hills around the track.
- **The ground isn't lit at render time.** Road, verge and terrain use vertex colors with their
  shading baked in by `src/track.js`, so a Node test can hold neighboring facets within the
  flash caps (Flash safety). One sun and a soft ambient light shade only the cars and roadside
  objects. No tone mapping: colors go straight to sRGB output, so what the tests check is what
  the screen shows.
- The cars are original small, rounded coupes built in code (a hard top, no faces or eyes, not
  modeled on any real car), each in one matte-to-satin paint color with its own roof mark, so
  color is never the only cue. Wheels turn and steer; the body leans a little in curves; the
  drawn heading includes the car's sideways slide, so a car steering across the road doesn't
  crab.
- **What tells you you're fast:** the ground's soft, non-repeating texture streaming past,
  roadside rocks and posts, the engine's pitch, and the hills sliding past in curves. The owner
  judges it in build 1.
- Scenery is instanced low-poly models: low rocks and dunes, trees and posts set well back,
  boards with arrows before bends, a lap gantry drawn as a plain beam with the lap board.
- **Not this look:** no palm trees on a beach road, no red open-top car, no striped sunset sky,
  no checkered start arch.
- The race scene uses its track's palette in light and dark mode alike (as pinball keeps its 3D
  table in a dark room); menus, buttons and dialogs follow the site theme.

### Camera

- A chase camera 6 m behind and 2.5 m above your car, looking at the road 15 m ahead along the
  track, so it turns into curves before the car does. Vertical field of view 55 degrees in
  landscape; in portrait 70 degrees, with the camera 8 m back and 3.5 m up so the road ahead
  stays in view at 390 px wide. Near plane 1 m, far plane 1,000 m.
- It eases toward its target and never shakes. It is a pure step (previous pose, car state,
  fixed time step) in `src/camera.js`, run in sim time, so tests and the flash check use the
  same camera as the game.
- The sim runs at 60 steps a second; the view draws a blend of the last two sim states, so 90
  and 120 Hz screens stay smooth.

### Tracks

Three tracks, each a closed loop of 1.8 to 2.2 km, so a good lap takes about 35 to 45 seconds
and a race about 2 minutes:

| Track | Feel | Palette |
| --- | --- | --- |
| **Shoreline** | Wide road, long sweeping bends, gentle rises, low dunes and rocks. The first track, easy to learn | Day: sea blue, sand, pale sky |
| **Ridge** | Narrower, real hills with crests, one tight hairpin | Dusk: violet sky, warm hills |
| **Night Loop** | A city-edge road with lamp posts set well back, two chicanes, a long straight | Night: deep blue, a painted far skyline |

**How a track is built** (`src/track.js`, our own code, pure and tested in Node without
three.js):

- Each track is plain data (`data/tracks.js`): a loop of control points (position and height).
  A closed uniform cubic B-spline through them gives the centre line, so every track closes on
  itself and its curvature changes smoothly (a Catmull-Rom spline would jump in curvature at
  every point).
- A sampled arc-length table turns the spline into stations every 2 m. At each station the
  across direction is world-up × tangent (no curvature-based frame, which flips between bends).
- The road is 12 m wide with a 6 m drivable verge each side. The tightest centre-line radius is
  30 m, so the drivable ribbon (12 m from the centre) never folds and 1 − offset × curvature
  stays above 0.6.
- Crests have a vertical radius of at least 370 m, so a car at top speed stays under 1 g and
  doesn't look glued to the road.
- **Line of sight:** every curve whose limit speed is below top speed has its entry, or its
  arrow board, in view over the road's profile from at least braking distance plus 0.5 s of
  reaction at the speed reachable there. No tight bend hides behind a crest.
- **Terrain meets the road** the usual way: the road-and-verge ribbon is one mesh; the terrain
  is a grid whose heights blend into the road's near it, sit a little below under the ribbon,
  with the ribbon's outer edge dipped under the grid. Within 30 m of the road edge, terrain
  stays no higher than 1 m above the road, so nothing hides your car or cuts into the camera.
- Each curve's **limit speed** comes from the centre line's curvature.
- No bridges or crossings: no two stations more than 200 m apart along the track come within
  30 m of each other.
- Night Loop's skyline is a mipmapped texture on a far backdrop that turns with the world but
  never shifts, with windows at least 3 px tall at 390 px wide and within the contrast caps
  against the sky.

### Your car

Arcade handling in our own units, all in `data/tuning.json`: top speed 60 m/s on the flat, the
sim at a fixed 60 steps a second.

- Gas speeds you up toward top speed; letting go coasts down; the brake slows you fast.
- In a curve you drift outward in proportion to your speed squared and the curve's tightness,
  so you steer into it and slow down for the tight ones. Every curve has a **limit speed**: the
  fastest a car on the centre line can hold it at full steering.
- Climbing slows you a little and going downhill speeds you up a little, and a push from behind
  adds a little; no car ever goes over 66 m/s, the **highest reachable speed** the flash rules
  use.
- Off the road (on the verge) your top speed halves until you're back on.
- Hitting a low rock or dune drops you to a crawl and nudges you back toward the road. Trees,
  posts and lamp posts stand at least 10 m beyond the verge and can't be reached. There are no
  crashes, spins, flips, damage or wrecks. Cars never leave the ground.
- Bumping a car from behind drops you to just under its speed and pushes you a little sideways.
  Being bumped from behind gives a small push. No car can pass through another.
- **Race end:** the race ends 30 seconds after the winner crosses the line. Cars still racing
  get their current place.

### Rivals (the twist)

The five rivals are named after the driving rule each one follows, and each has a one-line
card on the starting grid, so a player learns what makes a good racing line by watching them:

| Rival | How it drives |
| --- | --- |
| **Greedy** | Takes curves faster than they allow. Quick on straights, runs wide in tight turns |
| **Steady** | Never makes a mistake, never takes a risk. Slower at the top |
| **Drafter** | Follows the car ahead closely, then pulls out to pass |
| **Wobble** | A little unsteady on its line. The friendliest rival |
| **Lookahead** | Plans its braking far ahead and takes the inside line. The one to beat |

All five use one driver (`src/drivers.js`): pick a target speed from the tightest curve within
its lookahead distance, follow the track's racing line (inside of each curve, which is really
shorter), and move to the freer side when a slower car is ahead. Each personality is a few
numbers on that driver: top speed factor, corner factor (a share of each curve's limit speed),
lookahead, line noise and follow distance. Greedy's mistakes come from its corner factor being
above 1, so physics carries it wide; nothing is scheduled. Rivals can "see" through hills, which
gives them nothing a player lacks, because the line-of-sight rule keeps every bend in view in
time.

- **Rivals stay below the limit.** Every rival except Greedy has a corner factor at most 0.94,
  so a driver who corners at the limit always has something to gain.
- **Race form.** Each race, the seed gives every rival a small random form: up to ±2% on top
  speed and corner factor. It never depends on you. This is what makes races differ, and it
  makes win rates move smoothly as the numbers are tuned.
- **No rubber-banding.** A rival's target speed never reads your position, place or the gap to
  you. You are treated exactly like any other car when rivals steer around or follow a car.
  This is tested three ways.
- The level scales the field, and a track may carry a small field factor if one set of numbers
  can't hit the bands on all three:

| Level | Rivals' top speed vs yours | You start |
| --- | --- | --- |
| **Easy** | 82 to 88% | 4th of 6 |
| **Normal** | 90 to 97% | 6th of 6 |
| **Hard** | 96 to 102% (Lookahead is slightly faster than you on a straight) | 6th of 6 |

These are starting points: build 1 tunes them by simulation before anything is drawn (see
Tests).

### Easy help for kids

- **Auto-gas** follows the last input used: on with touch, off with a keyboard. The brake
  still works. It is a setting too.
- **Easy-only physics:** on Easy your car eases back toward the road when you drift off it, and
  the verge slowdown is gentler. Normal and Hard give everyone the same physics.

### Controls

- **Keyboard:** arrows or WASD to steer, gas and brake; P or Escape pauses; M mutes.
- **Touch:** hold the left half of the screen to steer left, the right half to steer right,
  both halves to brake. Turns ramp in over a fifth of a second. A small "your thumbs go here"
  hint shows at the first race.
- Touch rules: `touch-action: none` on the race; no long-press menu, text selection or
  double-tap zoom; each finger is tracked by pointer id, so sliding from one half to the other
  switches direction; every held input is released on pointer cancel, blur, pause and when the
  app goes to the background, so steering can't stick.
- The race pauses itself when the app goes to the background. Pause has Resume, Restart and
  Quit. The render loop stops while paused.
- Phones work in either orientation, with a one-time hint that sideways gives the widest view.
  No orientation lock.

### On-screen display

HTML over the 3D canvas, so it stays sharp and screen readers can reach it:

- Place ("3rd / 6"), lap ("Lap 2 / 3"), race time and your speed.
- A **progress strip** across the top: a line for the lap with a dot for each car in its color
  and mark, yours larger, so you can see where every rival is, including over a crest.
- A 3, 2, 1, Go countdown before the start (shown and heard; numbers only, no light gantry).
- **Results:** finishing order with times, your best lap, a new record when you set one, and
  Race again / Next track / Menu.

### Screen readers and other access

- Menus, settings, records and results are plain HTML buttons and text and work with a screen
  reader. Driving itself needs sight; About says so plainly.
- During a race a polite live region announces each lap and changes of place, at most once
  every 2 seconds.
- There is no camera shake and there are no speed lines. Reduced motion
  (`prefers-reduced-motion`, watched live) shortens menu transitions and turns off the body
  lean; the road still moves, since that is the game.

### Sound

Synthesized with the Web Audio API, so there is nothing to license: an engine hum from two
detuned oscillators through a low-pass filter, its pitch following your speed; a soft hum for
the nearest rival; a tire hiss in hard turns; a rumble on the verge; a thud on bumps; countdown
beeps; a lap chime; a short finish tune. No music in v1. Muted until the first tap, and the
mute setting is saved on the device.

## Flash safety

Racing games are a classic flash risk: striped curbs, road markings and scenery sweeping past
the edges of the screen can strobe several times a second over a large area, and in 3D the
first 20 m of road fills about half the screen. Kids will play, so the aim is zero flashes,
stricter than WCAG 2.3.1's limit of three a second. Every cap below applies to **rendered**
luminance (after shading and haze, with no tone mapping), not just palette values.

1. **Nothing repeats fast on the road.** Any pattern along the road or verge has a period of at
   least 40 m, so at the highest reachable speed (66 m/s) a point on the screen sees at most
   1.65 cycles, about 1.7 flashes, a second. Nothing periodic spans the road's width: marks
   stay in the edge strips. Lane and edge lines are continuous, never dashed, and there are no
   striped curbs.
2. **What changes is faint.** Neighboring ground facets, and any repeating pair, differ by less
   than 0.05 in relative luminance *and* by a ratio of at most 1.25 ((lighter + 0.05) /
   (darker + 0.05)), so dark Night Loop is as gentle as daytime Shoreline. Pairs stay in one hue
   family with a small chroma step (set in OKLCH, as in Vibes); no red against blue or cyan, and
   nothing alternates in saturated red. These caps are our own defaults, not from a standard,
   chosen well inside WCAG's 0.10.
3. **Nothing shimmers.** Lane and edge lines are at least 0.3 m wide, low contrast, and live in
   one small across-the-road texture with mipmaps and anisotropic filtering (nothing in it
   varies along the road). The ground texture is soft and low contrast, mipmapped. Night Loop's
   windows follow the skyline rule above. Blob shadows sit far enough above the road that the
   1 m near plane leaves no depth flicker.
4. **Quiet scenery.** Low rocks and dunes on one side stand at least 40 m apart (at most 1.65 a
   second) and are small. Trees, posts and lamp posts stand at least 10 m beyond the verge, so
   a post leaves the screen at about 1% of its width. Only cars cast shadows, as soft blobs, so
   no stripes of shadow cross the road. No glows, light pools, blinking windows or bloom.
5. **Calm skyline.** Hills and buildings on the skyline are within 0.05 of the sky behind them
   after haze, so turning hard in a hairpin doesn't sweep high-contrast outlines across the top
   of the screen. Crests are spaced so the horizon line moves on screen slower than a set rate
   in `tuning.json`, checked from the road's profile.
6. **Calm light.** No sun glare, lens flares or sparkly highlights: car paint has a roughness
   floor and no environment reflections. No tunnels, lightning, flashes on bumps or full-screen
   color changes, including at the start and finish.

**Checked three ways:**

- **Unit tests (CI):** every repeating pair, neighboring baked ground facets, every scenery
  color against the ground and sky behind it, and every skyline color, after the haze blend, in
  every track palette; the horizon-rate check from each track's profile.
- **`tools/scenery-flash.mjs` (CI),** a pure Node check: it takes the real camera along every
  track at the highest reachable speed on the road's centre and both edges, projects every
  roadside object's bounding box and every edge mark into cells about 40 px square (32 × 18 at
  1280 × 720, 10 × 21 at 390 × 844), and counts flashes with `art/flash.js`. It must count zero.
  It covers objects and marks only; ground, sky and skyline are covered by the unit tests and
  the scan.
- **`tools/flash-scan.mjs`** (Playwright with software WebGL, run before each PR that changes
  drawing or a track, not in CI, as prignano-theme's scan runs) steps the race with a fixed
  1/60 s test hook and captures every rendered frame at device pixel ratio 1, using the same
  cells. Per track: the Competent driver for a full race at both sizes, the verge-hugger for one
  lap at both sizes, and the Kid for one lap in portrait, plus one forced pixel-ratio step,
  the countdown and the finish (about 71,000 frames in all, run across several pages at once;
  build 1 records how long it takes). It must report zero, red included. If zero can't be
  reached, it goes back to the owner: the build thread never changes the scan or its threshold.
- Shimmer can't be counted by any of these, so "no shimmer on the road, lines or skyline" is
  one of the owner's phone checks.

## Quality and performance

The world is small: about 12,000 triangles of road, 7,000 of terrain and at most about 100
scenery instances a track. So there is one material set, no level of detail and no distance
culling.

- Budgets, checked by reading the code: under 30 draw calls (road one mesh, terrain one mesh,
  sky, each scenery type one instanced mesh, car bodies one material each, wheels and shadows
  instanced); no texture over 1024 px; no shadow maps; no post-processing.
- Target: a steady 60 frames a second on a three-year-old mid-range phone (the class pinball
  targets). Build 1 measures it on the owner's phone and records the numbers; nothing is
  promised before that.
- Shaders compile before the countdown (`compileAsync`), so the start never stutters.
- **Auto** (default) is a pixel-ratio ladder: 2, 1.5, 1.25, 1. Five seconds after the
  countdown, if the median frame takes over 20 ms across 3 seconds, it steps down one rung, mid
  race (that is cheap and safe; nothing is rebuilt). If a step doesn't bring the median frame
  time down (a screen capped at 30 frames a second, as in low-power mode), it goes back up a
  rung and stops stepping. If the lowest rung still takes over 50 ms, a one-time note says the
  device can't run it smoothly. On touch devices the canvas is also kept under about 3
  megapixels, a guess build 1 checks.
- The Graphics setting offers Auto, Sharp (ratio 2) and Fast (ratio 1); `?gfx=sharp|fast` and
  `?fps=1` (a frame-time readout) are for testing on a real phone.
- Changing tracks disposes every geometry, material and texture of the old one, since iOS kills
  tabs that leak GPU memory. Hidden instances are moved away, never scaled to zero.
- Without WebGL 2, or if three.js fails to load, the game says it needs a newer browser. If the
  WebGL context is lost, the race pauses; when the context comes back, the view is rebuilt and
  the race waits at Pause; if it hasn't come back in 3 seconds, a message asks for a reload.
  Menus and records work without WebGL. There is no 2D version.

### Loading and offline

- three.js loads with a dynamic `import()` when you first start a race, so the menus never wait
  for it. The two files are 2.13 MB uncompressed (about 420 KB compressed); a progress bar shows
  on the very first load.
- `sw.js` precaches every file, three.js included, so the installed app plays offline.

### Vendoring three.js

Copied from pinball: `tools/vendor-three.mjs` downloads `three@0.187.0` from the npm registry,
checks its published integrity hash, and copies only `three.core.js`, `three.module.js` and the
`LICENSE` into `vendor/three/` (none of pinball's post-processing files). No import map
(Safari 16.4) is needed. Upgrading means changing one version line.

## Records and your data

- Best race time and best lap per track, and the settings (auto-gas, graphics, mute), in
  `localStorage`. "Delete my data" in settings clears them.
- No names, accounts, analytics, ads, trackers or third-party scripts. The prignano-theme
  header loads from jsDelivr pinned to a commit SHA with a subresource integrity hash, as in
  chess, and is cached for offline use. three.js is served from our own files. A
  Content-Security-Policy header lists the only allowed sources.
- A short About screen says this in plain words, with the MIT notice, the credits (three.js;
  Lou's Pseudo 3d Page; javascript-racer, whose notice is in `THIRD_PARTY_NOTICES.md`) and a
  link to the source.

## Installable app

The PWA recipe from pinball PR #7, taken from Vibes' MIT copy: manifest (`display:
standalone`, no orientation lock), original icons (a small car on a road curving to the
horizon) in any, maskable and Apple touch sizes, iOS home-screen tags, a service worker that
caches every served file with its list stamped by `tools/sw-version.mjs` and a test that fails
when the list is stale, an Update button on the home screen, safe-area padding, the site header
hidden in the installed app, and sound resumed after an iPhone locks.

## Hosting

One Cloudflare Worker named after the repo, serving the folder as static assets only (no
Durable Objects, KV or secrets), as Infinite Recursion does. `.assetsignore` leaves out tests,
tools and docs. Workers Builds deploys each merge to `main`. Nothing here can create a bill.

The owner creates the repo and adds the custom domain. Claude writes `docs/DEPLOY.md` and sets
up the Worker on the account only with his yes (through the Cloudflare connector, as for
Infinite Recursion). This happens in build 1, so the owner's phone check has a URL. The last
step of `docs/DEPLOY.md` is the `content/games.json` PR in `prignanodotai-web`: the game goes
in hidden with an empty url when the repo is created, and the launch PR sets its url and
`live`.

## Files

```
index.html, styles.css, manifest.webmanifest, sw.js, _headers, icons/
LICENSE, THIRD_PARTY_NOTICES.md
data/tracks.js       the three tracks as control-point loops and palettes (pure data)
data/tuning.json     car physics, levels, rival personalities, field factors, track limits
src/track.js         spline, stations, curve, slope, limit speeds, racing line, sight lines,
                     scenery spots, baked ground colors, road and terrain vertex data (pure)
src/sim.js           fixed-step race: cars, bumps, laps, places, race end, results (pure, seeded)
src/drivers.js       the rival driver (pure)
src/camera.js        chase camera step from the car's state (pure)
src/three/view.js    renderer, pixel-ratio ladder, compile, dispose, context loss, messages
src/three/world.js   road, terrain, sky, skyline and scenery meshes from track.js's data
src/three/cars.js    the car models, wheels, lean, blob shadows
src/input.js         keyboard and touch, auto-gas (pure state machine plus DOM binding)
src/hud.js           place, lap, time, progress strip, countdown, live region
src/audio.js         Web Audio sounds
src/store.js         records and settings in localStorage
src/main.js          screens: home, race setup, race, results, records, settings, about
vendor/three/        three.js r187 core, MIT
wrangler.jsonc, .assetsignore, docs/DEPLOY.md
tools/vendor-three.mjs, tools/sw-version.mjs, tools/sim-bands.mjs,
tools/scenery-flash.mjs, tools/flash-scan.mjs, tools/check-3d.mjs
test/                node --test, including the scripted test drivers
```

No game code outside `src/three/` imports three.js, so the track, sim, drivers and camera run
in Node.

## Tests

### Scripted drivers

Fixed in `test/drivers.js`, never in `tuning.json`, so tuning can't move them:

| Driver | Numbers |
| --- | --- |
| **Expert** | Top speed 1.00, corner factor 1.00 (at the limit), long lookahead, no noise |
| **Competent** | Top speed 1.00, corner factor 0.95, Lookahead's lookahead, no noise |
| **Kid** | Auto-gas, never brakes, 400 ms reaction delay, steering error up to 15% of the road width, Easy's assists on |
| **Hands-off** | Auto-gas, never steers, Easy's assists on |

### `node --test` covers

1. **Tracks:** every track closes on itself with smooth curvature; every lap is 1.8 to 2.2 km;
   no centre-line radius under 30 m and no crest under 370 m; slopes inside the limits in
   `tuning.json`; the line-of-sight rule for every bend; no two stations more than 200 m apart
   along the track come within 30 m; terrain within 30 m of the road edge no higher than 1 m
   above it; the start straight is long enough for the grid; rocks and dunes at least 40 m
   apart on a side; trees and posts at least 10 m beyond the verge.
2. **Car physics:** gas, coasting and brake reach the numbers in `tuning.json`; no car ever
   exceeds 66 m/s; curves push outward; a car on the centre line at a curve's limit speed holds
   it at full steering and one 5% faster can't; the inside line covers less ground (true ground
   speed); the verge halves top speed; hitting a rock slows you to a crawl; a car that bumps
   another never ends up inside it; the race ends 30 seconds after the winner.
3. **Rivals:** over 100 seeded races on each track and level, every rival finishes; every rival
   but Greedy spends under a second off the road; no rival but Greedy has a corner factor over
   0.94.
4. **Fairness** (100 seeded races per track and level):
   - Easy: Competent wins at least 95%; Kid finishes in the top 3 at least 80%; Hands-off
     finishes but not in the top 3.
   - Normal: Competent wins 40 to 80% (tuned toward 60%).
   - Hard: Competent wins 10 to 50% (tuned toward 30%); Expert wins at least 90%, so Hard is
     always winnable.
   The bands are the targets; tuning changes `tuning.json`, never the test.
5. **No rubber-banding:**
   - the driver's target-speed function gives the same answer for states that differ only in
     the player's position and place;
   - with the player's collisions and visibility switched off, every rival's path matches frame
     for frame whether the player is parked or driving the Competent line;
   - with the player replaced by a scripted car flagged as a rival on the same path, every other
     rival's path is identical.
6. **Determinism:** the same seed gives the same race, step for step (a hash of the sim state).
7. **Camera:** the pose is smooth (no jump over 0.5 m or 3 degrees between steps, even on a bump
   or a crest), and the road centre 1 second ahead stays on screen in portrait at 390 × 844 on
   every track.
8. **Input:** synthetic pointer sequences: sliding across halves, both halves to brake, pointer
   cancel, blur, two fingers, pause while held. Nothing stays held.
9. **Flash:** the color checks and `tools/scenery-flash.mjs` (see Flash safety).
10. **Words:** a short banned list, stored encoded so the repo never spells it out (the names of
    well-known commercial racing games, their makers and their characters), appears in no
    shipped file.
11. **PWA:** the service worker's file list is current and includes the vendored files.

**Browser check:** `tools/check-3d.mjs` (from pinball; Playwright with software WebGL, phone and
desktop sizes, portrait and landscape, at each Graphics setting) loads every track, runs a race
with the Competent driver, steers one by synthetic touch, pauses and resumes, forces a context
loss and restore, and fails on any console error, a blank frame or an unexpected message. It
saves screenshots for the pull request. It runs locally before each build PR, not in CI.

**Done means:** all tests pass in CI; before each build PR the build thread runs the browser
check and the flash scan (zero flashes). Installing, playing offline, the frame rate, no
shimmer, and a child's hand on the touch controls, on a real phone both installed and in Safari,
are manual checks listed in `docs/DEPLOY.md`.

## Pull requests

After the owner creates the repo:

1. **Spec:** this file as `docs/SPEC.md`, `AGENTS.md`, `CLAUDE.md`, MIT `LICENSE`,
   `THIRD_PARTY_NOTICES.md`. Plus the hidden `games.json` entry in `prignanodotai-web`, and the
   small pinball README PR recording the MIT offer for the reused 3D pieces.
2. **Build 1, one track you can race:** first the pure track, sim, drivers, camera and
   `tools/sim-bands.mjs`, tuned until tests 1 to 5 pass on all three tracks' data, and the color
   checks and `tools/scenery-flash.mjs` passing, before anything is drawn (if the bands or the
   flash rules can't all be met, that goes back to the owner). Then vendored three.js, the 3D
   view with the pixel-ratio ladder, Shoreline's world and the cars, controls, auto-gas, HUD,
   records, the browser check, the flash scan, CI, `wrangler.jsonc`, `docs/DEPLOY.md`, and the
   Worker on the account with the owner's yes. **The owner plays it on his phone before build
   2**: the look, how it steers, how fast it feels and the frame rate decide the rest.
3. **Build 2, the rest:** Ridge's and Night Loop's worlds, sound, settings, About, the PWA
   recipe, test 11.

Then the launch `games.json` PR.

## Not in v1

Online play of any kind (by design), a Cup (several races with points), time trial and ghost
laps, records split by level, a shared leaderboard, game controllers, tilt steering, power-ups
or weapons, damage, flips or jumps, banking, bridges and crossings, a track editor, more tracks,
split screen, weather, tunnels, day-night changes, environment reflections, tone mapping,
bloom, shadow maps, level of detail, a 2D fallback view, music, theme line art on roadside
boards, camera shake and speed lines, a rival that learns your driving, WebGPU. Cup and time
trial are the first candidates for v1.1.

## Red team (Fable)

### First pass, 2026-11-10 (pseudo-3D draft)

| ID | Finding | Severity | Status |
| --- | --- | --- | --- |
| RC-1 | Roadside objects passing 6 a second per side would trip the zero-flash scan; the night track's lamps were the worst case | High | Fixed: 40 m spacing, scenery small or set back, no night glows, a Node projection check in CI |
| RC-2 | A 0.08 luminance cap is WCAG-legal but still allows strong 10 Hz flicker, worst on dark palettes; hue alternation was unbounded | High | Fixed: long periods, 0.05 and a 1.25 ratio cap, one hue family; caps marked as our own defaults |
| RC-3 | The scan drove only the clean line, and didn't say it steps and captures every frame | Medium | Fixed: 1/60 s hook, three drivers, countdown and finish; shake and speed lines cut |
| RC-4 | Normal and Hard win-rate bands couldn't be hit: no defined competent driver, no source of chance on Hard | High | Fixed: fixed scripted drivers, per-race form from the seed, rivals below the limit, an Expert that must win Hard, mid-band targets, per-track field factor, sim tuned before drawing |
| RC-5 | The no-rubber-banding test's distance condition was never met, and Drafter contradicted the rule | High | Fixed: rule restated as "never reads your position", three exact tests |
| RC-6 | The kid driver had no numbers, no floor, and races had no end | Medium | Fixed: Kid and Hands-off pinned, assists marked Easy-only, race end, auto-gas follows last input |
| RC-7 | Scope: Cup, time trial, ghosts and theme-art boards were extra; build 1 had no URL for the phone check | Medium | Fixed: deferred, records per track only, Worker setup in build 1 |
| RC-8 | The spec's numbers were javascript-racer's constants; its sprites come from a commercial game | Medium | Fixed: own units, MIT notice shipped anyway, facts corrected, "not this look" line |
| RC-9 | Two steer buttons under one thumb is hard for kids; touch edge cases and portrait were unspecified | Medium | Fixed: screen halves, touch rules, input tests, portrait camera, real-phone check with a child |
| RC-10 | Lap time, ghost size and the resolution rules didn't follow from the numbers | Low | Fixed: 1.8 to 2.2 km laps, ghosts deferred, measured not promised |

### Second pass, 2026-11-11 (real 3D)

| ID | Finding | Severity | Status |
| --- | --- | --- | --- |
| R3-1 | The caps bound palette colors, but sun-lit flat-shaded hills render neighboring facets about 0.15 apart, sweeping past several times a second | High | Fixed: ground unlit with baked shading tested in Node, light only on cars and objects, no tone mapping, caps apply to rendered luminance |
| R3-2 | The sim's distance across the road was undecided; at a 25 m hairpin it moves lap times about 3%, more than race form | High | Fixed: true ground speed, minimum radius 30 m, limit speed from the centre line, tested |
| R3-3 | A 20 m period at 60 m/s is 3 flashes a second, not 1.5; no area limit for a road filling half the screen; no named speed cue | Medium | Fixed: 40 m periods at the highest reachable speed (66 m/s), nothing periodic spans the road, speed cues named for the owner's check |
| R3-4 | 3D adds turning and crests; the skyline sweeps in hairpins and the horizon bobs on rollers, which the Node check didn't model | Medium | Fixed: skyline contrast rule, horizon-rate check from the profile, Node check's scope stated |
| R3-5 | Rivals see through blind crests while the player can't; tight crests look glued at 1.8 g | Medium | Fixed: line-of-sight rule for every bend, crests at least 370 m |
| R3-6 | The camera had no numbers; the portrait apex test was impossible; a 60 Hz sim judders on 90/120 Hz screens | Medium | Fixed: field of view, look-ahead, near and far planes, a pure step in sim time, drawing blends two sim states, reachable portrait test |
| R3-7 | Day-one geometry gaps: spline type, arc length, frame, folding, terrain meeting the road, "neighbors", car heading | Medium | Fixed: closed cubic B-spline, arc-length table, up × tangent, ribbon limits, terrain recipe, 200 m neighbor rule, heading includes slide |
| R3-8 | Shimmer had no check, and thin lines, night windows and fading posts would cause it | Medium | Fixed: lines in a filtered across-road texture, skyline texture rule, posts set back and unreachable (no fade), shimmer on the owner's phone check |
| R3-9 | Scope: reflections, two material sets and chunk culling weren't needed for a 30,000-triangle world; draw calls were over budget | Medium | Fixed: `RoomEnvironment` cut, one material set, no culling or level of detail, instanced wheels and shadows, budget 30 calls |
| R3-10 | The Auto rule had no numbers, its 30 fps exemption couldn't be detected, and the fallback message was wrong for a lost context | Medium | Fixed: pixel-ratio ladder with numbers, step-back when a step doesn't help, a context-loss path, megapixel cap marked a guess |
| R3-11 | The flash scan was about 280,000 software-rendered frames per run; the determinism test needed a browser | Medium | Fixed: about 71,000 frames at ratio 1 across pages, run time recorded; determinism hashes sim state in Node |
| R3-12 | The license claim holds, but pinball's `view.js` carries a GPL header and nothing records the owner's grant; the flash-scan source and grid were misstated | Low | Fixed: pinball README PR, only the generic pieces come across, PWA files from Vibes, scan source and cell grid corrected, real-car rule, name check recorded |

Accepted as is (checked and passed, so nobody reopens them): crediting Lou's page; no tunnels or
bump flashes; continuous lane lines; reusing `flash.js`; three tracks as data; three levels;
starting 4th on Easy and 6th otherwise; the Easy Competent band; auto-gas on touch; no tilt or
controllers; the PWA recipe; no music; track coordinates as the sim with no banking; heights
drawn not simulated (a 10% grade changes length by 0.5%); rivals disappearing over crests (the
progress strip covers it); instanced scenery, blob shadows and dispose on track change; no 2D
view (WebGL 2 is in Safari 15 and up); "nothing promised before measuring".
