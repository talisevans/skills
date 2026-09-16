# skills

Skills for [Claude Code](https://claude.com/claude-code). Each folder is one skill: a
`SKILL.md` with frontmatter naming it and describing when it applies, plus the instructions
Claude follows once it is loaded.

| Skill | What it does |
|---|---|
| [`longform-code`](longform-code/) | Write code for human readers: one idea per line, a plain-English comment above every block, banner headings between sections, simple stepped logic. |

## Install

**The whole collection**, for every project on this machine (only if `~/.claude/skills` does
not already exist):

```bash
git clone https://github.com/talisevans/skills.git ~/.claude/skills
```

**One skill**, copied into a skills folder you already have:

```bash
git clone https://github.com/talisevans/skills.git
cp -r skills/longform-code ~/.claude/skills/
```

For one project only, use `.claude/skills/` inside the repository instead of `~/.claude/skills/`
and commit it, so the whole team gets it.

Claude Code picks new skills up on the next session. Check with `/<skill-name>` at the prompt.

## Licence

MIT. See [LICENSE](LICENSE).
