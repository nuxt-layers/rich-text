# @nuxt-layers/rich-text

> **In short:** a small TypeScript library that lets any layer accept formatted text, such as bold, links, lists and tables, safely. It defines one format, a safe subset of Markdown; checks text against it before it is saved; and turns saved text into a **safe tree** that a screen can draw with no risk of injected HTML. It stores nothing and draws nothing.

Governed by ADR-0011 ([Resources](RESOURCES.md)).

**Using it?** Read [CONTRACT.md](CONTRACT.md), the complete contract, and nothing else ([changelog](CHANGELOG.md)).

## Who uses it

| Who | Uses | For |
|---|---|---|
| A layer that stores formatted text, such as `publisher` | `richTextSchema`, `checkRichText` | Refusing anything beyond a field's profile on every save |
| The same layer, when answering | `parseRichText` | Returning the safe tree, never HTML |
| `ui` | The tree's types | Drawing it |
| `content-editor` | The profiles | Offering only the tools a field allows |

## A library, not a layer

| | |
|---|---|
| **Is** | A package imported like `zod`, with no state, ports, screens or configuration |
| **Is not** | A Nuxt layer: it has no `nuxt.config.ts`, pages, routes, components or storage |
| **Why** | It runs inside other layers' server code and tests, where Node loads only JavaScript from installed packages. So its source is in `shared/` and its compiled JavaScript in `dist/`, committed because hosts install it from Git |

## What it holds

| Part | Gives |
|---|---|
| **The format** | A safe subset of Markdown: CommonMark with GFM tables and strikethrough. Raw HTML is always text |
| **Profiles** | What each formatted field may use: inline, block or document |
| **Checks** | `checkRichText` and `richTextSchema`: what a save must refuse, and which documents and assets a source names |
| **The safe tree** | `parseRichText`: nodes that carry no markup and only the format's links, for any renderer to draw |

Plain text is not rich text. A label, name or title is an ordinary string with a length limit, edited with a standard form control.

## Profiles, links and exports

The [contract](CONTRACT.md) states them once: what each profile allows, which links survive, every export, the problem codes and the safe tree.

## Guardrails

- Never renders HTML; returns safe trees only.
- Loosening a profile, or widening what the format allows, needs a new ADR.
- Imports only `zod` and the Markdown parser; `tests/surface.test.ts` holds it to that.

## Development

`pnpm install`, then `pnpm check` (type check and tests). After changing `shared/`, run `pnpm build` and commit `dist/`: hosts install from Git and receive it as committed. `python3 scripts/check_markdown.py` checks the documentation.
