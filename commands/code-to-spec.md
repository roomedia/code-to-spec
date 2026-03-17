# code-to-spec: Reverse-engineer code into implementation specs

Generate spec documents from source code. The specs are detailed enough
for another agent to recreate the code without reading the original.

## Usage

```
/code-to-spec @target
/code-to-spec @target --interactive --max-iterations 3 --lang en
/code-to-spec @target --output-dir path/to/output
```

## Flags

| Flag | Default | Description |
|---|---|---|
| `--interactive` | off | Confirm at each audit iteration |
| `--max-iterations N` | 5 | Max audit-supplement cycles |
| `--output-dir path` | `docs/specs/{feature}/` | Output directory |
| `--lang code` | auto-detect | Language for generated specs |

## Execution

1. Parse ARGUMENTS to extract target path and flags.
2. Validate the target path exists. If not, halt with:
   "Error: target path `{path}` not found. Check the path and try again."
3. Read `references/workflow.md` from this plugin's directory.
4. Execute the workflow with these inputs:
   - `{target}` = parsed target path
   - `{output-dir}` = `--output-dir` value or `docs/specs/{feature-name}/`
   - `{max-iterations}` = `--max-iterations` value or 5
   - `{interactive}` = true if `--interactive` present, else false
   - `{lang}` = `--lang` value or "auto-detect"
