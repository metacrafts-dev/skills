# skills

Handy AI skills. My skill bank for Claude Code — I add custom skills under
[`custom-skills/skills/`](custom-skills/skills/).

## Install

In Claude Code:

```
/plugin marketplace add metacrafts-dev/skills
/plugin install custom-skills@skills
```

Or with the skills CLI:

```
npx skills add metacrafts-dev/skills
```

## Skills

| Skill | What it does |
|---|---|
| [`telegram-ping`](custom-skills/skills/telegram-ping) | Ping me on Telegram when work is blocked or a round of work ends |
| [`refactor-prompt`](custom-skills/skills/refactor-prompt) | Turn a vague refactor ask into a precise, delegable prompt |
| [`research-prompt`](custom-skills/skills/research-prompt) | Turn a research question into a precise, delegable prompt |
| [`orchestrate`](custom-skills/skills/orchestrate) | Multi-model delegate orchestration with lanes, relays and gates |
| [`refine-and-execute-prompt`](custom-skills/skills/refine-and-execute-prompt) | Refine a prompt then decompose and execute it with verification |

## Adding a skill

Create a folder under `custom-skills/skills/<skill-name>/` with a `SKILL.md`
(name, description frontmatter + instructions). That's it — the plugin picks up
every folder there. Commit and push, then reinstall the plugin or run
`/plugin` → update to see it on other machines.
