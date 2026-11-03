---
name: bitbucket-pr-resolver
description: "Trigger: Bitbucket PR, PR comments, PRISM findings, review comments, resolve comments, PR number in the softgbq workspace. Fetch, triage, fix, reply and resolve review comments, then record new rules in the project skills."
license: Apache-2.0
metadata:
  author: rh-team
  version: "2.1"
  scope: project
  stack: "Bitbucket REST 2.0, PowerShell scripts, workspace softgbq, base branch develop"
---

## Activation Contract

Load when the user asks to resolve, review or answer comments of a Bitbucket pull request, or gives a PR number. Needs three per-developer scripts (`get_pr_comments.ps1`, `post_pr_reply.ps1`, `resolve_comment.ps1`) in `<your-scripts-folder>`; never commit them. Each reads the API token from the `BITBUCKET_API_TOKEN` user environment variable and aborts without it; set it once with `[Environment]::SetEnvironmentVariable('BITBUCKET_API_TOKEN', '<token>', 'User')`. Token scopes: `read:pullrequest:bitbucket`, `write:pullrequest:bitbucket`.

## Hard Rules

- Frontend PRs live in repo `frontend_rh`, the scripts' default; for a backend PR pass `-repo rh_new` to every call.
- Verify each claim in the code before accepting it. Wrong premise: reply with evidence and leave it open. Half right: fix the wording, reply with the precise limit, leave the debt open.
- Fixes follow strict TDD when it is on; run one Karma spec per call (`ng test --include <spec>`), and report RED then GREEN.
- Reply and leave open for false positives and tech debt. Resolve a comment only after its fix is pushed.
- Commit and push are separate decisions: commit on the PR branch, ask before pushing, then publish replies and resolve in one pass.
- Updating the project skills is part of resolving the comments: never ask whether to do it and never skip it. It is committed before the push, so replies and resolutions (step 7) come after it.
- Conventional commits; no AI attribution and no `Co-Authored-By`.
- Spanish text with accents breaks from the shell: write the `.ps1` as UTF-8 **with BOM** and run it exactly once.
- Never print the API token, and never write it in a script, a `.ps1` for replies or any file. Do not grep or `cat` the scripts' parameter or auth lines: they once printed it on screen (PR 1888).

## Decision Gates

| Comment is | Action |
|------------|--------|
| Real bug or simple typo | Fix, reply `Corregido en <sha>: ...`, resolve after the push |
| False positive | Reply with evidence, leave open |
| Architecture / tech debt | Reply with the justification, leave open |
| Rule someone could break again | Add it as a rule to the project skill that covers the area (step 6) |
| No new rule | Still record the PR, the comment and the verdict in `references/gotchas.md`, Triage (step 6) |

## Execution Steps

1. Fetch the active comments with `get_pr_comments.ps1 -prId <ID>`.
2. Read `.agents/skills/README.md` and the `SKILL.md` that match the files each comment touches.
3. Write a checklist: comment ID, file, diagnosis, decision, and for each one the skill change: a new rule (which skill) or "no new rule" and why.
4. Present the plan and wait for explicit approval (code and skill changes) before editing; that approval covers step 6.
5. Apply the fixes, run the tests, commit.
6. Update skills, always and without asking, once the fixes are committed. Classify every comment (fixed, false positive or debt). If someone could break its rule again in this repo, add one verifiable rule to the skill in the README table that covers the area (create `rh-<topic>/SKILL.md` following the README standard when none does) and cite the proof (file, spec, PR). If it adds no rule, record the PR, the comment and the verdict in `references/gotchas.md`, Triage. Every PR therefore leaves at least one skill change. Bump `metadata.version`, commit with `docs(skills): ...` and refresh your tool's skill index (README, "Registry") before asking to push.
7. After the user approves the push, post replies and resolve with the other two scripts.

## Output Contract

Final report: comments fixed, replied, and left open (with reasons), commits, tests run with results, and the skills updated or created. Edge cases and API details: `references/gotchas.md`.
