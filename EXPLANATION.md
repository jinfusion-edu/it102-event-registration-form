# How this works, and why

---

## 1. The concept being tested

**Defensive programming** — assuming the user will do the wrong thing, and
deciding in advance what happens when they do.

The assignment structures this as two prompts on purpose:

1. Write the **Happy Path** — the code that works when nobody makes a mistake.
2. Then go hunting for the ways a "malicious or distracted user could exploit
   the form".

That order mirrors how software actually fails. The happy path is easy and
usually correct. The bugs live in everything else: blank fields, negative
numbers, decimals where you expected integers.

The headline example the assignment gives is a good one. With no validation:

```
quantity = -3, ticketType = "vip"
total = -3 × 120 = -360
```

The form cheerfully issues a receipt for **−$360**. As the assignment puts it:
*"Does the application pay them money?"*

---

## 2. The two functions, and why they are separate

```js
function calculateRegistration(quantity, ticketType) { ... }  // trusts its input
function validateInputs(name, quantity, ticketType) { ... }   // trusts nothing
```

`calculateRegistration` is deliberately naive. It does not check anything,
because checking is not its job. Keeping it that way means the pricing logic is
one readable expression rather than being tangled up with error handling.

`validateInputs` is the gate. Everything hostile is stopped there, and
`calculateRegistration` is only ever called with input that has already passed.

This is the "validate at the boundary" pattern: check once, at the edge, then
let the interior code assume clean data.

### The return-type trick

`validateInputs` returns **either `true` or a string**. That is unusual — most
functions return one type — and it is what lets the caller stay simple:

```js
if (validationResult !== true) {
  // it's an error message; show it
} else {
  // it's fine; proceed
}
```

Note `!== true`, not `if (validationResult)`. A non-empty string is *truthy*, so
`if (validationResult)` would take the success branch for **every** error
message. That is a real bug avoided by comparing against `true` explicitly.

---

## 3. Order is load-bearing

The four checks run in a fixed order, each returning the **first** failure:

```js
if (name is blank)            return "Name field cannot be left blank.";
if (ticketType is empty)      return "Please select a valid ticket type.";
if (quantity <= 0)            return "Ticket quantity must be at least 1.";
if (quantity is not integer)  return "You cannot purchase fractional tickets.";
return true;
```

This is not stylistic. The assignment's Test 1 supplies a **blank name with a
valid quantity and type**, and expects the name error. Test 3 supplies a valid
name with a fractional quantity, and expects the fractional error. Reorder these
and a test that should report one message reports another.

Whenever a spec hands you both rules *and* expected outputs, the outputs pin down
details the prose leaves open — like ordering. That is what makes the test table
more authoritative than the rule list.

### Why `Number.isInteger` and not `% 1`

```js
if (!Number.isInteger(quantity)) { ... }
```

The obvious alternative:

```js
if (quantity % 1 !== 0) { ... }     // subtly worse
```

Both reject `1.5`. But:

| value | `Number.isInteger(v)` | `v % 1 !== 0` |
|---|---|---|
| `1.5` | `false` → rejected ✅ | `true` → rejected ✅ |
| `NaN` | `false` → rejected ✅ | `NaN !== 0` is `true` → rejected ✅ |
| `Infinity` | `false` → rejected ✅ | `Infinity % 1` is `NaN` → rejected ✅ |

They agree here, but `Number.isInteger` says what it means, and it also rejects
non-number types outright rather than coercing them. When two implementations are
equivalent, prefer the one whose name states the intent.

### Blank vs whitespace

```js
if (name.trim() === "") { ... }
```

`""` and `"   "` are different strings but the same problem: no name. `.trim()`
removes leading and trailing whitespace, collapsing both cases to `""`. The
assignment asks for exactly this — *"empty or contains only spaces"*.

The receipt then shows `enteredName.trim()`, so `"  Bo  "` renders as `Bo`
rather than with its padding intact.

---

## 4. Strings again: converting the quantity

```js
var enteredQuantity = Number(quantityInput.value);
```

As in the tip calculator, `.value` is always a **string** — even from
`<input type="number">`. Without conversion:

```js
"-3" <= 0     // false! String/number comparison coerces, but…
"1.5" <= 0    // false
Number.isInteger("3")   // false — a string is never an integer
```

That last one is the killer: **every** quantity would be rejected as fractional,
including valid ones, because a string is not an integer no matter what it
contains.

### Why `Number()` here but `parseFloat()` in the tip calculator

They differ on empty input:

| input | `parseFloat` | `Number` |
|---|---|---|
| `""` | `NaN` | `0` |
| `"12abc"` | `12` | `NaN` |

Here `Number("")` → `0` is exactly what we want: an empty quantity field falls
through to the `quantity <= 0` check and produces *"Ticket quantity must be at
least 1."* — the sensible message. With `parseFloat` it would be `NaN`, which is
not `<= 0`, and would fall through to the fractional message instead.

The tip calculator's assignment names `parseFloat` explicitly, so it uses that.
Different tools for different requirements; the reasoning is what matters.

---

## 5. Toggling both boxes, every time

The spec describes each branch mentioning one box:

> *If it returns an error string: Hide the `#receiptBox`, display the text inside
> `#errorBox`…*
> *If it returns `true`: Hide the `#errorBox`. Run your `calculateRegistration`
> function, display the success `#receiptBox`…*

Read carefully, it does cover both — but it is worth being explicit about why.
Consider: submit valid data (receipt appears), then blank the name and submit
again. If the failure branch only *showed* the error without hiding the receipt,
the screen would display an error **and** a stale receipt for a purchase that did
not happen.

So every click sets both boxes' visibility. The relevant test —
"receipt hidden after a later failure" — exists precisely to catch this.

---

## 6. Alternatives considered

**HTML5 validation attributes** (`required`, `min="1"`, `step="1"`). The browser
would enforce much of this for free. Rejected: the assignment is explicitly about
writing the conditional chain yourself, and client attributes are trivially
bypassed via DevTools — which is the "malicious user" the assignment names.

**Collecting all errors and showing them at once.** Better UX for long forms. The
spec's return type (a single string or `true`) and its test table (each row
expecting one specific message) both require first-failure-wins.

**Throwing exceptions instead of returning strings.** More conventional for error
signalling in larger systems. `try/catch` is beyond the course's current scope,
and the spec dictates the return shape.

**Toggling a CSS class instead of writing `style.display`.** Cleaner, and what
the theme-toggle assignment required. Here the spec says to style `#errorBox`
"so it is visible (`display: block`)", so the literal reading ships. Noted as a
cross-assignment inconsistency in `AUDIT-NOTES.md`.

**Validating on `input` rather than on `click`.** Real-time feedback, as in the
password validator. This assignment specifies a `click` listener on
`#registerBtn`, so validation happens on submit.
