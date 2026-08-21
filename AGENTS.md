# RW-MVP — Core Agent

> Workshop entry point. Every mob member points Copilot Agent mode at this file (or `.github/copilot-instructions.md`).
> Read `state/current-stage.md` first, then load the persona for the current stage.

## Project identity

- **Pod name:** RW-MVP Workshop Pod
- **Problem:** Build a working MVP web application end-to-end across the SDLC using AI
- **Target users:** Workshop participants (BA, Dev, Test roles)
- **Tech stack:** Next.js (App Router) + TypeScript + Prisma + PostgreSQL + Tailwind CSS — deployed on Vercel
- **Product brief:** `docs/product-brief.md` (create during Part 02)

## How this harness routes work

1. Read `state/current-stage.md` for the current SDLC stage, **mode** (`mob` or `breakout`), and active lead role(s).
2. Load the matching persona from `.github/agents/` (routing table below). In `breakout` mode, each subgroup loads only its own persona.
3. Pull in any narrow skill from `.github/skills/` that the persona's Skills section names — see `.github/skills/README.md`.
4. Apply the stack-specific rules in `instructions/tech-stack.md`.
5. Consult `knowledge/domain-notes.md` for domain facts and decisions already made.
6. Follow the stage flow and Definition of Done in `workflow/sdlc-workflow.md`.

## Stage → agent routing

| Stage | Active lead | Mode | Persona file | Primary output |
|---|---|---|---|---|
| Requirements | BA | Breakout OK | `.github/agents/ba.agent.md` | `docs/requirements/` |
| Design / SAD | Dev | Breakout OK — parallel with Requirements | `.github/agents/developer.agent.md` | `docs/design/` |
| Task breakdown | Dev | Full mob only | `.github/agents/developer.agent.md` | `docs/dev-spec.md` |
| Build | Dev + Test | Full mob only | both agent files | working code + tests |
| Test pass | Test | Breakout OK to draft strategy; full mob to review | `.github/agents/tester.agent.md` | `docs/testing/` |
| Polish / demo prep | All | Full mob only | *(this file)* | stable build, demo script, `docs/token-log.md` |

**Breakout discipline:** a breakout never advances `current_stage` past `design` on its own. The full team reconvenes, reviews both outputs, then flips `mode: mob` and `current_stage: task-breakdown`.

## Global working agreements

- **Store every artifact in the repo** — requirements, design, dev spec, test strategy, and test cases are committed, never left only in chat.
- **Review AI output** — a human reads every change before it's committed.
- **Commit often**, in small working slices.
- **Scope guard** — anything not in the MVP goes to `docs/product-brief.md` out-of-scope list.
- **Token efficiency** — prefer scoped context (`#file`, `#selection`); log notable spends in `docs/token-log.md`.

## Workshop checklist

- [ ] `state/current-stage.md` initialized to `requirements` / `mob` / `BA`
- [ ] `docs/product-brief.md` written with problem, target users, and MVP scope
- [ ] `instructions/tech-stack.md` filled with your chosen stack details
- [ ] `knowledge/domain-notes.md` seeded with at least the core glossary
- [ ] `.github/skills/README.md` lists any narrow skills your pod is using
