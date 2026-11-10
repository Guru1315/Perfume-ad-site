# How it works

ÉCORCE is one long scroll story where **a single 3D glass bottle travels between sections** instead of each section
having its own image. Everything below is how that is held together. (Plain-language and pseudocode only — the
source is private.)

## The page flow

1. **Intro** — the bottle follows the cursor over a dark stage. Press and hold to spray mist; a ring fills; at 100% a
   cream bloom floods out of the nozzle, the site appears underneath and the bottle flies to its place in the carousel.
2. **Home** — ten bottles on a diagonal arc (bottom-left → top-right) over a star-shaped seal with rotating manifesto text.
3. **Fly-in** — the chosen bottle flies into its product page while the other nine fall away.
4. **Product hero** — a two-line editorial title, and the bottle on a travertine plinth with leaves, a flower, cloves and a plum.
5. **Story** — the title bursts apart; story words flip up; the bottle makes one full showcase turn (with a degree counter).
6. **Marquee** — two giant rows of notes and tagline that lean with scroll speed.
7. **Other extraits** — a concave arc of cards; the hero bottle docks into a dashed “Yours” card, the other nine sit in theirs.
8. **Into the scent** — a 3D tunnel of note words the camera travels through.
9. **Composition** — top / heart / base take turns while every note orbits the bottle on a ring of text.
10. **Worn & wanted → ten moods → contact → footer.**

## Layering rule

```
backgrounds / photos   <   DOM text   <   WebGL canvas (the bottle)   <   header
```

The canvas is fixed over the page with pointer events switched off; the DOM does its own hit-testing against the
on-screen boxes the renderer publishes each frame. Photos stay in side lanes so they never fight the bottle.

## One clock

```
on every animation tick(time, dt):
    smoothScroll.update(time)          # Lenis
    scrollTriggers.update()            # GSAP ScrollTrigger
    carouselPhysics(dt)                # glide, snap, colour blend
    domWrites(dt)                      # seal rotation, counters, tunnel, rings
    bottlePoses(dt)                    # where every bottle should be
    if something changed: render()     # WebGL, on demand
```

Nothing else runs its own animation loop, so the DOM and the 3D never drift by a frame.

## A bottle that follows the page

Every section that “holds” the bottle has an invisible placeholder element. Each frame:

```
measure every placeholder's rectangle on screen
find the two placeholders either side of their focus line
blend position, size and rotation between them by how far you've scrolled from one to the other
convert that screen rectangle into a 3D pose
```

- Pinned (sticky) sections report their own focus line, so the bottle docks wherever the layout puts it — mid-screen
  on desktop, the top half of the split-screen story on phones.
- Rotations chain: the story's turn ends where the card dock starts, the tunnel adds one more whole turn, the
  composition sways around it — so the bottle never unwinds between sections.
- Home ↔ product uses the same idea: the carousel pose and the hero pose are both screen rectangles, and a click
  just blends from one to the other.

## The carousels (home arc, card rings)

```
position(i, u):              # u = continuous offset
    d = wrapped distance of card i from the centre
    place on the arc / ring by d   (scale and fade with distance)

drag:     offset follows the pointer, velocity is smoothed
release:  glide with friction, then snap to the nearest card
idle:     slow drift — only while in view, not hovered, and the page isn't scrolling
```

The card rings are CSS 3D with the camera inside the ring, so outer cards are nearer and larger. Their size unit is
`min(1vh, 1.6vw)` so a tall phone still gets a ring that fits.

## Glass, light and the still life

- **Procedural bottles** — five silhouettes × six cap styles, labels drawn with the real fonts on canvases.
- **Real glass** — transmission (refraction) with the liquid inside. A backdrop plane is drawn *only* into the
  refraction pass, so the glass bends the page colour instead of the renderer's default white.
- **Linear pipeline** — the scene renders into a linear half-float target and is tone-mapped once at the end. The
  refraction pass and the main pass then share shader variants (otherwise every material switches programs twice a frame).
- **Still life** — plinth, leaves (alpha-cut, bent planes), gardenia, cloves (instanced) and a plum, merged per material
  into a handful of draw calls; a warm sun casts real shadow maps, redrawn only when something actually moves.
- **Textures** — Poly Haven plaster (with travertine pits drawn over it), walnut veneer and marble; 2K on desktop, 1K on phones.

## Fast first load

```
behind the preloader:
    load fonts, HDRI and textures
    draw the labels
    compile every shader variant (including shadow depth shaders)
    upload every texture, one per task
reveal only when that's done
```

## Mist

Intro mist is a pool of GPU point sprites advected by a curl-like noise field — they grow, swirl and fade, and the
geometry is hidden (no rendering) once the last one is gone. The contact section uses a 2D-canvas version that follows
the cursor and bursts from the orb on click.

## Menu

The Menu button opens a full-screen panel that grows out of the button as a circle. It lists the ten scents — picking
one uses the same flight as clicking a bottle, so the chosen bottle flies to its page — plus every section of the
current page (smooth-scrolled), and the contact details. Esc, Close or any link shuts it.

## Phones

A device tier gives phones 1K textures, a pixel ratio ≤ 1.3, a lighter refraction pass, 2× anti-aliasing and a smaller
shadow map. Every section has a phone layout, and touch works for swipe, tap and press-and-hold.
