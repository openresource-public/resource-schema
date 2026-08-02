# resource schema

JSON schemas for the structured resource format used by Open Resource. The same JSON drives slide, web, print, PPTX, and DOCX output. Resources can be imported from Oak National Academy slide decks (and other sources) or authored from scratch.

## Files

- `SCHEMA.md` — resource model overview.
- `blocks/bento.md` — the block: tile catalogue and content shape.
- `MODELS.md` — computed models (parameterised coded figures): the concept and where they appear.
- `models/<name>.md` — per-model parameter interface (one published example).
- `examples/` — real lessons in the schema, stripped of platform internals.
- `tiptap-doc-schema.json` — rich-text document schema referenced from every text field.

## Contributing

Issues welcome. The source lives in a separate repository; this one is regenerated on each schema-version release.

## Licence

MIT. See [LICENCE](LICENCE). The example lessons carry their own content licences (OGL v3, CC BY-SA 4.0) — see [examples/README.md](examples/README.md).
