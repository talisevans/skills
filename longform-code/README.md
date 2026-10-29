# longform-code

A [Claude Code](https://claude.com/claude-code) skill that makes Claude write code for human
readers: one idea per line, a plain-English comment above every block, banner headings between
sections, and simple stepped logic instead of clever one-liners.

It does not make code slower or less correct. It makes it longer, and easier to read six months
later.

## What it changes

Without the skill:

```typescript
const names = users.filter(u => u.active).map(u => u.name);
```

With the skill:

```typescript
// Collect the names of the active users
const names: string[] = [];
for (const user of users) {
  if (!user.active) continue;
  names.push(user.name);
}
```

The five rules, in full in [`SKILL.md`](SKILL.md):

1. **Long form over condensed.** Loops are multi-line. No chains longer than two calls. No nested
   ternaries. Intermediate values get names. Steps laid out inside one function.
2. **A comment above every block.** One line, plain English, what the block does. Docstrings on
   every function and class. Never a changelog.
3. **Sections with banner headings.** Files are broken into titled sections, ordered the way the
   work flows, so a reader sees the shape of a file before reading a line of it.
4. **Don't be fancy.** Linear over nested, explicit over implicit, boring over clever.
5. **Keep simple logic in one place.** Keep the framework's structure and the project's config
   files, but do not scatter one job across controllers, services, helpers and constants files.
   A new file is earned by being shared, required by the framework, or architecturally needed.

Examples are given in TypeScript, Python and PHP. The rules apply to any language.

## Install

This skill lives in the [talisevans/skills](https://github.com/talisevans/skills) collection.
Skills are a folder containing a `SKILL.md`; put the folder where Claude Code looks for them.

**For every project on your machine:**

```bash
git clone https://github.com/talisevans/skills.git
cp -r skills/longform-code ~/.claude/skills/
```

**For one project only** (commit it so the whole team gets it):

```bash
git clone https://github.com/talisevans/skills.git
cp -r skills/longform-code .claude/skills/
```

Claude Code picks the skill up on the next session. Check with `/longform-code` at the prompt.

## Use

Claude applies the skill when it judges it relevant — the description names the triggers
("longform", "make this readable", "spell it out") — or when you invoke it:

```
/longform-code
```

To apply it to existing code:

```
/longform-code rewrite src/parser.ts
```

To review a file against the checklist at the end of the skill:

```
check src/parser.ts against longform-code
```

## Make it the default

A skill loads when invoked or judged relevant. To have Claude follow it for all code, all the
time, add one line to your `CLAUDE.md` — `~/.claude/CLAUDE.md` for everything you do, or the
project's `CLAUDE.md` for one repository:

```markdown
# Code style
Always follow the longform-code skill when writing or refactoring code. Invoke it via the
Skill tool before the first code edit in a session.
```

`CLAUDE.md` is loaded into every session, so the instruction is always present and the skill's
full text is pulled in the first time code is written.

## Adapt it

The skill is plain Markdown. Fork it and change what you disagree with:

- The banner format in rule 3 (`// ---- #`) is a house style; swap it for yours.
- The "no chain longer than two calls" line in rule 1 is a threshold, not a law.
- Add examples in your own language beside the TypeScript, Python and PHP ones.

Keep the `description` in the frontmatter accurate: it is what Claude reads to decide when the
skill applies.

## Licence

MIT, with the rest of the [skills](https://github.com/talisevans/skills) collection.
