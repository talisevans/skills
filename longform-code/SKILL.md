---
name: longform-code
description: >
  Write code in long form for human readers: one idea per line, a plain-English comment above
  every block, banner headings between sections, and simple stepped logic instead of clever
  one-liners. Keeps simple logic in one method instead of scattering it across many small files,
  especially in frameworks (Laravel, CodeIgniter, NestJS). Applies to every language (TypeScript,
  Python, PHP, SQL, shell). Use whenever writing
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
- **Steps inside a function, not functions inside a file tree.** When a function does three
  things, lay them out as three commented steps, in order, in the one function. Split a step
  into its own function only when it is reused, or when the function no longer fits on a screen.
  A split function stays in the same file as its only caller (see rule 5).
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

## 5. Keep simple logic in one place

A reviewer wants to read a feature in one spot. They should not need six files open to follow
the bouncing ball from a controller to a service to a helper to a constants file and back,
only to find the logic was ten lines all along. This matters most in frameworks (Laravel,
CodeIgniter, NestJS, Express, Django and the like), where it is tempting to give every small
piece its own class, provider or file.

**Keep the framework's real structure.** Use the layers the framework expects and that the
project already has: routes, controllers, models, migrations, config files, middleware,
modules. A central config file (`config/*.php`, `config.ts`, `.env`) that declares
project-wide values once is good architecture. Keep it.

**Do not split simple logic across files.** Inside that structure, the logic for one job lives
in one method, written out in commented steps. Create a new file, class or module only when
at least one of these is true:

- **It is shared.** Two or more places in the project actually call it today. Not "might one
  day".
- **The framework requires it.** A migration, a job, an event listener, a form request, a
  NestJS provider that must be injected, a model. Follow the convention, and keep each one's
  body whole.
- **It has a real architectural reason.** It wraps an external system (a payment gateway, an
  S3 client) so it can be swapped or mocked, it holds a security boundary, or it is a value
  that belongs in project-wide config because other parts of the system depend on it.
- **It is genuinely large.** The method has grown past a screen or two even after it is
  written plainly, and the piece being moved out is a complete idea with a clear name.

If none of these apply, the code stays inline.

**Constants follow the same rule.** A value used by one method is declared at the top of that
method's file, or of the method itself, with a comment. It moves to a shared constants or
config file only when other code reads it too, or when it is a setting someone would change
per environment.

**When a helper is split out for size, keep it next to its caller.** A private method in the
same class, under the same banner section, is one scroll away. A new file is a context switch.

```php
// Bad: the logic for "apply a member discount" is spread over five files
// app/Http/Controllers/OrderController.php
public function store(StoreOrderRequest $request, DiscountService $discounts)
{
    $order = Order::create($request->validated());
    $discounts->apply($order);
    return redirect()->route('orders.show', $order);
}

// app/Services/DiscountService.php
public function apply(Order $order): void
{
    if (DiscountEligibility::check($order->customer)) {
        $order->total = DiscountCalculator::calculate($order->total);
        $order->save();
    }
}

// app/Support/DiscountEligibility.php
public static function check(Customer $customer): bool
{
    return $customer->is_member && $customer->orders()->count() >= DiscountConstants::MIN_ORDERS;
}

// app/Support/DiscountCalculator.php
public static function calculate(float $total): float
{
    return round($total * (1 - DiscountConstants::RATE), 2);
}

// app/Support/DiscountConstants.php
const MIN_ORDERS = 3;
const RATE = 0.10;
```

```php
// Good: the controller keeps its framework role, the rate lives in config
// because finance changes it, and the discount reads top to bottom in one place
// app/Http/Controllers/OrderController.php

/** Number of past orders a member needs before the discount applies. */
private const MEMBER_DISCOUNT_MIN_ORDERS = 3;

/**
 * Create an order, applying the member discount when the customer qualifies.
 */
public function store(StoreOrderRequest $request)
{
    // Create the order from the validated form input
    $order = Order::create($request->validated());

    // Work out whether this customer qualifies for the member discount
    $customer = $order->customer;
    $pastOrderCount = $customer->orders()->count();
    $hasEnoughOrders = $pastOrderCount >= self::MEMBER_DISCOUNT_MIN_ORDERS;
    $qualifiesForDiscount = $customer->is_member && $hasEnoughOrders;

    // Take the discount off the total and save it
    if ($qualifiesForDiscount) {
        $discountRate = config('orders.member_discount_rate');
        $discountedTotal = $order->total * (1 - $discountRate);
        $order->total = round($discountedTotal, 2);
        $order->save();
    }

    // Send the customer to their new order
    return redirect()->route('orders.show', $order);
}
```

If a second controller later needs the same discount, that is the moment to move it into a
shared service, and the move is easy because the logic is already in one piece.

## Checklist before finishing a file

- Every loop is multi-line.
- Every block has a plain-English comment above it.
- Every function and class has a docstring.
- The file has banner sections, ordered the way the work flows.
- No line does more than one thing; no ternary is nested; no chain is longer than two calls.
- Constants are named and explained.
- Reading the file top to bottom needs no decoding.
- Every new file, class or module is shared, required by the framework, or has a real
  architectural reason. Simple logic for one job reads in one method.
- Single-use constants and helpers sit in the file that uses them.
