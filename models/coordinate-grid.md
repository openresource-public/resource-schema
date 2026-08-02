# `graphing/coordinate-grid`

A coordinate grid with numbered axes, optional plotted points (labelled, with optional coordinate readouts), optional line segments between points, and axis-parallel constant lines (`x = k` / `y = k`) with derived equation labels. Ranges that cross zero produce a four-quadrant grid automatically — first quadrant vs four quadrants is derived from the axis ranges, not a separate flag.

This is the worked reference for the model interface — every model is referenced and parameterised the same way (see [MODELS.md](../MODELS.md)).

## Reference

```ts
{
  ref: 'graphing/coordinate-grid@4',
  params: { /* see below */ },
  altText: string
}
```

## Parameters

```ts
{
  axes: {                              // required
    x: [number, number],              // [min, max], e.g. [0, 10] or [-5, 5]
    y: [number, number],
    step?: number,                    // grid spacing in axis units (default 1)
    labelStep?: number,               // number the axes every N units (default: every grid line)
    labels?: [string, string]         // axis names, default ["x", "y"]
  },
  grid?: boolean,                      // show grid lines (default true)
  points?: Array<{
    x: number,
    y: number,
    label?: string,                   // point name shown beside the marker
    style?: 'dot' | 'open' | 'cross', // marker shape (default 'dot')
    occlude?: boolean,                // hide this point's readout as "(?, ?)" — the question hook
    color?: string                    // CSS colour for the marker (default theme accent)
  }>,
  segments?: Array<[string, string]>,  // line segments as pairs of point labels, e.g. [["A","B"]]
  lines?: Array<{
    x?: number,                       // vertical line at x = k (exactly one of x or y)
    y?: number,                       // horizontal line at y = k
    label?: string,                   // override the derived "x = k" / "y = k" label
    showLabel?: boolean,              // default true
    occlude?: boolean,                // hide the equation label as "?" — the question hook
    highlight?: boolean               // draw in the emphasis colour
  }>,
  highlight?: string[],                // labels of points to ring (draws the eye)
  showCoordinates?: boolean,           // render "(x, y)" readouts beside labelled points (default false)
  header?: string,                     // short title above the grid
  draggable?: boolean                  // web-only: points can be dragged, snapping to the grid
}
```

`labelStep` lets a unit grid count its axis numbers in 2s or 5s so wide ranges stay legible. Combine `showCoordinates` with a per-point `occlude` to turn a worked example into a question ("what are the coordinates of C?"). A constant line spans the whole grid and needs no points — where a segment joins two named ones; `occlude` on a line hides its equation for "what is the equation of this line?".

### Answering on the model

Hosted on a [`model` tile](../blocks/bento.md) with a `responseMode`, this model supports three response modes:

- `place-point` — the student clicks a grid intersection; the answer is `{ x, y }`.
- `select-point` — the student taps one of the rendered points; the answer is its `label`.
- `place-line` — the student drags the two handle points of a pre-placed wrong line onto the target line. The target lives in the `placeLine` param (`{m, c}` | `{x}` | `{y}`), the key is `correctAnswer: { derived: true }`, and any two distinct points on the correct line pass. Static surfaces render a blank answer space for it.

## Example paramsets

Blank first-quadrant grid (an answer space):

```json
{
  "axes": { "x": [0, 10], "y": [0, 10] },
  "header": "Plot the points",
  "points": []
}
```

Worked example — plotted points with readouts:

```json
{
  "axes": { "x": [0, 10], "y": [0, 10] },
  "showCoordinates": true,
  "points": [
    { "x": 3, "y": 2, "label": "A" },
    { "x": 7, "y": 5, "label": "B" }
  ]
}
```

Question — one readout occluded and ringed:

```json
{
  "axes": { "x": [0, 10], "y": [0, 10] },
  "showCoordinates": true,
  "points": [
    { "x": 3, "y": 2, "label": "A" },
    { "x": 7, "y": 5, "label": "B" },
    { "x": 4, "y": 8, "label": "C", "occlude": true }
  ],
  "highlight": ["C"]
}
```

Four quadrants with a segment:

```json
{
  "axes": { "x": [-5, 5], "y": [-5, 5] },
  "points": [
    { "x": -3, "y": 2, "label": "P" },
    { "x": 4, "y": -1, "label": "Q" }
  ],
  "segments": [["P", "Q"]]
}
```

Wide range numbered in 2s (unit grid, uncrowded labels):

```json
{
  "axes": { "x": [0, 20], "y": [0, 14], "labelStep": 2 },
  "showCoordinates": true,
  "points": [
    { "x": 6, "y": 8, "label": "A" },
    { "x": 15, "y": 3, "label": "B" }
  ]
}
```

Constant lines crossing on four quadrants:

```json
{
  "axes": { "x": [-6, 6], "y": [-6, 6] },
  "lines": [{ "x": 2 }, { "y": -3 }]
}
```

Question — name this line (occluded equation):

```json
{
  "axes": { "x": [-5, 5], "y": [-5, 5] },
  "header": "What is the equation of this line?",
  "lines": [{ "y": 3, "occlude": true }]
}
```
