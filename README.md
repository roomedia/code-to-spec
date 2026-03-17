# code-to-spec

Reverse-engineer source code into implementation-ready spec documents.

## What it does

Analyzes your codebase and produces a set of SPEC_*.md documents detailed enough for another agent (or developer) to recreate the code from scratch — without ever reading the original source.

## Install

First, add the marketplace:

```
/plugin marketplace add https://github.com/roomedia/claude-plugins.git
```

Then install the plugin:

```
/plugin install code-to-spec
```

## Usage

### Command (explicit)

```
/code-to-spec @src/feature/
/code-to-spec @src/feature/ --interactive --max-iterations 3
/code-to-spec @src/feature/ --lang ko --output-dir docs/specs/feature/
```

### Skill (natural language)

Just ask:
- "Reverse engineer this code into specs"
- "Generate spec documents from src/feature/"
- "Document this codebase for reimplementation"

The skill will ask you to confirm target, output path, iteration count, and language before proceeding.

## Flags

| Flag | Default | Description |
|---|---|---|
| `--interactive` | off | Confirm at each audit iteration |
| `--max-iterations N` | 5 | Max audit-supplement cycles |
| `--output-dir path` | `docs/specs/{feature}/` | Output directory |
| `--lang code` | auto-detect | Language for generated specs |

## Output

Generates role-specific spec documents in the output directory:

| Document | When | Content |
|---|---|---|
| SPEC_INDEX.md | Always | File manifest, resource index |
| SPEC_API.md | API present | Endpoints, DTOs, mappers |
| SPEC_MODELS.md | Models present | Domain models, enums, types |
| SPEC_COMMON.md | Shared deps | Component signatures |
| SPEC_STATE.md | State mgmt | State logic, edge cases |
| SPEC_HOST.md | Host layer | Lifecycle, navigation |
| SPEC_UI.md | UI present | Screen states, components |
| SPEC_IMPROVEMENTS.md | Always | Design issues + suggestions |

## How it works

1. **Explore** — Analyzes target code structure and dependencies
2. **Guide** — Loads spec writing conventions (built-in or project SPEC_GUIDE.md)
3. **Write** — Generates SPEC_*.md documents
4. **Audit** — Subagent attempts to implement from specs alone, finds gaps
5. **Judge** — Decides whether gaps need fixing
6. **Supplement** — Fills gaps, then re-audits

Repeats audit→supplement until specs are complete or max iterations reached.

## Customization

Place a `SPEC_GUIDE.md` in your project root, `docs/`, or `.claude/` to override default spec writing conventions. Section-level override: your guide's sections replace matching built-in sections.

## License

MIT
