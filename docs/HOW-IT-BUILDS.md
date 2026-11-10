# How it builds

## Stack

| Layer | Tools |
|---|---|
| Framework | Next.js 16 (App Router, TypeScript), React 19 |
| 3D | three.js via React Three Fiber + drei, post-processing output pass |
| Motion | GSAP + ScrollTrigger, Lenis smooth scroll — all on one shared ticker |
| State | Zustand for page mode / selection; plain mutable objects for per-frame data |
| Content | All copy and product data (10 scents, notes, moods, bottle shapes) in one data file |
| Assets | Poly Haven CC0 HDRI + plaster / walnut / marble textures; CC0 / public-domain perfume-bottle photos; Fraunces, Manrope, Anton via `next/font` |
| Hosting | Vercel |

## Build and deploy

```
install dependencies
production build   → Next.js statically generates "/" and all ten "/extrait/<scent>" pages
deploy             → Vercel (framework preset: Next.js)
```

Static pages + client-side WebGL: there is no server work per request. Everything heavy (fonts, HDRI, textures, shader
compilation) happens in the browser behind the preloader.

## Built in stages

Each stage was checked running in a browser — motion and frame timing, not only screenshots — before moving on.
Motion was measured with headless Chrome driven over the DevTools protocol (frame times, per-section timing, pose
continuity, docking accuracy), on an Intel UHD laptop GPU.

| # | Stage | What landed |
|---|---|---|
| 1 | Setup + home + fly-in | One fixed canvas, one clock, 10 procedural glass bottles on the diagonal arc over the star seal, drag/wheel/keys, “View” cursor pill, click → fly-in to a product hero |
| 2 | Scroll story | Title burst, story word flips, full showcase turn with degree counter, velocity-leaning marquee, 3D word tunnel |
| 3 | Other extraits | Concave card ring, “Yours” dock for the hero bottle, the other nine bottles in their cards, card click → product-to-product flight |
| 4 | Composition | Sections reordered to the design's flow; three movements with notes orbiting the bottle |
| 5 | Worn & wanted | Shared ring mechanics, filters, bounded small sets |
| 6 | Ending | Ten-moods gallery, contact with cursor mist + magnetic spray orb + copy email, giant wordmark footer |
| 7 | Intro | Bottle follows the cursor, hold-to-spray WebGL mist, cream bloom reveal |
| 8 | Still life | Travertine plinth, leaves, gardenia, cloves, plum, warm sun with real shadows |
| 9 | PBR textures | Poly Haven plaster / walnut / marble, 2K desktop / 1K phone |
| 10 | Phone pass | Device tier, phone layouts for every section, touch, real-phone check over Wi-Fi |
| 11 | Photos + menu | Free-licence perfume-bottle photos in every photo slot, a working full-screen menu, real contact email |

## Performance fixes that mattered

| Problem (measured) | Fix |
|---|---|
| ~21 fps idle; JS work 12–16 ms/frame | Render on demand; scene rendered into a linear target and tone-mapped once, so the refraction pass stops switching every material's shader twice a frame → ~4 ms/frame |
| Drag and clicks did nothing | The WebGL canvas wrapper was catching pointer events — switched off; the DOM hit-tests the bottles itself |
| Big seal layers repainting every frame | Seal colour blend done as a two-layer opacity cross-fade (compositor only) |
| Bottle jumped ~40% of the screen between sections | Interpolate between the two slots either side of the focus line instead of weighting all slots |
| Stutters while scrolling past the card arc and tunnel | Arc drift pauses while the page scrolls; tunnel words get a pinned raster so Chrome stops re-rasterising them |
| 2 s freeze after the reveal | This three.js version swaps the deprecated soft shadow type at first use, recompiling every lit shader — start with the plain PCF type; compile shadow shaders behind the preloader |
| 0.75 s hitch on first frame | Upload every texture behind the preloader, one per task |
| Still life ~4 ms/frame CPU | ~30 meshes merged per material into ~6 draw calls; shadows redrawn ≤ 12×/s and frozen once the bottle lifts off |
| Hold-to-spray heavy | Fewer, softer mist particles; the bloom is a small disc scaled up rather than a huge layer |

**Result (headless Chrome, Intel UHD, 1600×900):** whole product page scroll-through ~55–58 fps (median frame 17 ms);
home idle 60 fps with no GPU work. Phone emulation (390×844 @2×): ~45 fps scrolling, 60 fps idle.

## Reference

Built from the public specs (design notes, prompt sequence, pseudocode and six reference screenshots) in
[alagappan567/perfume-ecorce-showcase](https://github.com/alagappan567/perfume-ecorce-showcase).
