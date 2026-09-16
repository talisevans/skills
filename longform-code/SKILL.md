---
name: longform-code
description: >
  Write code in long form for human readers: one idea per line, a plain-English comment above
  every block, banner headings between sections, and simple stepped logic instead of clever
  one-liners. Applies to every language (TypeScript, Python, PHP, SQL, shell). Use whenever writing
  or refactoring code, when the user says "longform", "make this readable", "spell it out",
  or invokes /longform-code.
---

# Long-form code

Code is read far more often than it is written. Every rule here trades a few extra lines for a
reader who can follow the file top to bottom without decoding anything.

## 1. Long form over condensed

Sacrifice brevity for clarity. If a construct makes the reader stop and unpack it, expand it.

- **Loops are always multi-line.** Never a `for` on one line, never a loop body on the same line
  as its header.
- **One step per statement.** Do not chain more than two operations on one line
  (`a.filter(...).map(...).sort(...)` becomes three named steps).
- **Name intermediate values.** A value used in a condition gets a name that says what it means
  (`const overCap = text.length > cap;` then `if (overCap)`), not an inline expression.
- **Prefer `if`/`else` to ternaries** for anything longer than a short assignment. Never nest a
  ternary.
- **No clever compression.** No `reduce` where a loop is clearer, no comprehension that spans
  more than one simple clause, no regex where a few string checks read plainly, no
  `x && doThing()` for control flow.
- **Small functions with one job.** When a function does three things, it becomes three
  functions with names that say what each does, called in order.
- **Types are spelled out.** In TypeScript, name interfaces for the shapes you read rather than
  casting `Record<string, unknown>` at each use.

Efficiency is not the trade. The code stays performant; only the syntax gets longer.

```python
# Bad
totals = {k: sum(r.spend for r in rows if r.seat == k) for k in seats}

# Good
# Add up the spend landing in each seat
totals = {}
for seat in seats:
    seat_rows = [row for row in rows if row.seat == seat]
    totals[seat] = sum(row.spend for row in seat_rows)
```

```typescript
// Bad
const names = users.filter(u => u.active).map(u => u.name);

// Good
// Collect the names of the active users
const names: string[] = [];
for (const user of users) {
  if (!user.active) continue;
  names.push(user.name);
}
```

```php
// Bad
$total = array_sum(array_map(fn($item) => $item['price'] * $item['qty'], array_filter($items, fn($i) => $i['inStock'])));

// Good
// Add up the cost of every item that is in stock
$total = 0;
foreach ($items as $item) {
    if (!$item['inStock']) {
        continue;
    }
    $total += $item['price'] * $item['qty'];
}
```

## 2. A comment above every block

Every block of code — a loop, a condition, a group of statements that do one thing — has a
single-line comment directly above it, in plain English, saying what the block does.

- **Plain words, no jargon.** Write for someone who knows the domain but not this file.
  "Keep the newest row for each key", not "Deduplicate via window function partitioned on PK".
- **What, not how.** The code shows how. The comment says what it achieves and, when it is not
  obvious, why.
- **One line.** If it needs two, the block is probably doing two things; split the block.
- **Never a changelog.** No "fixed 2026-07-28", no ticket numbers, no "previously this did X".
  History lives in git.
- **Docstrings on every function and class.** One or two sentences: what it does.

```typescript
// Bad
for (const share of SHARE_FILTERS) {
  const clauses = share.keys.filter(k => filters[k]?.length).map((k, i) => `${share.columns[i]} IN (SELECT unnest($f_${k}))`);
  ...

// Good
// Build one clause per weight-table filter that has values
for (const share of SHARE_FILTERS) {
  const clauses: string[] = [];
  for (const [index, key] of share.keys.entries()) {
    const values = filters[key];
    if (!values || !values.length) continue;
    clauses.push(`${share.columns[index]} IN (SELECT unnest($f_${key}))`);
  }
  ...
```

## 3. Sections with banner headings

Break every file longer than a screen into sections, each under a banner, so a reader scanning
the file sees its shape before reading a line of code.

- **Banner format:**

  ```
  // ---------------------------------------------------------------- #
  // Section title
  // ---------------------------------------------------------------- #
  ```

  In Python and shell use `#` for all three lines. In SQL use `--`. In PHP use `//` as in TypeScript.

- **Order sections the way the work flows.** Types and constants first, then helpers, then the
  steps in the order they run, then the entry point last. A module header comment that lists
  the steps as a numbered list should match the section titles.
- **Section titles are short and concrete:** "Check 2: relations and functions", "Counting and
  paging", "The consent page". Not "Utilities", "Misc", "Helpers 2".
- **A section is a unit of attention.** If a reviewer wants to check how paging works, one
  banner takes them to everything about paging.

## 4. Don't be fancy

Keep the logic simple and stepped out, one section at a time.

- **Do one thing per section, one thing per function, one thing per line.**
- **Linear over nested.** Early returns and guard clauses instead of deep nesting.
- **Explicit over implicit.** Spell out defaults (`let start = floor; if (...) start = ...`),
  name magic numbers as constants with a comment saying what they are.
- **No premature abstraction.** Two similar blocks are fine. A generic helper appears when the
  third copy would.
- **Boring is good.** If there is a plain way and a clever way, the plain way wins even when
  the clever way is shorter.

## Checklist before finishing a file

- Every loop is multi-line.
- Every block has a plain-English comment above it.
- Every function and class has a docstring.
- The file has banner sections, ordered the way the work flows.
- No line does more than one thing; no ternary is nested; no chain is longer than two calls.
- Constants are named and explained.
- Reading the file top to bottom needs no decoding.
