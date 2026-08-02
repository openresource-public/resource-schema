# Models (computed figures)

A **model** (also called a primitive) is a versioned, parameterised coded visual — a coordinate grid, an area model, a geometry construction, and so on. Where an image is a bound asset, a model is *generated*: the block stores only a pinned reference plus parameters, and the platform resolves the reference, runs the published code, and renders the figure. Published versions are immutable, so a reference renders identically forever, across web, print, and slides.

A model is referenced with a uniform envelope wherever it appears:

```ts
type PrimitiveRef = {
  ref: string,        // 'family/name@version', e.g. 'graphing/coordinate-grid@1'
  params: object,     // matches the referenced model's published interface
  altText: string,    // accessibility, search, and render-failure fallback
  steps?: { id: string, params: object }[]   // staged build a beat can step through
}
```

`ref` always pins a version (`@1`). `params` is validated against that model's published interface when the block is authored. `altText` is required — it is the accessibility text, the search surface, and the fallback shown if the figure cannot render. `steps` gives the figure a staged build: each step is a full paramset (not a patch), advanced by a beat's `stage` op.

## Where models appear

The model slot is a peer of `image` — same envelope position, but generated from `params` rather than bound to an asset. See [SCHEMA.md](SCHEMA.md#models) for how it sits in the block envelope. It appears in two places:

- **a `model` tile** — the model as a figure in the grid: a standalone figure, a teaching visual, or the stimulus beside a question tile. With a tile-level `responseMode`, the model is the *answer surface*: the student responds by acting on it (placing a point, selecting a point, shading a region, or running it to a value).
- **items of markable and collection tiles** — an option, matching item, ordering item, or grid item can carry a model as its figure (the item slot is named `primitive`).

## Answering on a model, and derived keys

When a `model` tile carries a tile-level `responseMode`, the model itself is the answer surface — the student places points, draws a curve, shades a region, or runs the model, and that action is what gets marked. Two marking shapes exist:

- **Literal key** — `marking.correctAnswer` stores the expected placement explicitly (point coordinates, a selected id, a region), shaped per the model's published interface.
- **Derived key** — `marking.correctAnswer: { derived: true }`. The key is *computed from the model's own `params` by the pinned model version*: a cumulative-frequency plot derives the expected curve from the stored grouped distribution, a box-plot derives its five values from the placed data, and so on. There is no separate evaluator to version — the model version referenced by `ref` IS the evaluator, so `@4` marks identically forever, exactly as it renders identically forever. What `derived` computes, and with what tolerance, is part of each model version's published interface (`models/<name>.md`).

Draw-style response modes accept an error band around the derived target; by convention the band is a quarter of the axis step (`step/4`) unless the model's interface documents otherwise. Typed read-off answers that accompany a figure (e.g. "estimate the median") are ordinary typed keys with an explicit `tolerance` — the tolerance expresses acceptable graph-reading error, and the stored value is the source's published answer.

## The library

The model library is large and grows over time; it is discovered programmatically rather than enumerated in this repo. This repo publishes **one** model's interface in full as a worked reference:

- [`graphing/coordinate-grid`](models/coordinate-grid.md) — a labelled coordinate grid with optional plotted points and segments.

Each model's parameters are documented at [`models/<name>.md`](models/). The interface shape (a JSON-schema of `params` with per-property descriptions, plus worked example paramsets) is the same for every model.
