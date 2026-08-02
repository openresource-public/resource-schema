# The block

A block is a collection of **tiles**, each carrying exactly one payload — prose, a figure, a table, a dialogue, or a question interaction — optionally with a **beat timeline** that reveals tiles one click at a time. Each tile's `cell` arranges the collection on a bento grid of 12 columns × unbounded rows (the grid gives the block type its fixed discriminator, `'bento'`). The grid applies fully on the slide surface; elsewhere it is advisory — linear surfaces (print, narrow viewports) read tiles in list order. One block can hold a worked example beside its diagram, a multi-part question, a discussion split into voices.

The thirteen tile kinds:

| kind | carries | markable |
|---|---|---|
| `text` | prose (rich text) | — |
| `working` | stepped written maths, relation-aligned | — |
| `image` | a bound (or searched-for) image | — |
| `model` | a computed figure — see [MODELS.md](../MODELS.md) | with a `responseMode` |
| `video` | an embedded video | — |
| `dialogue` | named speakers, ordered turns | — |
| `table` | rows × cells | gap cells |
| `options` | a single / multi / true-false choice | yes |
| `matching` | two columns to pair or categorise | yes |
| `ordering` | items to sequence | yes |
| `gaps` | a cloze with typed blanks | yes |
| `items` | a uniform collection grid | — |
| `glossary` | a term–definition list | — |

## Shape

```ts
{
  type: 'bento',
  content: {
    stem?: { text: Doc },      // optional leading prompt above the grid
    tiles: Tile[],             // at least one
    beats?: Op[][]             // reveal timeline; each beat is ops fired together
  },
  explanation?: BlockId[]      // revealed worked answer (see SCHEMA.md#marking)
}

type Tile = {
  id: string,                  // stable; beats and marking key on it
  cell: { col: number, row: number, w: number, h: number },  // 1-based, col+w-1 ≤ 12
  part?: string,               // 'a' / 'b' / … — groups tiles into question parts
  caption?: Doc,               // small caption under the tile
  speaker?: string,            // attribute the tile to a dialogue speaker
  marking?: Marking,           // markable kinds only — see below
  responseMode?: 'place-point' | 'select-point' | 'select-region' | 'run-value'
    | 'run-query' | 'run-program' | 'fill-derived' | 'place-derived' | 'construct'
    | 'place-line' | 'show-inequality' | 'mark-cross' | 'plot-curve' | 'draw-histogram'
    | 'draw-journey' | 'shade-region' | 'line-of-best-fit' | 'read-off'
    | 'transform-curve' | 'draw-tangent',
                               // marked model tiles only — answer-on-the-model interaction;
                               // the full per-mode contract is in the JSON schema

  // exactly ONE of (the kind is derived from which key is present):
  text?: Doc,
  working?: { lines: WorkingLine[], result?: { lines: string[] }, lhsAlign?: 'right' | 'left' },
                               // stepped written maths, relation-aligned — see below
  image?: Image,
  model?: PrimitiveRef,
  video?: { url: string, altText: string, title?: string },
  dialogue?: { speakers: Speaker[], turns: Turn[] },
  table?: { rows: { cells: Cell[] }[], hasHeaderRow?: boolean, hasHeaderCol?: boolean },
  options?: { variant: 'single' | 'multi' | 'boolean', options?: Item[], justification? },
  matching?: { variant?: 'pairs' | 'categorise', leftColumn: Item[], rightColumn: Item[] },
  ordering?: { items: Item[] },
  gaps?: { blanks: { id: string, input?: 'math' }[], template: ({ text } | { blank })[], wordBank?: string[] },
  items?: { items: Item[], cols?: number, numbered?: boolean },
  glossary?: { items: { term: Doc, definition: Doc }[] }
}
```

Tiles never overlap; cells are validated. Reading order for linear surfaces (print, export) is tile list order, not grid position.

The entries of `options` / `matching` / `ordering` / `items` and a table `Cell` share the text-and-figure envelope:

```ts
type Item = { id: string, text?: Doc, image?: Image, primitive?: PrimitiveRef }
                                       // an item's model slot is named `primitive`
type Cell = { id?: string, text?: Doc, image?: Image, gap?: boolean }
                                       // gap: content withheld — revealed by a beat, or typed into when the tile is marked
```

## Payload shapes

The remaining named types.

A **working** is a vertical sequence of lines — gutter label, relation-aligned statement, reason note — with an optional answer box. Alignment is derived: each aligned statement splits at its first top-level relation token; authors never write alignment markup. `result.lines` entries are LaTeX strings, one per centred answer line.

```ts
type WorkingLine = {
  statement: Doc,                  // the line: one paragraph of text and/or math runs
  label?: string,                  // gutter lead-in — "(1)", "(1)+(2)", "×100"
  note?: Doc,                      // right-column reason ("angles in a triangle sum to 180°")
  rule?: 'above' | 'below',        // thick combining rule (elimination results, column sums)
  align?: boolean,                 // join the relation-alignment group; default true —
                                   //   set false for prose lines and full-width givens
  arcs?: { from: string, to: string, side?: 'above' | 'below' }[]
                                   // term-link arcs between marked terms in the statement
}
```

A **dialogue** is named speakers and ordered turns:

```ts
type Speaker = { id: string, name?: string, avatar?: Image }
type Turn    = { id: string, speaker: string, text: Doc, about?: string }
                                   // speaker: a speakers[].id
                                   // about: a sibling tile id the bubble points at
```

## Marking

A question is a block with a markable tile — `options`, `matching`, `ordering`, `gaps`, `table` (gap cells), or `model` (with a `responseMode`) — flagged `role: 'question'` on its envelope ([SCHEMA.md](../SCHEMA.md#blocks)). The key lives on the tile at `marking` (never at block level), shaped per kind:

| kind | `correctAnswer` |
|---|---|
| options `single` | option id |
| options `multi` | option id[] |
| options `boolean` | boolean |
| matching | `{ leftId: rightId }` |
| ordering | item id[] in the intended order |
| gaps / table | `{ blankId or cellId: answer or answer[] }` |
| model | per `responseMode` — place-point `{x, y}`, select-point label(s), select-region id[], run-value `{ value }`; the run/derive/draw modes `{ derived: true }` (the model self-checks against what it derives) |

A `gaps` blank whose answer is mathematical notation (a π form, a power, a surd, a rearranged formula) declares `input: 'math'`: the pupil answers it on a math keyboard, and its key is an accepted set of LaTeX forms matched structurally — cosmetic notation (braces, `\times` vs `\cdot`, `\frac` vs `/`) folds; a mathematically different form never matches. The tile-level `match` continues to govern the other, typed blanks.

The boolean variant of `options` may carry `justification` — secondary "justify your answer" options, each tagged with the truth value it is correct for. Several markable tiles tagged `part: 'a' / 'b' / …` make a multi-part question, marked separately. A question too large for one block spans several via the block-envelope `question` group id — see [SCHEMA.md § Multi-block questions](../SCHEMA.md#multi-block-questions). The full `Marking` union (`auto` / `rubric` / `none`) is in [SCHEMA.md](../SCHEMA.md#marking).

## Beats

`beats` is the block's reveal timeline. Each beat is an array of ops that fire on one click:

```ts
type Op = {
  tile: string,                      // target tile id
  op?: 'reveal' | 'hide' | 'stage',  // default 'reveal'
  stage?: number,                    // stage: advance a model to steps[stage]
  paragraph?: number,                // sub-address: one paragraph of a text tile
  line?: number,                     //   one statement line of a working tile
  turn?: number,                     //   one dialogue turn
  tableRow?: number, tableCol?: number,  // one table cell
  item?: number,                     //   one collection item
  answer?: true                      // this reveal is the answer — withheld on student-facing surfaces
}
```

A tile targeted by any `reveal` starts hidden; untargeted tiles are visible throughout. `hide` + `reveal` in one beat swaps two tiles sharing a cell (pose a misconception, then correct it). Progressive prose is one `text` tile revealed paragraph by paragraph — not a tile per line. A working reveals the same way, line by line; a tile-level op reveals the rest, including its result.

## Example

A posed question: figure, prompt, and an auto-marked choice revealed in beats.

```json
{
  "id": "blk-line-equation-mcq",
  "type": "bento",
  "role": "question",
  "content": {
    "stem": { "text": { "type": "doc", "content": [
      { "type": "paragraph", "content": [{ "type": "text", "text": "What is the equation of the line shown on the grid?" }] }
    ] } },
    "tiles": [
      {
        "id": "fig",
        "cell": { "col": 1, "row": 1, "w": 6, "h": 4 },
        "model": {
          "ref": "graphing/coordinate-grid@4",
          "params": {
            "axes": { "x": [-5, 5], "y": [-5, 5] },
            "lines": [{ "y": 3, "occlude": true }]
          },
          "altText": "A horizontal line drawn across a four-quadrant coordinate grid, its equation hidden."
        }
      },
      {
        "id": "q",
        "cell": { "col": 7, "row": 1, "w": 6, "h": 4 },
        "options": {
          "variant": "single",
          "options": [
            { "id": "A", "text": { "type": "doc", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "x = 3" }] }] } },
            { "id": "B", "text": { "type": "doc", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "y = 3" }] }] } },
            { "id": "C", "text": { "type": "doc", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "y = -3" }] }] } }
          ]
        },
        "marking": { "mode": "auto", "correctAnswer": "B" }
      }
    ],
    "beats": [
      [{ "tile": "fig" }],
      [{ "tile": "q" }]
    ]
  }
}
```
