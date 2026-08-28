# angular-skills

A collection of reusable AI agent skills for Angular/TypeScript frontend
development (ag-grid, NGXS), plus a few general-purpose workflow skills (PR
review, memory discipline, skill authoring). Originally consolidated from an
Angular 13 HR frontend project.

Each skill is a self-contained `SKILL.md` file: YAML frontmatter (`name`,
`description`, `license`) followed by a markdown body with the actual rules,
examples, and gotchas. This format works with any tool that can load a
markdown file into an agent's context — Claude Code skills, a system prompt
section for any other LLM, or just documentation a human reads before
writing code.

## Skills

| Skill | What it's for |
| --- | --- |
| [`angular-best-practices`](skills/angular-best-practices/SKILL.md) | Component lifecycle, clean code, lazy loading with permissions, `*ngFor trackBy`, forced UI component conventions |
| [`angular-component-structure`](skills/angular-component-structure/SKILL.md) | Class-member ordering, state isolation (no business services in components), NGXS-only data access |
| [`ag-grid-best-practices`](skills/ag-grid-best-practices/SKILL.md) | Ivy compatibility, theming, grid state persistence |
| [`ag-grid-style`](skills/ag-grid-style/SKILL.md) | This project's own concrete AG-Grid v18 implementation convention (column resize, async auto-sizing, sanitization) |
| [`ngxs-best-practices`](skills/ngxs-best-practices/SKILL.md) | Typed actions, no action barrels, selector memoization, single-source-of-truth state |
| [`typescript-best-practices`](skills/typescript-best-practices/SKILL.md) | Strict typing, form initialization, module structure, RxJS error handling |
| [`api-design-best-practices`](skills/api-design-best-practices/SKILL.md) | HTTP pagination, parameter validation, centralized endpoint URLs |
| [`bitbucket-pr-resolver`](skills/bitbucket-pr-resolver/SKILL.md) | Fetch Bitbucket PR review comments, triage them (real bug / false positive / tech debt), fix and reply |
| [`chained-pr`](skills/chained-pr/SKILL.md) | Split oversized changes into chained PRs that protect review focus |
| [`cognitive-doc-design`](skills/cognitive-doc-design/SKILL.md) | Writing docs that reduce cognitive load |
| [`comment-writer`](skills/comment-writer/SKILL.md) | Tone/voice guidance for PR feedback, issue replies, and review comments |
| [`judgment-day`](skills/judgment-day/SKILL.md) | Blind dual-judge adversarial code review protocol |
| [`memory-protocol`](skills/memory-protocol/SKILL.md) | Discipline for using Engram-style `mem_*` persistent-memory tools |
| [`engram-cloud-setup`](skills/engram-cloud-setup/SKILL.md) | Configuring a project for Engram Cloud memory sync |
| [`engram-save-sync`](skills/engram-save-sync/SKILL.md) | Save-and-sync workflow for Engram memory |
| [`project-initializer`](skills/project-initializer/SKILL.md) | Bootstrap SDD + memory + skill-registry setup for a new project |
| [`rdd-defect-workflow`](skills/rdd-defect-workflow/SKILL.md) | Receipt-driven development defect/recovery workflow |
| [`skill-creator`](skills/skill-creator/SKILL.md) | Meta-skill: how to write a new `SKILL.md` |
| [`skill-improver`](skills/skill-improver/SKILL.md) | Audit and upgrade existing skills |
| [`systemic-issue-triage`](skills/systemic-issue-triage/SKILL.md) | Attack issues by root class instead of one-by-one |
| [`trello-doc`](skills/trello-doc/SKILL.md) | Document development process to Trello |
| [`work-unit-commits`](skills/work-unit-commits/SKILL.md) | Plan commits as reviewable work units |

## Usage

**Claude Code**: drop a skill folder into `.agents/skills/` (project-local)
or your global skills directory (user-level) and it becomes available.

**Any other model/tool**: paste the relevant `SKILL.md`'s body into your
system prompt or context before starting work, or point your agent at the
raw file URL.

## Notes on portability

`ag-grid-style` and `angular-component-structure` were authored specifically
for one codebase and encode that project's exact conventions (component
names like `<app-quick-search>`, service names like `SweetAlertHelper`,
`ag-theme-balham`, `calc(100vh - Xpx)` grid sizing verified against 163
actual files in that repo). Treat these as a worked example of a real,
internally-consistent convention set rather than universal rules — verify
against your own codebase's actual patterns before applying them elsewhere.

## License

Apache-2.0 / MIT per each `SKILL.md`'s own frontmatter (mixed — some skills
here were authored globally, others are project-specific).
