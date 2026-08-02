# Examples

Real lessons in the schema, to read alongside [SCHEMA.md](../SCHEMA.md). Binary assets are stripped: image `src` values read `"PLACEHOLDER"`, while the surrounding envelope (alt text, captions, attribution) is real.

| File | Subject | Key stage | Source |
|---|---|---|---|
| [viking-york.json](./viking-york.json) | History | KS2 | Oak National Academy |
| [pythagoras.json](./pythagoras.json) | Maths | KS3/4 | Maths Genie |
| [algebra-quadratics.json](./algebra-quadratics.json) | Maths | KS4 | Original (CC0) |
| [trig-and-exponential-graphs.json](./trig-and-exponential-graphs.json) | Maths | KS4 | Maths Genie |

`viking-york.json` is "How we know so much about Viking York" — 30 blocks across 4 sections. A prose-led lesson exercising `text`, `image`, `dialogue`, `options`, `items`, and `glossary` tiles, with stems, beats, and auto-marked choices. Imported from an Oak slide deck and structured by an agent.

`pythagoras.json` is "Pythagoras" — a 29-block exam question set. A maths lesson exercising `model` tiles (×22, computed triangle and quadrilateral figures), `working` tiles (stepped written maths), and auto-marked `gaps`, with worked solutions carried as `explanation` blocks.

`algebra-quadratics.json` is "Quadratics: graphs and factorising" — a compact 7-block lesson across 3 sections. Exercises `model`, `table`, `working`, `matching`, `ordering`, and `gaps` tiles, including a rubric-marked task and a staged model build.

`trig-and-exponential-graphs.json` is "Trig and Exponential Graphs" — an 18-block exam question set. Exercises `model` tiles (computed function graphs) and a **multi-block question**: blocks `q5` and `q5b` share `question: "q5"` (a stimulus block of four graphs plus the markable matching task), the schema's question-group form (see [SCHEMA.md § Multi-block questions](../SCHEMA.md#multi-block-questions)).

## Licence

Each example carries its source attribution in `meta.credits`.

`viking-york.json` is Oak National Academy content under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/):

> © Oak National Academy 2024. Produced in partnership with Pearson. Licensed on the Open Government Licence v3.0, except where otherwise stated. See Oak terms and conditions.

`pythagoras.json` contains questions from [Maths Genie](https://www.mathsgenie.co.uk), © Maths Genie Limited, licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); each question block carries the full attribution in `meta.attribution`.

`algebra-quadratics.json` is original content dedicated to the public domain under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).

`trig-and-exponential-graphs.json` contains questions from [Maths Genie](https://www.mathsgenie.co.uk), © Maths Genie Limited, licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); the attribution rides in `meta.credits`.

The rest of this repo stays MIT (see [LICENCE](../LICENCE)).
