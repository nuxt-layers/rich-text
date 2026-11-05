# Resources

> **In short:** every external architectural document `rich-text` relies on, with the revision reviewed. This is the only file that links outside the repository (ADR-0010). Software dependencies are resolved as ADR-0008 says, not here.

All rows were checked on 5 November 2026 against `nuxt-layers/architecture-rules` at commit `7a2d5c3`.

| Document | Relied on for |
|---|---|
| [ADR-0008 — Independent layer delivery](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/decisions/ADR-0008-independent-layer-delivery.md) | Consumers follow `master`; release tags suspended |
| [ADR-0010 — Documentation style](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/decisions/ADR-0010-documentation-style.md) | Which document holds what |
| [ADR-0011 — Shared rich text](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/decisions/ADR-0011-shared-rich-text-and-configurable-copy.md) | The format, its profiles, the safe tree, and what loosening needs |
| [ADR-0013 — Development toolchain baseline](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/decisions/ADR-0013-development-toolchain-baseline.md) | Node, pnpm and CI |
| [Layer standard](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/layer-standard.md) | A library's place among the layers, `capability.json` and versioning |
| [Privacy](https://github.com/nuxt-layers/architecture-rules/blob/7a2d5c34d1943f7b92edbc41f54b4201236cbfb5/docs/privacy.md) | §5: a stateless library keeps no storage register; its contract says it persists nothing |
