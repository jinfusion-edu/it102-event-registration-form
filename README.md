# The Bulletproof Event Registration Form

IT102 · Introduction to Programming · Seattle Colleges

An event ticket registration form built around **edge-case engineering** — the
validation is the assignment, not the arithmetic.

## What it does

Collects a name, a ticket quantity and a ticket type. On submit it validates the
inputs, and either shows a specific error message or produces a receipt with the
calculated total.

Standard tickets are $50 each, VIP $120 each.

## Files

| File | Role |
|---|---|
| `event-registration.html` | **The assignment deliverable.** Starter markup + the `<script>` block. |
| `index.html` | A one-line redirect, so the live URL works (Pages serves `index.html`). |

## How to run it

```bash
git clone https://github.com/jinfusion-edu/it102-event-registration-form.git
cd it102-event-registration-form
```

Open `event-registration.html` in a browser. No dependencies, no build step.

## Expected output

A **Tech Conf 2026 Registration** card. On **Purchase Tickets**:

| Name | Qty | Type | Result |
|---|---|---|---|
| *(blank)* | 2 | Standard | Error — `Name field cannot be left blank.` |
| Alex | −3 | VIP | Error — `Ticket quantity must be at least 1.` |
| Sam | 1.5 | Standard | Error — `You cannot purchase fractional tickets.` |
| Bo | 3 | VIP | Receipt — Attendee **Bo**, Tickets **3**, Total **$360.00** |

The receipt stays hidden whenever validation fails, and any previous error is
cleared on success.

## Live URL

https://edu.jinfusion.dev/it102-event-registration-form/

## AI collaboration — tool and prompts

**Tool used:** Claude (Anthropic), via Claude Code.

The assignment deliberately splits this into two prompts — first the naive
version, then the defensive one.

### Prompt 1 — the Happy Path

> Act as a JavaScript developer. Write a function called `calculateRegistration`
> that accepts two parameters: `quantity` (a number) and `ticketType` (a string:
> 'standard' or 'vip').
>
> Standard tickets cost $50 each, and VIP tickets cost $120 each. The function
> should calculate the total cost and return it as a number. Return only the
> function code.

### Prompt 2 — hunting the edge cases

> Now, write a validation function called `validateInputs` that accepts three
> parameters: `name` (string), `quantity` (number), and `ticketType` (string).
>
> Write a conditional chain (if/else statements) that checks for these exact
> validation rules:
> 1. If `name` is empty or contains only spaces, return the string error message:
>    "Name field cannot be left blank."
> 2. If `ticketType` is empty, return: "Please select a valid ticket type."
> 3. If `quantity` is less than or equal to 0, return: "Ticket quantity must be
>    at least 1."
> 4. If `quantity` is not a whole integer (a decimal like 1.5), return: "You
>    cannot purchase fractional tickets."
>
> If all inputs pass inspection, the function should return `true`. Return only
> the JavaScript function using clear if/else blocks and comparison operators.

The event listener was written by hand, per Step 3.

### What I corrected after reviewing the output

- **Rule order is load-bearing** and must match the spec exactly, because Test 1
  has a blank name *alongside* otherwise-valid input. Reordering changes which
  message appears.
- **`Number.isInteger()` rather than `quantity % 1 === 0`**, because the modulo
  version accepts `NaN` and `Infinity`.
- **Both boxes are toggled on every click.** The spec names only one box per
  branch; following it literally would leave an error and a receipt on screen at
  the same time.

Full reasoning in `AUDIT-NOTES.md`.

## Verification

```bash
node ../../tools/run_tests.js
```

**All four of the professor's test rows pass**, plus pricing checks and five
further edge cases. See `TEST-CASES.md`.
