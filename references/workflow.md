# code-to-spec Workflow

This file is the shared workflow executed by both the `/code-to-spec` command and the `code-to-spec` skill. Do not modify the phase structure.

## Inputs

- `{target}` — file or directory to reverse-engineer
- `{output-dir}` — where to write SPEC_*.md files (default: `docs/specs/{feature}/`)
- `{max-iterations}` — max audit-supplement cycles (default: 5)
- `{interactive}` — whether to confirm at each iteration (default: off)
- `{lang}` — output language override (default: auto-detect)

## Pre-flight Validation

Before starting any phase:

1. Verify `{target}` path exists. If not:
   "Error: target path `{target}` not found. Check the path and try again."
2. Verify `{target}` contains source files. If empty or no recognized source files:
   "Error: no source files found in `{target}`. Scanned for: [list extensions tried]. Try a broader target path."
3. Create `{output-dir}` if it does not exist.

## Phase 1: Exploration

Dispatch an Explore agent (`Agent` tool with `subagent_type="Explore"`) to analyze the target code and its dependencies.

**Prompt the Explore agent to examine these layers** (only those that exist):
1. Entry points (Fragment, Activity, Controller, route handler, main function)
2. State management (ViewModel, Store, Redux, Vuex, Context, MobX)
3. UI / View layer (Compose, SwiftUI, React component, Vue template, XML layout)
4. Domain models (data class, struct, interface, enum, type alias, protocol)
5. API / Network (endpoints, DTOs, response schemas, GraphQL queries)
6. Data layer (Repository, Mapper, caching strategy, ORM, database access)
7. Shared components / utilities (only those the target code depends on)

**Language detection** (for spec output language):
1. Read project README — identify dominant natural language
2. Sample code comments from up to 10 files in `{target}` — identify language
3. Check up to 20 recent git commit messages
4. Majority vote across these signals. If inconclusive, default to English.
5. If `{lang}` is set (not "auto-detect"), use that instead and skip detection.

Store the detected language as `{detected-lang}` for use in Phase 3.

## Phase 2: Guide Resolution

1. Search for `SPEC_GUIDE.md` in: project root, `docs/`, `.claude/`
2. Read `references/built-in-guide.md` from this plugin's directory.
3. If project `SPEC_GUIDE.md` found, merge at **section level**:
   a. Parse both guides into sections by heading
   b. Sections present in both: **project guide replaces** the entire built-in section
   c. Sections only in built-in guide: keep as-is
   d. Sections only in project guide: append after built-in sections
4. If no project `SPEC_GUIDE.md` found: use built-in guide as-is.

The merged guide governs document structure, required information, and writing style for Phase 3.

## Phase 3: Document Generation

Generate role-specific SPEC_*.md files in `{output-dir}/`. Write all documents in `{detected-lang}`.

**Document list is dynamically determined** from Phase 1 exploration results. Only generate documents for layers that exist:

| Document | Condition | Content |
|---|---|---|
| `SPEC_INDEX.md` | Always | Index of all spec files, source file manifest, resource list |
| `SPEC_API.md` | API/network layer exists | Endpoints, request/response schemas, mappers, caching |
| `SPEC_MODELS.md` | Domain models exist | Models, enums, sum types, type aliases, extension functions |
| `SPEC_COMMON.md` | Shared component dependencies | Component signatures, presentation models, parameter types |
| `SPEC_STATE.md` | State management exists | State shape, state logic, side effects, edge cases |
| `SPEC_HOST.md` | Host/container layer exists | Lifecycle management, logging, navigation, deep linking |
| `SPEC_UI.md` | UI layer exists | Screen state branches, component details, layout specs, responsive behavior |
| `SPEC_IMPROVEMENTS.md` | Always | Design issues found during analysis + improvement suggestions |

**150-line rule:** If any document exceeds 150 lines, split by sub-role (e.g., `SPEC_UI_LIST.md`, `SPEC_UI_DETAIL.md`). `SPEC_IMPROVEMENTS.md` is exempt from this rule. This rule applies after every phase including Phase 6 supplements.

Follow the merged guide from Phase 2 for all document structure, content requirements, and writing style.

## Phase 4: Audit

**Source-code isolation:** The audit agent receives ONLY the `{output-dir}/` path, never the `{target}` path. This is a prompt-level constraint — the agent is instructed to read spec files only. If the agent reads source code despite instructions, the audit result is invalid but non-destructive.

Dispatch audit subagent (`Agent` tool with `subagent_type="general-purpose"`):

```
You are an implementation agent. You must write code using ONLY the spec
documents, without access to the source code.

1. Read all SPEC_*.md files in {output-dir}/.
2. Assume you will write each file listed in SPEC_INDEX.md's file manifest.
3. Find points where you get stuck:
   - High: impossible to implement from docs alone
   - Medium: can guess but risk of error
   - Low: reasonable default exists
4. Do NOT report information already present in the docs. Cross-check ALL documents.
5. General knowledge from official framework/language docs is NOT a gap.
   Only report absence of project-specific knowledge (custom classes, internal
   utilities, project conventions).

Output format:
### [Severity] Item title
- File being written
- Missing information
- Why implementation is blocked
```

If the audit subagent fails to respond, retry once. If it fails again, halt with error:
"Error: audit subagent failed to respond after retry. Check agent availability."

## Phase 5: Judgment

Record audit results in `{output-dir}/SPEC_MISSING.md` and decide:

| Condition | Action |
|---|---|
| High + Medium + Low = 0 | Terminate → go to Completion |
| `{interactive}` is true | Show results to user, ask whether to continue |
| Current iteration >= `{max-iterations}` | Terminate → go to Completion (remaining items stay in SPEC_MISSING.md) |
| Otherwise | Proceed to Phase 6 |

## Phase 6: Supplement

Address each item in SPEC_MISSING.md:

- **Items needing more source analysis:** Dispatch Explore agent (`subagent_type="Explore"`) to investigate specific code, then update the relevant SPEC_*.md.
- **Items needing design decisions:** Choose a reasonable default, document the choice and rationale in the spec.
- **Completed items:** Move to a "Resolved" table in SPEC_MISSING.md.

After supplementing, check the 150-line rule again (split if needed), then return to Phase 4.

## Completion

Output:

```
[code-to-spec complete]
- Target: {target}
- Output: {output-dir}/
- Iterations: {N}
- Language: {detected-lang}
- Remaining gaps: High {n}, Medium {n}, Low {n}
- Documents:
  - SPEC_INDEX.md
  - SPEC_API.md
  - ...
```

## Error Handling

| Scenario | Behavior |
|---|---|
| Target path does not exist | Halt: "Error: target path `{target}` not found." |
| No source files in target | Halt: "Error: no source files found in `{target}`." |
| Language auto-detection inconclusive | Default to English, note in completion output |
| Explore agent returns empty results | Halt: "Error: exploration found no code structure. Try a broader target." |
| Audit subagent fails to respond | Retry once, then halt with error |
| Output directory not writable | Halt: "Error: cannot write to `{output-dir}`. Check permissions." |
