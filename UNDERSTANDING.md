# Understanding: pbakaus/impeccable

- Original project: https://github.com/pbakaus/impeccable
- Fork: https://github.com/Kilganon725/impeccable
- Licence: Apache-2.0 (verified on the original repository page on 2026-10-31)
- Default branch: main
- Scale on 2026-10-31: 97,074 stars, 5,716 forks, 2,600 commits, 93 releases
- Trending on 2026-10-31: GitHub Trending daily, with 2,300 stars gained that day — the largest daily gain among that day's top 8 trending repositories (checked against thedotmack/claude-mem at 1,140, Renhuai123/ziwei-doushu at 1,126, mattpocock/skills at 876, CopilotKit/OpenBot at 706, and the rest below 1,000).
- This note was added to the Kilganon725 fork on 2026-10-31. The original author and licence are unchanged; nothing here claims authorship of the original project.

## What it is

Impeccable is a design vocabulary for a coding agent. Instead of asking an agent to
"make it pop" and hoping it guesses what that means, you give it named, narrow
commands — `polish`, `bolder`, `critique`, `audit` — each of which loads a playbook
written for that one move and points it at one target, such as a pricing card or a
landing-page hero section.

It is strongest on interfaces that already exist: point it at a page or a component
and iterate. It can also build new pages, but the project itself says that is the
harder mode and that a single section reviewed closely beats a whole page glanced at.

## Why it is popular

AI-generated interfaces converge on the same defaults: Inter for everything,
purple-to-blue gradients, cards nested in cards, grey text on coloured backgrounds,
and a rounded-square icon tile above every heading. Developers recognise that look
instantly as "AI slop", and it ships because nobody gave the agent a better
vocabulary or a reviewer.

Impeccable attacks both halves of that problem at once:

1. **Steering words.** Vague taste ("make it nicer") becomes a concrete instruction
   with a defined scope, so the agent stops guessing.
2. **A reviewer that does not need a model.** A deterministic slop detector with
   62 rules flags the tells above, plus quality problems such as cramped padding
   and skipped headings. No LLM call and no API key are needed for detection, so it
   can run in a terminal, in CI, or as a hook on every UI file an agent edits.

The third reason is distribution: one install covers Claude Code, Cursor, Codex,
GitHub Copilot, Devin, Gemini CLI, Grok Build, OpenCode, Antigravity, Kiro, Trae
and more, with the skill committed per tool at the repository root, so a team does
not have to standardise on one agent to use it.

## How it works at its core

### 1. Context files: `PRODUCT.md` and `DESIGN.md`

Run `/impeccable init` once per project. It reads the code, asks only what it
cannot infer (who the product is for, what it needs to do, the constraints), and
writes `PRODUCT.md`. Every later command reads that file. Run
`/impeccable document` to capture the existing visual system in `DESIGN.md`, so
changes stay on brand instead of drifting with each prompt.

### 2. Twenty-four named commands

The commands fall into four groups:

- **Steer:** `bolder`, `quieter`, `distill`, `colorize`, `typeset`, `layout`,
  `animate`, `delight`, `overdrive` — amplify, tone down, strip, colour, set type,
  fix spacing, add motion, add personality, or go for shaders and scroll effects.
- **Review:** `critique` scores a design against usability heuristics and ranks
  what to fix first; `audit` checks accessibility, performance, theming and
  responsive behaviour, with findings ranked P0 (blocking) to P3 (polish).
- **Finish:** `polish`, `harden`, `clarify`, `adapt`, `optimize`, `onboard` — the
  final pass, error/i18n/overflow edge cases, clearer copy, screen-size adaptation,
  loading performance, and first-run/empty states.
- **Iterate live and plan:** `live` lets you pick an element in the browser and
  cycle through variants in place; `generate` produces variants of a named element;
  `shape` plans UX before code; `init`, `document` and `extract` set up and
  distil the design system. (`craft` is a deprecated alias for a plain build request.)

A frequently used command can be pinned, e.g. `/impeccable pin audit` gives a
standalone `/audit` shortcut.

### 3. The deterministic slop detector

The detector is the part that needs no model. It runs three ways:

- **In the agent**, as a design hook that scans every UI file the agent edits and
  hands the findings back before the agent calls the work done. The project says
  this is where most slop gets caught, because otherwise the detector only runs
  when somebody remembers to run it.
- **In CI or a terminal**, with no agent at all:
  `npx impeccable detect src/`, `npx impeccable detect https://example.com`
  (rendered-page rules, such as low contrast and tiny text, need a rendered page),
  `npx impeccable detect --json .` for machine-readable output, and
  `npx impeccable detect --github src/` for GitHub Actions annotations.
  Exit code 2 means findings; exit code 1 means a target could not be scanned.
- **In the browser**, as a Chrome extension overlaying the same findings on a
  live page.

Rules can be silenced deliberately rather than ignored silently: allow a brand
font everywhere with `npx impeccable ignores add-value overused-font Inter
--reason "Brand font"`, or disable one rule for one file with a comment such as
`<!-- impeccable-disable overused-font: exported brand doc -->`.

### 4. Live mode

`/impeccable live` opens the running dev server (Vite, Next, SvelteKit, Astro,
Nuxt or plain HTML) with an overlay. Select an element, pick a direction, and
Impeccable hot-swaps variants in place; accepting one writes it back to source.
The project warns that live mode edits a local checkout only, and that it should
not be injected into a deployed site or used to weaken a production content
security policy.

### 5. How it is shipped

Node is only needed for `npx`. The skill itself runs a self-contained engine
binary that ships next to it or downloads once into `~/.impeccable/bin/`. Each
supported tool gets its own committed build at the repository root
(`.claude/`, `.cursor/`, `.codex/`, `.agents/`, and so on), and there is a manual
copy route with no Node at all: copy the tool's skill folder into the same place
in your project or into the global skills folder. That manual copy skips the
design hook and subagents; running `npx impeccable install` restores them.

## Technology stack

- **Skill and command layer:** Markdown skill definitions, one per coding tool,
  plus plugin manifests for Claude Code, Cursor, Codex, Copilot and others.
- **Detector and engine:** Rust (the repository root carries `Cargo.toml`,
  `Cargo.lock` and a `crates/` directory) with a self-contained engine binary.
- **CLI and distribution:** Node/`npx` installer (`npx impeccable install`,
  `npx impeccable update`), plus a browser bundle and a VS Code extension.
- **Live mode and demos:** a browser bundle and a demo app (`demos/landing-demo`),
  with dev-server support for Vite, Next, SvelteKit, Astro and Nuxt.
- **No model dependency for detection:** the 62-rule detector runs locally, which
  is why it can also run in CI without an API key.

## Where it fits

- A team whose agent-generated UI keeps coming back looking generic, and which
  wants a shared, reviewable vocabulary instead of ad-hoc taste prompts.
- A project with an existing interface that needs iterative review: run
  `critique` on one component, fix the ranked findings, then `audit` before ship.
- CI quality gates for front-end work: fail or annotate on detector findings,
  with explicit ignores for genuine brand choices.
- Multi-agent shops: the same skill, installed per tool, keeps design decisions
  consistent whether the developer uses Claude Code, Cursor, Codex or Copilot.

## Limitations

- **It reviews and steers; it does not replace design judgement.** A high
  `critique` score is not proof a page is good, and a detector pass is not proof
  it is accessible in practice — rendered-page rules only run against a rendered
  page, and source files (JSX/TSX) get pattern rules only.
- **New-page generation is the weaker mode.** The project is explicit that
  iterating on an existing interface works better than building from nothing.
- **The hook is the whole point, and it is easy to skip.** A manual skill copy
  installs no hook, and three tools (Codex, Grok Build, Hermes Agent) need an
  extra approval step before the hook runs. Without the hook, detection depends
  on somebody remembering to run it.
- **Live mode is local-only by design.** It must not be pointed at a deployed
  site, and applying copy edits runs the project's optional
  `impeccable:manual-edit-validate` script with the user's permissions, so that
  script deserves a read before use in an unfamiliar repository.
- **Scale of the repository itself.** With 2,600 commits, many tool adapters, a
  browser bundle, an extension and a Rust engine, the full codebase is far more
  than a one-day read. This note therefore scopes the understanding to the
  documented core — commands, context files, detector and live mode — rather
  than claiming a line-by-line review of every adapter.
- **Taste is still encoded by its authors.** The 62 rules embody one view of good
  design; a brand that deliberately uses, say, Inter everywhere must maintain
  explicit ignores, or the detector will keep flagging a deliberate choice.

## What this fork changes, and why

This fork adds two things in a single commit, and nothing else:

1. **This understanding file** (`UNDERSTANDING.md`), so the fork records why the
   project was chosen and how its core works, instead of being a bare fork.
2. **`ZH-QUICKSTART.md`** — a Chinese quick-start that translates the verified
   install, init, critique/audit and detector commands from the upstream README
   into a short, runnable sequence, plus a worked example of reviewing one
   component (a pricing card) end to end. The reason for this change is practical:
   the upstream README is English-only, and a Chinese-speaking developer should
   be able to go from install to a first `critique` and a first detector run
   without translating the whole README first. No upstream code, rule or default
   is changed, and the Apache-2.0 licence and original authorship are untouched.
3. **`covers/2026-10-31-cover.png`** — a cover image made for this daily entry,
   themed on the project and dated 2026-10-31, so the day's fork is identifiable
   at a glance.

## Sources checked on 2026-10-31

- Original repository page (description, licence Apache-2.0, default branch main,
  stars/forks/commits/releases): https://github.com/pbakaus/impeccable
- Upstream README sections used: Why, How it works (24 commands, live mode),
  Three ways to run the detector, Installation, Project files and git, and the
  per-tool skill-folder table.
- GitHub Trending daily page, 2026-10-31: https://github.com/trending?since=daily
- Kilganon725 repository list, checked before forking: no existing
  Kilganon725/impeccable fork, and none of the day's top 8 trending projects had
  already been forked under this account except the three earlier daily picks,
  which do not appear in the day's top 8.
