# mind-spec

**Markdown is the runtime.**

`.mind/` is a version-controlled folder where an agent keeps everything it knows — identity, memories, skills, and context — as plain markdown files. No databases. No vector stores. No custom SDKs. Just files an LLM can read, write, and reason about.

- **Spec:** [`SPEC.md`](./SPEC.md)
- **Website:** <https://mind.modelbound.co>
- **CLI:** [`ModelBound/mind-cli`](https://github.com/ModelBound/mind-cli)
- **MCP server:** [`ModelBound/mind-mcp`](https://github.com/ModelBound/mind-mcp)
- **Examples:** [`ModelBound/mind-examples`](https://github.com/ModelBound/mind-examples)

## Why

Agent tools invented one bespoke context format after another — system prompts, `.cursorrules`, `AGENTS.md`, custom memory stores, proprietary vector databases. Each one locks knowledge into a single vendor.

`.mind/` is the smallest common denominator: a folder of markdown that any agent — Claude, Cursor, Copilot, or something you built yourself — can read today with no library.

## Try it in 60 seconds

```bash
npx @modelbound/mind-cli init
```

You get:

```
.mind/
  INDEX.md
  self.md
  memory/
  skills/
  context/
```

Point any LLM at the folder and it can start working.

## Design principles

1. **Human-readable first.** A non-technical user should understand a `.mind/` folder after five minutes of reading.
2. **Git-native.** Every change is a commit. `git log` is provenance.
3. **No required runtime.** A static `.mind/` folder is useful on its own. Tools make it faster, not mandatory.
4. **LLM-agnostic.** The format does not depend on any single model provider.
5. **Minimal surface.** Fewer rules means fewer reasons to disagree.

## Contributing

The spec evolves through RFCs opened as issues on this repository. Minor versions are additive; breaking changes require a numbered RFC and a migration note.

## License

Apache 2.0. See [`LICENSE`](./LICENSE).

## Stewardship

The `.mind` specification is stewarded by [ModelBound](https://modelbound.co), which also operates a managed registry and proposal review UI on top of the open spec. The spec itself is vendor-neutral and will always be.
