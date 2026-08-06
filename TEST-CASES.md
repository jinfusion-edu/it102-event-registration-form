# Test cases

The assignment supplies a four-row verification table (Step 4) and asks for test
cases in the video. The professor's four are reproduced first; further cases
were authored.

> All rows were **executed** by `node ../../tools/run_tests.js`, which loads the
> real `<script>` block from `event-registration.html`, fires the actual click
> handler, and reads the actual `#errorBox` / `#receiptBox` elements including
> their `display` state. The "Actual" column is harness output.

## Step 4 — the professor's verification sandbox

| Test | Inputs | Expected Output Behavior | Actual Output & UI Status | Pass/Fail |
|---|---|---|---|---|
| **Test 1** | Name *(blank)*<br>Qty `2`<br>Type Standard | Error display showing `"Name field cannot be left blank."`<br>Receipt must remain hidden. | Error box visible, text exactly `"Name field cannot be left blank."`<br>Receipt `display: none` | ✅ **PASS** |
| **Test 2** | Name `Alex`<br>Qty `-3`<br>Type VIP | Error showing `"Ticket quantity must be at least 1."`<br>Total must **NOT** calculate as −$360. | Error box visible, exact match.<br>Receipt hidden; `calculateRegistration` never called. | ✅ **PASS** |
| **Test 3** | Name `Sam`<br>Qty `1.5`<br>Type Standard | Error showing `"You cannot purchase fractional tickets."` | Error box visible, exact match.<br>Receipt hidden. | ✅ **PASS** |
| **Test 4** | Name `Bo`<br>Qty `3`<br>Type VIP | Perfect processing. Receipt shows Attendee: Bo, Tickets: 3, Total Price: `$360.00`. | Receipt visible → `["Bo", "3", "$360.00"]`<br>Error box hidden. | ✅ **PASS** |

Test 2 is the heart of the assignment. `-3 × $120 = -$360` is exactly the figure
the brief warns must never appear. It does not, because validation runs **before**
calculation and `calculateRegistration` is never reached.

## Pricing checks

| Call | Expected | Actual | Status |
|---|---|---|---|
| `calculateRegistration(2, "standard")` | `100` | `100` | ✅ PASS |
| `calculateRegistration(3, "vip")` | `360` | `360` | ✅ PASS |

## Edge cases

| # | Case | Expected | Actual | Status |
|---|---|---|---|---|
| E1 | Name is whitespace only — `"   "` | `"Name field cannot be left blank."` | exact match | ✅ PASS |
| E2 | Quantity `0` (boundary of `<= 0`) | `"Ticket quantity must be at least 1."` | exact match | ✅ PASS |
| E3 | No ticket type selected | `"Please select a valid ticket type."` | exact match | ✅ PASS |
| E4 | **Rule order** — blank name *and* `-5` *and* no type | Name error wins (first failure) | `"Name field cannot be left blank."` | ✅ PASS |
| E5 | Name padded — `"  Bo  "`, 1, VIP | Receipt shows trimmed `Bo` | `["Bo", "1", "$120.00"]` | ✅ PASS |
| E6 | Stale receipt — succeed, then submit a blank name | Receipt must disappear | receipt `null` | ✅ PASS |

**E1** proves the `.trim()` clause, distinct from a plain `=== ""` check.

**E2** is the boundary. `0` is neither negative nor positive, and `<= 0` must
include it — a `< 0` comparison would let a zero-ticket purchase through with a
$0.00 receipt.

**E4** is the ordering guarantee. Three simultaneous failures must produce the
*first* message, not the last-evaluated one.

**E6** is the state-leak check. Without hiding the receipt on the failure branch,
the screen would show an error and a receipt for a purchase that did not happen.

## Video walkthrough — 6 cases to demonstrate

**Normal:** Test 4 (Bo / 3 / VIP → $360.00) · a Standard purchase (2 × $50 =
$100.00) · E5 showing the trimmed name on the receipt.

**Edge:** Test 2 (−3 tickets — say out loud that the total is *not* −$360) ·
Test 3 (1.5 tickets) · Test 1 followed immediately by E6, to show the receipt
disappearing when a later submission fails.

## Explicitly NOT tested

- Any visual rendering — no browser was opened. The harness confirms
  `style.display` was **set** to `block`/`none`, not that the red and green
  boxes actually appear.
- The `NaN` quantity path through a real `<input type="number">`. Reasoned from
  coercion rules: it produces the **wrong error message** (fractional rather
  than a numeric-input error) while still correctly refusing the purchase. See
  `AUDIT-NOTES.md` — this is a known, deliberate imperfection.
- Screen-reader announcement of the error box (**expected to be absent** — there
  is no `aria-live` / `role="alert"`).
- Keyboard-only submission and focus handling.
- Extremely large quantities beyond `Number.MAX_SAFE_INTEGER`.
