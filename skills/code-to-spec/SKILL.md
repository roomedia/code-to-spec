---
name: code-to-spec
description: >
  Use when the user asks to reverse-engineer code, generate specs from source,
  create implementation-ready documentation from existing code, or wants to
  document a codebase so another agent can recreate it. Triggers on phrases like
  "reverse engineer this", "generate spec", "document this for reimplementation",
  "code to spec", "create spec documents from code". Collects target, output path,
  iteration count, and language preference via conversation, then executes the
  shared workflow.
---

# code-to-spec

Reverse-engineer source code into implementation-ready spec documents.

## When to Use

- User asks to reverse-engineer, document, or create specs from existing code
- User wants documentation detailed enough to recreate code from scratch
- User mentions "code to spec", "reverse engineer", "legacy documentation"

## When NOT to Use

- User wants to write NEW code from a design (that's a different workflow)
- User wants API documentation or user-facing docs (not implementation specs)

## Process

### Step 1: Collect Parameters

Ask the user ONE question at a time to determine:

1. **Target** — "What file or directory should I reverse-engineer?"
   - If user already provided a path or @ reference, confirm it
   - Validate the path exists before proceeding

2. **Output directory** — "Where should I save the spec documents?"
   - Suggest default: `docs/specs/{feature-name}/`
   - Accept user override

3. **Iteration count** — "How many audit-supplement cycles? (default: 5)"
   - Brief explanation: "Each cycle has a subagent try to implement from
     the specs and report gaps. More cycles = more thorough but slower."
   - Accept default or user override

4. **Interactive mode** — "Want to review and approve each audit cycle? (default: no)"
   - Brief explanation: "If yes, I'll show you the gaps found and ask
     whether to continue supplementing."

5. **Language** — Only ask if project language is ambiguous.
   - If project clearly uses one natural language, auto-detect silently
   - If ambiguous: "The project has mixed languages. What language should
     the spec documents be written in?"

### Step 2: Execute Workflow

Read `references/workflow.md` from this plugin's directory and execute it with
the collected parameters:
- `{target}` = confirmed target path
- `{output-dir}` = confirmed output directory
- `{max-iterations}` = confirmed iteration count
- `{interactive}` = confirmed interactive preference
- `{lang}` = confirmed language or "auto-detect"
