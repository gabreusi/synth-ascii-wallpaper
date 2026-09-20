# Outrun ASCII

An outrun wallpaper generator rendered entirely in ASCII characters: a sliced sun,
a sky of twinkling stars and a perspective grid that runs toward the horizon.
The parameters can be adjusted in real time, and the scene can be cropped and
exported as a PNG.

## Running

```bash
npm install
npm run dev      # dev server with HMR
npm run build    # typecheck + static bundle into dist/
npm run preview  # serve dist/ to check the build
```

There is no framework: Vite and TypeScript, plus two runtime packages
(`html2canvas` to capture the stage, `cropperjs` for the crop).

## How it works

The scene is text, not a canvas. Two `<span>` elements inside a single `<pre>`,
one for the sky and one for the floor, share the same monospace grid. The layers
are separate because they change at different rates: the sky is expensive to
build and almost static, so it is only reserialized when a star changes
character, while the floor is redrawn every frame.

Two decisions explain most of the code.

The grid comes from a measurement. `src/engine/layout.ts` measures one cell of
the font in an invisible `<pre>` (`#probe`), because the width to height ratio
changes with whichever font the system ends up choosing and with the CSS
`clamp()`. All of the perspective depends on that number.

Nothing is allocated inside the animation loop. `CharBuffer` is created once per
layout and reused, and each frame clears and repaints it. Serialization groups
neighboring cells of the same color into one `<span>`, since otherwise there
would be thousands of elements.

## Structure

```
src/
  config.ts          adjustable state and character palettes
  types.ts           Layout, Cell, Star
  dom.ts             lookup of the required elements
  engine/
    noise.ts         deterministic noise per coordinate
    buffer.ts        character grid + serialization to HTML
    layout.ts        cell measurement, perspective, background gradients
    sun.ts           sun disc and horizontal slices
    stars.ts         star spawning and brightness cycle
    floor.ts         horizon, fog, rails and depth lines
    renderer.ts      animation loop and scene state
  ui/
    controls.ts      settings panel
    export.ts        capture, crop and download
  styles/            tokens, base, scene, panel, modal
```

`src/config.ts` is the only source of the default values: the HTML declares just
the range of each slider, and the panel initializes itself from `settings`.
