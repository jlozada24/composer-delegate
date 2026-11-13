# composer-delegate

A Claude Code skill that makes Claude hand work to Cursor's Composer 2.5 model through the `cursor-agent` CLI. Claude stays the lead: it decides what to delegate, writes the handoff, and reviews the result.

For each request Claude picks one of three paths:

- **Full** — Composer does the whole task; Claude reviews.
- **Split** — Composer does the reading, implementing, and testing; Claude makes the decisions in between.
- **None** — the task is mostly judgment, so Claude does it directly.

The criteria for each are in [SKILL.md](SKILL.md).

## Requirements

- [Claude Code](https://claude.com/claude-code)
- The Cursor CLI, installed and logged in:

  ```bash
  curl https://cursor.com/install -fsS | bash
  cursor-agent login
  cursor-agent --list-models   # composer-2.5 should be listed
  ```

## Install

```bash
git clone https://github.com/jlozada24/composer-delegate.git
mkdir -p ~/.claude/skills
ln -s "$PWD/composer-delegate" ~/.claude/skills/composer-delegate
```

Start a new Claude Code session afterward so the skill is picked up.

## Use

```
/composer-delegate rename the `fetchUser` helper to `loadUser` everywhere and run the tests
```

The skill only runs when you invoke it, and covers that one request.

## Things to know

- Edit runs use `--trust --force`, which lets Composer run shell commands in your workspace without asking. Use it in a Git repository so every change is reviewable in `git diff`.
- The model name `composer-2.5` is written into `SKILL.md`. If Cursor renames the model, change it there.
- Composer usage counts against your Cursor plan.
