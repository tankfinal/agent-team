# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository scope

This repository has two distinct parts:

1. **Agent-team orchestration template** (root-level `.claude/`, `prompts/`, `scripts/`, `docs/`)
2. **Resume web app** in `resume-site/` (Vite + React + TypeScript + Tailwind)

Most changes will target one of these areas, not both.

## Important repo conventions (from `CLAUDE.md`)

- Default communication language is **Traditional Chinese** unless the user asks otherwise.
- Do **not** run `git push` unless explicitly asked.
- Do **not** modify files under `~/.claude/`.
- Do **not** install new packages/tools unless explicitly asked.

For develop-team behavior, preserve the intended role boundaries:
- `pm` is the user-facing coordinator and does requirement clarification/task delegation.
- `be`/`fe` implement.
- `pm` is intentionally configured without code-writing tools.

## Common commands

Run commands from repository root unless noted otherwise.

### Agent-team operations

- Rebuild **develop** team session (kills old `tank-dev`, verifies expected agent files, starts lead with `prompts/dev-lead.md`):
  - `./scripts/dev-rebuild.sh`
- Attach to develop session:
  - `./scripts/dev-attach.sh`
- Rebuild **consult** team session (kills old `tank-consult`, verifies expected agent files, starts lead with `prompts/consult-lead.md`):
  - `./scripts/consult-rebuild.sh`
- Attach to consult session:
  - `./scripts/consult-attach.sh`

### Resume site (`resume-site/`)

- Install dependencies:
  - `cd resume-site && npm install`
- Start local dev server:
  - `cd resume-site && npm run dev`
- Build production bundle:
  - `cd resume-site && npm run build`
- Lint:
  - `cd resume-site && npm run lint`
- Preview built site:
  - `cd resume-site && npm run preview`

### Testing status

- There is currently **no test script configured** in `resume-site/package.json`.
- Single-test command is therefore **not available** yet in the current setup.

## High-level architecture

## 1) Agent-team orchestration (root)

- `.claude/agents/*.md` defines teammate personas, tools, and behavioral constraints.
- `prompts/dev-lead.md` and `prompts/consult-lead.md` define lead bootstrapping flow:
  - create a named team,
  - spawn teammates with `team_name`,
  - then route user messages by mention/default rules.
- `scripts/*-rebuild.sh` is the operational entrypoint:
  - validates expected agent files + lead prompt,
  - recreates tmux session,
  - launches `claude` with prompt file contents as initial message.
- `scripts/*-attach.sh` is only for reconnecting to an existing tmux session.
- `.claude/settings.json` enables agent-team mode and tmux teammate layout.

Critical coupling when adding/renaming teammates:
- update `.claude/agents/<name>.md`
- update corresponding `prompts/*-lead.md` spawn/routing content
- update `EXPECTED_AGENTS` in matching rebuild script

Without keeping all three aligned, rebuild checks or runtime routing will break.

## 2) Resume site app (`resume-site/`)

- Entry: `src/main.tsx` mounts `<App />`.
- Composition root: `src/App.tsx` is a section-assembler page shell:
  - `Navbar` + ordered content sections (`Hero`, `About`, `Experience`, `Skills`, `Education`, `Contact`) + `Footer`.
- Components are currently static/presentational; section data is colocated in component files (arrays/constants), with no external API/state layer.
- Styling is utility-first via Tailwind v4 imported in `src/index.css`; `src/App.css` is intentionally minimal.
- Build/lint toolchain:
  - Vite + React plugin (`vite.config.ts`)
  - TypeScript project references (`tsconfig*.json`)
  - Flat ESLint config (`eslint.config.js`)

Deployment coupling:
- Vite `base` is set to `/agent-team/` for GitHub Pages pathing.
- CI workflow `.github/workflows/deploy-resume.yml` builds only `resume-site/` and publishes `resume-site/dist`.
- If repo name or Pages path changes, update both Vite `base` and deployment expectations together.
