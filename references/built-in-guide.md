# Spec Writing Guide

## 1. Document Structure

- Split by role into focused documents (150 lines max recommended)
- Every document starts with a link to SPEC_INDEX.md:
  > See [SPEC_INDEX.md](SPEC_INDEX.md) for the full file manifest and resource index.
- Use `---` horizontal rules to separate major sections
- Number sections sequentially starting from 1
- If a document exceeds 150 lines after any phase (including supplement phases), split by sub-role (e.g., `SPEC_UI_LIST.md`, `SPEC_UI_DETAIL.md`)
- `SPEC_IMPROVEMENTS.md` is exempt from the 150-line rule

---

## 2. Required Information

Every spec document MUST include the following when applicable:

- Layout dimensions, spacing values, padding, margins
- Color tokens and font tokens — use design system names, not raw values
- ALL conditional branches — enumerate every case, not just the happy path
- ALL cases for sum types (sealed class, enum, union, tagged union, discriminated union) — list every variant with its fields
- Event/analytics definitions: event name, all parameters, data source for each parameter
- Edge cases and error handling for every operation
- State transitions: what triggers each transition, what state results
- Default values for every configurable field
- Null/empty/loading states explicitly documented

---

## 3. May Omit

- Internal implementation of shared/common components — document only the name, parameter signature, and return type
- Framework standard patterns: Dependency Injection (DI) wiring, import statements, module declarations, boilerplate
- Language/framework idioms that can be found in official documentation
- Build configuration (gradle, webpack, package.json) unless project-specific

---

## 4. Code Blocks

- Structure descriptions: use pseudo-code tree notation with `├──` and `└──` characters
- Actual code snippets: only when a specific implementation pattern is non-obvious or project-specific
- Mappings, branching logic, value comparisons: always use tables
- Application Programming Interface (API) request/response: show full schema structure, mark optional fields

---

## 5. Writing Style

- Noun-phrase endings, not full sentences (e.g., "User profile screen" not "This is the user profile screen")
- One piece of information per line
- No unnecessary modifiers or filler words
- Spell out abbreviations on first use, then use the abbreviation (e.g., "Data Transfer Object (DTO)" then "DTO")
- Use consistent terminology throughout all spec documents in a set
- Prefer tables over prose for structured data
- Use bullet lists for enumeration, not numbered lists (unless order matters)
