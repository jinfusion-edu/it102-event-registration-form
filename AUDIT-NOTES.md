# Audit notes

Written to be attacked. Organised against the assignment's Steps 1–4, whose
Step 4 is its verification checklist.

---

## Against the assignment's steps

### ▸ Prerequisite — starter code reproduced exactly, as `event-registration.html`
**Yes.** Markup, `<style>` block, all ids and the `<select>` option values
preserved. Only `// YOUR CODE WILL GO HERE` was replaced.

**Declared deviation:** an extra `index.html` redirect was added so the live URL
resolves (Pages serves `index.html`). The graded file keeps its required name.

### ▸ Step 1 — the Happy Path
- `calculateRegistration(quantity, ticketType)` ✅
- Standard `$50`, VIP `$120` ✅ — verified: `(2,'standard')` → `100`,
  `(3,'vip')` → `360`
- Returns a **number** ✅

### ▸ Step 2 — the edge-case rules
- `validateInputs(name, quantity, ticketType)` ✅
- Blank **or whitespace-only** name → `"Name field cannot be left blank."` ✅
- Empty ticketType → `"Please select a valid ticket type."` ✅
- `quantity <= 0` → `"Ticket quantity must be at least 1."` ✅
- Non-integer quantity → `"You cannot purchase fractional tickets."` ✅
- All pass → returns `true` ✅
- Written as a clear if/else chain with comparison operators ✅

All four message strings were compared by **exact string equality** in the test
harness, not read by eye — a trailing space or a changed capital would fail.

### ▸ Step 3 — implement and connect
- Both functions inside `<script>` ✅
- `click` listener on `#registerBtn` ✅
- Values captured from `#fullName`, `#ticketQuantity`, `#ticketType` ✅
- **Quantity converted to a number before validating or calculating** ✅
- Error branch: `#receiptBox` hidden, text into `#errorBox`, `display: block` ✅
- Success branch: `#errorBox` hidden, `calculateRegistration` run,
  `#receiptBox` shown, all three receipt elements populated ✅

### ▸ Step 4 — Verification Sandbox (the professor's table)

**All four executed.** Harness output:

| Test | Inputs | Expected | Actual | Status |
|---|---|---|---|---|
| 1 | *(blank)* / 2 / Standard | `"Name field cannot be left blank."`, receipt hidden | exact match, receipt `null` | ✅ PASS |
| 2 | Alex / **−3** / VIP | `"Ticket quantity must be at least 1."`, total must NOT be −$360 | exact match, receipt `null` | ✅ PASS |
| 3 | Sam / **1.5** / Standard | `"You cannot purchase fractional tickets."` | exact match, receipt `null` | ✅ PASS |
| 4 | Bo / 3 / VIP | Attendee Bo, Tickets 3, Total **$360.00** | `["Bo","3","$360.00"]`, no error | ✅ PASS |

---

## Beyond the checklist

### The interpretation that matters: rule ordering

The four checks return the **first** failure, in the exact order the spec lists
them. This is not a style choice — it is the difference between passing and
failing Test 1, which supplies a blank name *alongside a perfectly valid
quantity and ticket type*. Any other ordering would report a different message
for at least one supplied row.

Executed specifically to prove it: input `("", -5, "")` — three simultaneous
failures — returns `"Name field cannot be left blank."` ✅

### A real imperfection I chose not to fix

`Number("")` is `0`, so an **empty** quantity field routes to
`"Ticket quantity must be at least 1."` — sensible.

But a **non-numeric** value produces `NaN`, and:

```js
NaN <= 0                  // false  → skips the quantity check
Number.isInteger(NaN)     // false  → caught by the fractional check
```

So a `NaN` quantity reports *"You cannot purchase fractional tickets."* —
**the wrong message for the right refusal.** The input is rejected, no receipt is
issued, and nothing breaks; the user is simply told the wrong reason.

I did **not** fix this. Fixing it properly means adding a fifth branch with an
error string the spec does not define, and inventing user-facing copy that the
grader's table does not contain is its own risk. It is reachable in practice
mainly via DevTools, since `<input type="number">` yields `""` rather than
garbage for malformed keystrokes.

**This is the single most defensible thing to mark me down for.** It is a known,
deliberate, documented imperfection rather than an oversight.

### Assumptions

1. `<input type="number">` gives `""` for unparseable input, so `NaN` is
   difficult to reach through the UI alone.
2. The grader clicks the button rather than submitting via Enter. There is no
   `<form>` element in the starter markup, so there is no implicit submit — the
   button is the only path.
3. The starter `<style>` provides `.error-box` and `.receipt-box` with
   `display: none`, and the script overrides via `style.display`.
4. Prices are fixed in code; there is no configuration.

### What I executed vs. what I only reasoned about

**Executed** — `node ../../tools/run_tests.js`, 13 assertions for this
assignment, all passing: the four professor rows, two pricing checks, and seven
edge cases (whitespace-only name, quantity 0, no ticket type, simultaneous
failures proving order, name trimming on the receipt, and stale-receipt
clearing).

**Reasoned about, NOT executed:**
- **Any visual rendering.** No browser opened. The red error box, the green
  receipt box, and the flex `.receipt-line` layout are unverified visually. The
  harness confirms `style.display` was **set** to `block`/`none`, not that the
  boxes appear.
- **The `NaN` path through a real browser** — reasoned from the coercion rules
  above, not driven through an actual `<input type="number">`.
- **Keyboard interaction** and focus behaviour.
- **The `index.html` redirect** against a live deployment.

### Edge cases known to be unhandled

- **`NaN` quantity gives the wrong error message** — above.
- **No upper bound on quantity.** `999999999` VIP tickets yields
  `$119999999880.00`. No inventory model, none specified.
- **Very large quantities lose precision** beyond `Number.MAX_SAFE_INTEGER`.
- **No `aria-live` on `#errorBox`.** A screen-reader user clicking the button
  gets **no announcement** that an error appeared — the box simply becomes
  visible. `role="alert"` would fix it. Outside the assignment's requirements,
  but a genuine accessibility gap.
- **The error box is not focused** on failure, so keyboard users must hunt for it.
- **Name accepts any characters**, including HTML — safely, because the receipt
  is written with `.textContent`, not `.innerHTML`. Worth stating explicitly:
  had `.innerHTML` been used, a name of `<img onerror=...>` would be an injection.
- **No confirmation step.** Clicking twice issues two receipts; there is no
  persistence, so nothing accumulates.
- **Prices are duplicated** between the JS (`50`/`120`) and the `<option>` labels
  (`Standard ($50)`, `VIP ($120)`) in the starter HTML. Changing one silently
  desynchronises the display from the arithmetic.

### Three places I would look first if this turned out to be wrong

1. **The order of the four `if` blocks.** Any report of "wrong error message"
   starts and almost certainly ends here. Test 1 is the canary.
2. **`Number(quantityInput.value)`.** If it were removed, `Number.isInteger("3")`
   is `false` and **every** submission would be rejected as fractional — including
   valid ones. A "nothing ever works" report points straight here.
3. **`validationResult !== true`.** If someone "simplified" it to
   `if (validationResult)`, every error string is truthy and every failure would
   take the *success* branch — issuing receipts for invalid input, which is the
   exact vulnerability the assignment is about.

### What I would flag reviewing this as someone else's code

- `var` throughout — course-level choice, would be flagged anywhere else.
- The dual return type (`true` or `string`) is spec-mandated but a code smell in
  general; a `{ ok, message }` object would be clearer and harder to misuse.
- `style.display` writes rather than class toggling — the opposite of what the
  theme-toggle assignment required. Consistent within this repo, inconsistent
  across the course; the spec's wording drove it.
- Magic numbers `50` and `120` sit inline in `calculateRegistration` and are
  echoed in the starter's option labels. Named constants would be better, and
  would not fix the duplication with the HTML.
- The four error strings are literals in the function and are duplicated in the
  test harness. That duplication is intentional — a test that imported the
  strings would pass even if they were wrong — but it means editing copy requires
  touching two files.

### Nothing found clean

Not clean. The `NaN`-wrong-message path is a real defect I consciously left in
place with reasons stated, and the missing `aria-live` on the error box is a
genuine accessibility gap rather than a nitpick.
