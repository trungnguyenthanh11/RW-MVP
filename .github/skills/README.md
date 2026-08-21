# Skills

Narrow, task-focused instruction files an agent pulls in only when the task needs them.
All BA skills live in `awesome-agents/.github/skills/` — **reference** them from agent personas; don't copy their content inline.

## Available skills

| Skill | Used by | Trigger | Path |
|---|---|---|---|
| **ba-design-thinking-ideation** | BA | Problem is vague / pre-requirements; need ideation | `awesome-agents/.github/skills/ba-design-thinking-ideation/SKILL.md` |
| **ba-functional-decomposition** | BA | Break product into capabilities → features → stories | `awesome-agents/.github/skills/ba-functional-decomposition/SKILL.md` |
| **ba-generate-user-story** | BA | Quick single story + acceptance criteria from an idea | `awesome-agents/.github/skills/ba-generate-user-story/SKILL.md` |
| **ba-user-story-authoring-review** | BA | Full backlog pass — author, split, review with INVEST checks | `awesome-agents/.github/skills/ba-user-story-authoring-review/SKILL.md` |
| **ba-process-modelling-bpmn** | BA | Workflow needs a process diagram to clarify handoffs | `awesome-agents/.github/skills/ba-process-modelling-bpmn/SKILL.md` |
| **ba-wireframe-mockup-generation** | BA | Story needs a visual — text wireframe first, HTML if needed | `awesome-agents/.github/skills/ba-wireframe-mockup-generation/SKILL.md` |
| **ba-vision-scope-document** | BA | Project needs a Vision & Scope document | `awesome-agents/.github/skills/ba-vision-scope-document/SKILL.md` |
| **create-development-plan** | Dev | Task breakdown / dev spec generation | `awesome-agents/.github/skills/create-development-plan/SKILL.md` |
| **code-review** | Dev | Before merging any change | `.github/skills/code-review/SKILL.md` |
| **testing-test-strategy** | Test | Define testing scope, risks, approach, and quality gates | `awesome-agents/.github/skills/testing-test-strategy/SKILL.md` |
| **testing-analyze-requirements** | Test | Assess requirement completeness and testability | `awesome-agents/.github/skills/testing-analyze-requirements/SKILL.md` |
| **testing-design-test-case** | Test | Generate traceable, risk-based test cases | `awesome-agents/.github/skills/testing-design-test-case/SKILL.md` |
| **testing-review-test-case** | Test | Review test-case coverage, consistency, and quality | `awesome-agents/.github/skills/testing-review-test-case/SKILL.md` |
| **testing-generate-page-object** | Test | Build reusable Playwright page objects | `awesome-agents/.github/skills/testing-generate-page-object/SKILL.md` |
| **testing-implement-automation** | Test | Implement Playwright automation from approved test cases | `awesome-agents/.github/skills/testing-implement-automation/SKILL.md` |
| **testing-analyze-bug** | Test | Reproduce, assess impact, and identify root-cause area | `awesome-agents/.github/skills/testing-analyze-bug/SKILL.md` |

## How to use a skill

1. Open the skill's `SKILL.md` in your prompt (e.g. `#file:awesome-agents/.github/skills/ba-generate-user-story/SKILL.md`).
2. The skill file describes its workflow, output contract, and any templates to follow.
3. Reference it from the persona in `agents/` — don't paste its contents into the persona file.

## Adding a skill

One folder per skill under `.github/skills/`, named in `kebab-case`. Its `SKILL.md` must state:
- **What it covers** — one or two sentences
- **Trigger** — concrete condition for loading it
- **The rules** — short and imperative; if it exceeds a screen or two, split it
