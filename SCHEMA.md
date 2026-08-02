# Open Resource schema v0.11

A data model for educational resources.

## The shape

```ts
{
  id: string,
  type: 'resource',
  title: string,
  description?: string,

  blocks:   { [blockId]:  Block },
  sections: { [sectionId]: Section },
  sequence: SectionId[],

  objectives:       Objective[],
  keywords:         Keyword[],
  misconceptions:   Misconception[],
  assumedKnowledge: AssumedKnowledge[],

  meta: {
    schemaVersion: '0.11',
    credits?: string     // optional provenance line for the resource as a whole
  }
}
```

A resource is a flat dictionary of blocks and sections referenced by id. The top-level `sequence` carries section ids in order. Resource-level lists (objectives, keywords, misconceptions, prior knowledge) live at the root. `meta.credits` is an optional human-readable source line ("Contains questions from …"); finer-grained provenance lives per block at `meta.attribution` (see [Blocks](#blocks)).

## Sections

A section is one phase of a lesson. Its `type` is a hint, not a rule — it shapes how the lesson is authored and displayed, but doesn't change what the content means. The six types cover the usual stages of a lesson:

- `intro`: opening — introduction, learning outcome, glossary.
- `retrieval`: activates prior knowledge.
- `learning-cycle`: a full teach → check → practice arc on one sub-topic.
- `activity`: extended practice or investigation; no new teaching.
- `assessment`: standalone summative phase, separate from any learning cycle.
- `plenary`: closing — summary and reflection.

```ts
{
  id: string,
  type: 'intro' | 'retrieval' | 'learning-cycle' | 'activity' | 'assessment' | 'plenary',
  heading: string,
  sequence: BlockId[]
}
```

A section's `sequence` carries the ids of the blocks that make up that phase, in order.

## Blocks

A block is the atomic content unit: a collection of **tiles** — prose, workings, figures, tables, dialogue, collections, question interactions — with an optional click-through reveal timeline. Each tile carries a cell that arranges the collection on a 12-column bento grid. The grid is the *composed* layout — slides and print render it as authored; the web reading surface deliberately drops the grid and linearizes tiles to one reading order (list order), so a resource never depends on a minimum viewport. A single paragraph is a block with one text tile; a question is a block with a markable tile.

```ts
{
  id: string,
  type: 'bento',         // fixed — a format discriminator, not a choice
  role?: 'question' | 'worked-example' | 'key-point' | 'objectives' | 'glossary',
  heading?: string,
  question?: string,     // question-group id — see Multi-block questions
  content: object,       // blocks/bento.md
  meta?: object          // optional provenance metadata — see below
}
```

See [blocks/bento.md](blocks/bento.md) for the tile catalogue and content shape. A markable tile carries `marking` (how it's scored), and any block may carry an `explanation` (block-id refs to the revealed worked answer) — see [Marking](#marking) below and [blocks/bento.md](blocks/bento.md).

`role` declares the block's pedagogical register; absent means plain content. `question` is a posed or auto-marked task — it drives assessment extraction, quiz mode, answer withholding, and question numbering. `worked-example` is a modelled solution the pupil reads; it may carry marking (an inline check) but never enters assessment. `key-point` is a callout (a definition, a theorem statement). `objectives` and `glossary` are projection blocks — they carry no tiles and render the resource-level registries where they sit; an optional `content.projection.ids` selects a subset of entries. The role is declared, not derived: a block with a marked tile or `explanation` refs must be flagged `question` (or `worked-example`), and validation enforces the tie.

### Multi-block questions

The usual multi-part question is ONE block whose tiles carry `part` tags (`'a'`, `'b'`, …) — see [blocks/bento.md](blocks/bento.md). When one source question is too large for a single block (each part needs its own full grid, say), it may span several blocks instead: every member carries the SAME `question` value (a group id such as `"q3"`), `role: 'question'`, and the members sit contiguously in their section's `sequence`. Part letters stay on tiles, never in the group id.

Surfaces then treat the group as one question: it takes one number, renders one summed "(Total for question: …)" line (after the last member), folds to one answers-key entry, and continuation members suppress their repeated `heading`. Worked-answer `explanation` refs stay wherever they were authored — conventionally on the last member for a combined solution, or per member for per-part solutions. Validation requires members to be contiguous and warns on a single-member group (a single-block question simply omits the field).

`meta` is optional provenance metadata. The one defined sub-shape is `meta.attribution` — where the block's content came from and under what licence (see the pythagoras example, whose question blocks each carry one):

```ts
type Attribution = {
  source: string,        // the publishing site or body
  work: string,          // the specific work the content is drawn from
  sourceUrl?: string,
  author?: string,
  licence: string,       // licence identifier, e.g. 'CC-BY-SA-4.0'
  licenceUrl?: string,
  changes?: string       // how the content was adapted
}
```

## Rich text

Every text-bearing field on a block or registry entry stores a [TipTap](https://tiptap.dev) JSON document. Inline styling (bold, italic, colour, subscript, superscript, links) survives between authoring surfaces and renderers. The full shape is in [tiptap-doc-schema.json](tiptap-doc-schema.json); below, `Doc` is shorthand for it:

```ts
type Doc = {
  type: 'doc',
  content?: Node[]
}
```

### Inline math

Equations are nodes inside a paragraph:

```json
{
  "type": "math",
  "attrs": {
    "latex": "x^2 + y^2 = z^2",
    "display": false
  }
}
```

LaTeX is the canonical form. `display` is a boolean: `false` renders the equation inline with the surrounding text, `true` as its own centred line.

## Image

Many places pair an optional image with text: an `image` tile, table cells, and the option / matching / ordering / grid items of markable and collection tiles. Wherever this pairing appears, the image envelope is uniform:

```ts
type Image = {
  src?: string,        // the bound asset
  search?: string,     // unfilled slot: an image-search query in place of src
  altText: string,
  caption?: Doc,       // figure gloss, under the image
  attribution?: Doc,   // credit / licence line, under the caption
  orientation?: 'landscape' | 'portrait'   // layout hint beside text
}
```

`altText` is required for accessibility. `caption` is a `Doc`, so it can carry inline styling and links. `src` is an asset URL or a platform asset id; the example lessons use `"PLACEHOLDER"` where binary assets were stripped. A slot authored before any asset is bound carries `search` — a query the platform resolves to openly-licensed candidates for the teacher to pick — instead of `src`.

Stored resources may also carry platform-written presentation fields on these slots. All are must-ignore for consumers:

- `srcRect` — fractional crop `{ l, t, r, b }`
- `rotation` — degrees clockwise
- `width` / `slideWidth` — drag-resize width, as a percent of the host
- paragraph attrs `bullet`, `pEndDefaults`, `align` inside tiptap docs — import provenance

The general rule: unknown fields are must-ignore, never must-understand. Both `text` and `image` are optional in each block where this pair appears — renderers compose text-only, image-only, or both. Where a block can instead carry a computed figure, that figure slot is a **model** rather than an image — see [Models](#models).

## Models

Some blocks can carry a **computed model** (also called a primitive) as their figure, instead of or alongside an image. A model is a versioned, parameterised coded visual from the platform library (coordinate grids, area models, geometry constructions, …): the block stores a pinned reference plus parameters, and the platform resolves the reference, runs the published code, and renders the figure. Published versions are immutable, so a reference renders identically forever.

```ts
type PrimitiveRef = {
  ref: string,        // 'family/name@version', e.g. 'graphing/coordinate-grid@1'
  params: object,     // matches the model's published interface
  altText: string,    // accessibility, search, and render-failure fallback
  steps?: { id: string, params: object }[]   // staged build — see below
}
```

`steps` gives a figure a staged build: each step is a full paramset (not a patch), and a beat's `stage` op advances the figure through them in sync with the prose.

The model slot is a peer of `image` — the same envelope position, generated from `params` rather than bound to an asset. It appears on a `model` tile (a standalone figure is a block with one model tile; as a stimulus beside a question tile; or, with a `responseMode`, as the answer surface the student acts on) and on the option / matching / ordering items of markable tiles. See [MODELS.md](MODELS.md) for the concept and one fully-worked model interface.

## Marking

A markable tile carries a `marking` object describing how it is scored, discriminated by `mode`:

```ts
type Marking =
  | { mode: 'auto',          // machine-checked
      correctAnswer: unknown, // the key — shape per question type (a value, an array for an accepted set, or [min, max] for a range)
      match?: MatchMode,      // how to compare; inferred by type when omitted
      tolerance?: number,     // numerical: ± acceptance band
      marks?: number }        // omit for an ungraded formative check
  | { mode: 'rubric',         // human-marked extended response
      scheme: Rubric,         // how the marks are awarded
      marks: number }
  | { mode: 'none' }          // discussion / unmarked

type MatchMode = 'exact' | 'set' | 'range' | 'tolerance' | 'latex'
```

`correctAnswer` is the single home for the answer, shaped per tile kind (see [blocks/bento.md](blocks/bento.md#marking)). Typed answers are normalised before comparison — case, whitespace, unicode symbol variants, thousands separators, redundant zeros — so an accepted-answer set lists only semantically distinct forms.

On auto marking, `marks` is display metadata — what the question is worth (the "[2 marks]" margin), not an award scheme: machine verdicts are binary. A multi-part question is one block with several markable tiles, each tagged `part: 'a' / 'b' / …` and holding its own key; part totals and the question total derive from the tile marks, never stored.

`mode: 'rubric'` sits on a `text` or `working` tile — the tile is the task statement, and the scheme says how a human awards its marks. Rubric marking is never machine-scored. A rubric-marked tile may carry a tile-level answer space:

```ts
response?: { kind: 'free-text' | 'working', lines?: number }
```

rendered as an input on the web and as ruled prose lines (`free-text`) or working space (`working`) in print. A `Rubric` is one of three shapes:

```ts
type Rubric =
  | { type: 'criteria',
      criteria: { description: Doc | string, marks: number }[] }
  | { type: 'levels',
      levels: { range: [number, number], descriptor: Doc | string }[] }
  | { type: 'method-accuracy',
      marks: { kind: 'method' | 'accuracy', description: Doc | string,
               marks: number, answer?: unknown, tolerance?: number }[] }
```

A block may also carry an ordered list of block ids whose blocks present the revealed worked answer:

```ts
explanation?: BlockId[]
```

Each `explanation` block lives in the `blocks` map but in no section sequence, and is referenced by exactly one owning block. Any `marking` mode may carry an `explanation` — even an auto-marked multiple-choice question can reveal why an option is correct. A question declares itself: the owning block carries `role: 'question'` (see [Blocks](#blocks)). Explanation blocks themselves stay role-less — they are answer furniture reached through their owning question.

## Registries

Resource-level lists at the root.

```ts
type Objective        = { id, text: Doc }
type Keyword          = { id, term: Doc, definition?: Doc }
type Misconception    = { id, text: Doc, response?: Doc }
type AssumedKnowledge = { id, text: Doc }
```

---

Resources can also be imported from existing pptx decks. Imported blocks additionally carry an `elements` array referencing source-deck shapes for round-trip back to pptx. That field is part of the import pipeline and is empty when authoring from scratch.
