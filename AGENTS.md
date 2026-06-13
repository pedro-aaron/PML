# AGENTS.md — PML

This file orients Codex when working on this sub-project. It is complementary to the workspace `AGENTS.md` at `/mnt/c/Users/pedro/Documents/atideas/`.

## What this project is

A **pure Markdown specification, in English**. There is no code, build system, tests, dependencies, or infrastructure. The spec defines **PML (PEPPO Markup Language)**: a bracket-tag markup language for LLMs to emit interactive UI components inside a chat. Current version: **v1.6**.

This is not an implementation repo. Renderers, parsers, SDKs, and tooling live in other (future) repos.

## The only normative artifact: `README.md`

The **only normative artifact** of this project is `README.md` (in English). That is the specification. Everything else is auxiliary.

## File layout

| File / dir          | Role                                                                                          |
| ------------------- | --------------------------------------------------------------------------------------------- |
| `README.md`         | **The specification.** In English. This is the only thing that matters to keep current.      |
| `LICENSE`           | MIT.                                                                                          |
| `.gitattributes`    | Minimal git configuration.                                                                    |
| `internals/`        | User-owned directory for auxiliary documents: tracking, QA, translations, analysis, notes, decisions, etc. **Not part of the spec.** |

## Policy for `internals/`

`internals/` is where **every intermediate or auxiliary document** the user requests, or that is generated during work, must live. Examples: translations of the README into other languages, strategic analyses, QA rubrics, design notes, market comparisons, decision logs, experimental drafts.

**Rules:**

- **Do not place new `.md` files at the project root.** Any additional document goes to `internals/`.
- **Do not mix `internals/` with the spec.** `README.md` must not reference files in `internals/`; they are private to the user.
- **Treat `internals/` contents as internal context**, not as a public audience. Do not mention them in commits, PR descriptions, or responses the user might share externally.
- If the user asks "create a document X", default to placing it in `internals/`, unless they explicitly ask for the root.

## Project positioning

PML is **internal PEPPO infrastructure**, not a public competitive specification. Its justifiable value is reducing token cost in the PEPPO ChatApp. It is not promoted as an open standard.

**Rules that derive from this positioning:**

- **Do not bump to v1.7+ without a working renderer.** Iterating the spec without code hides lack of validation. If the user asks for new features (cancel-label, form-section, cross-field validation, stepper, etc.), remind them of this rule and confirm before proceeding.
- **Do not promote the spec externally** (GitHub release, blog post, social media, communities) unless the user explicitly asks.

## Operating rules

### Editing `README.md`

1. Any normative change bumps the minor version (v1.6 → v1.7) and requires:
   - A new entry in §12.1 (versioning).
   - A new migration appendix (E, F, G, ...).
   - Updating the footer `_PML v1.x — PEPPO Platform — Month 2026_`.
   - Updating the edit notice at the top of the document (right under the version subtitle).
2. Cosmetic changes (typos, formatting) do not bump the version.
3. **Do not touch Appendices C, D, E** except to fix errors: they are immutable historical records of past migrations.

### PML language conventions

When adding examples or documenting new components, respect:

- **kebab-case** in component, attribute, and ID names (`[a-z][a-z0-9-]*`). Never camelCase, snake_case, or uppercase.
- **No JSON.** No `< >`. No HTML.
- **Bracket syntax:** `[PML:type ...]` containers; `[PML:type ... /]` self-closing (with a space before `/]`).
- **Interactions:** `[[INTERACTION id="..." value="..."]]` for single-value; `[[INTERACTION id="..." field-x="..." field-y="..."]]` for multi-field (v1.6).
- **Double quotes** preferred on attribute values.
- **Reserved values** (`confirmed`, `cancelled`, `true`, `false`, etc.) are **never translated**.

## What NOT to do

- **Do not create renderers, parsers, SDKs, or tooling** in this repo. The spec stands alone.
- **Do not add `form` features** (cancel button, styles, form-section, cross-field validation). They are explicitly deferred to v1.7+ and blocked by the "no bump without renderer" rule.
- **Do not publish anything externally** (GitHub release, blog post, social media) unless the user asks.
- **Do not iterate the spec on your own initiative.** Versioning without code hides lack of validation; wait for the user to request concrete changes.
- **Do not create new `.md` documents at the root.** They go to `internals/`.

## Workspace context

PML is one of several sub-projects under `/mnt/c/Users/pedro/Documents/atideas/`. The others (`Peppo/`, `abeone/`, `atideas.com/`, `_experimentos/`) have their own stacks and `CLAUDE.md`. PML is the protocol that feeds the PEPPO ChatApp (`Peppo/Peppo_android/`); integration and tenant/auth logic live in `Peppo/`, not here.

**Peppo multi-tenant (F1–F7):** PML wire format is unchanged by tenancy. The Android client sends `X-Peppo-Account-Id` on API calls; runner isolation is by `account_id` server-side. When reasoning about conversations or agent turns, read `../Peppo/CLAUDE.md` and `../abeone/tasks/runner/README.md` §15.0 — not this spec.

For general workspace context see `/mnt/c/Users/pedro/Documents/atideas/CLAUDE.md`.
