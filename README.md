# skills

My personal agent skills, distributed as an installable skills repo.
I use them with my current setup: [ghostty](https://ghostty.org), [herdr](https://herdr.dev), and [pi](https://github.com/earendil-works/pi).

## Install

```bash
npx skills add gwynnnplaine/skills
```

Pick the skills and agents you want in the interactive prompt, or go headless:

```bash
# all skills, global, pi
npx skills add gwynnnplaine/skills --skill '*' --global --agent pi --yes
```

Skills install into your agent's skill directory (`~/.pi/agent/skills` for pi).
Update later with `npx skills update`.

## Skills

| Skill | What it does |
| --- | --- |
| `coding-standards` | TypeScript coding standards and design taste: domain modeling, typed failures, deep modules, parsing, async, tests, contracts — routed by topic. |
| `commit` | git commit workflow + terse Conventional Commits messages, applied with git. |
| `design-patterns` | Book-grounded guide for choosing, comparing, and reviewing GoF design patterns, with TypeScript examples. |
| `grill-me` | Interview you relentlessly about a plan until shared understanding, one question at a time. |
| `grill-with-types` | Grilling session that produces type signatures as the design artifact before implementation. |
| `herdr` | Control herdr from inside it — workspaces, tabs, panes, agent waits, and spawn recipes over the socket CLI. |
| `never-trust-green` | Lock existing behavior with a characterization test, then sabotage-check the net before changing it. |
| `pi-pool` | Delegate a batch of work items to a pool of pi workers in herdr panes — worktree per worker, branch per item, review left to you. |
| `review` | Deep, read-only, standards-backed code review of local changes, a commit/range, or a GitHub PR, grounded in the coding-standards non-negotiables. |
| `tdd` | Test-driven development with a red-green-refactor loop. |
| `teach` | Teach a new skill or concept within the workspace. |
| `type-design` | Turn resolved decisions into a type system — signatures, seams, error channels — before implementation. |
| `type-review` | Score an existing change's type system 0–5 and label it fast or lasts. |
| `typescript-meta` | TypeScript decision meta-skill for implementation, review, debugging, and refactoring. |

## Credits

Structure and distribution approach inspired by [mattpocock/skills](https://github.com/mattpocock/skills).

The `coding-standards` skill and the `review` rewrite are lovingly stolen from [dmmulroy/skills](https://github.com/dmmulroy/skills). [@dmmulroy](https://github.com/dmmulroy) uses Pi and Herdr, like me. I'll take that as a good sign.
