# @nuxt-layers/rich-text — notes for Claude

## What this is
The platform's one rich-text format (ADR-0011, in `RESOURCES.md`): a safe subset of Markdown, its profiles (inline, block, document), the checks before saving and the safe tree readers receive. A stateless library, not a Nuxt layer. British spelling everywhere.

## Commands
- Toolchain (ADR-0013), the same locally and in CI: Node 26.13.0 (`.node-version`), Corepack 0.36.0 (`npm install --global corepack@0.36.0`, then `corepack enable`; Node 26 does not bundle it) and pnpm 12.16.1 from `packageManager`. Install scripts run only for packages in `allowBuilds` (`pnpm-workspace.yaml`; none today). CI uses only actions from `nuxt-layers` or GitHub, pinned by full commit SHA
- `minimumReleaseAge` (pnpm 12) checks every lockfile entry on every install. If it rejects entries an update locked too early, wait until they are a day old, or rebuild with `pnpm clean --lockfile` then `pnpm install`. Never weaken the setting
- `pnpm install`, then `pnpm check` (= `tsc --noEmit` + `vitest run`); run it after every code change
- `pnpm build` after changing `shared/`, and commit `dist/`: hosts install from Git, so `dist/` is what they run. CI fails when it is stale. Never add a `prepare`/`postinstall` script
- `python3 scripts/check_markdown.py` after every documentation change: headings, local links, external links only in `RESOURCES.md` and none in `CHANGELOG.md`

## Rules
- Stateless: no storage, ports, screens, configuration, Nuxt or Vue. Imports only `zod` and the Markdown parser packages; `tests/surface.test.ts` enforces it.
- Never renders HTML. Readers receive the safe tree only.
- Plain text is not rich text: there is no label profile.
- Loosening a profile, or widening what the format or its links allow, needs a new ADR. Tightening is a breaking change, stated in `CHANGELOG.md`.
- Problems are codes; never put a source's text in a problem, message or log.
- Never throws on any input; nesting is bounded by `RICH_TEXT_MAX_DEPTH`.
- Keep `CONTRACT.md` (the exports, codes and tree, stated once), the README and `capability.json` in step with the code, briefly.
- Documents follow ADR-0010: external links and verified revisions appear only in `RESOURCES.md`, pinned to a commit; every other document names the entry. `CHANGELOG.md` holds no links. `scripts/check_markdown.py` enforces both.
- Package manager: pnpm. Commit `pnpm-lock.yaml`, regenerated locally when dependencies change. While the platform is in development CI installs with `--no-frozen-lockfile` and shows any drift; it returns to a frozen install with the first beta.
