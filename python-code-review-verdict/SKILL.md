---
name: python-code-review-verdict
description: Reviews an AI model's Python response against the prompt it was given, then returns a verdict, the bug and its line, a proof input with actual vs expected output, a minimal fix, and a 1–5 rating. Use when asked to review, grade, rate, check or score AI-written Python code, compare two responses to the same prompt, or answer a code-review assessment item.
---

# Python code review verdict

You review one AI-written Python response (or two, side by side) against the prompt it answered. You are judging whether it does what the prompt asked, not whether you would have written it the same way.

## Step 1: Read the prompt before the code

List every requirement the prompt states, in a short bullet list, before you look at the code. Pay special attention to:

- Words that change the answer: "all" rows, "any iterable" (including generators), "rounding half up", "most recent first", "goes last", "the last chunk may be shorter".
- Where the input comes from. "From a web form" or "from a URL" means the input is untrusted.
- The prompt's own example. If it gives one, it is your first test.

## Step 2: Trace the code against each requirement

Run the code in your head on the smallest inputs that touch each requirement: the prompt's example, an empty input, one item, the boundary (last page, last window, exact half cent), and one hostile input if the input is untrusted. Write down what the code actually returns, not what it was meant to return.

Check these bug families every time, because they are the ones that look correct at a glance:

1. Off-by-one: `range(len(x) - k)` vs `range(len(x) - k + 1)`, `while page < total - 1`, `len(items) // n` dropping a remainder.
2. Loops that never shrink: `lo = mid` instead of `lo = mid + 1` in a binary search.
3. Mutable default arguments (`tags=[]`). Before calling it a bug, check whether a shared default is deliberate, like a memo dict.
4. Single-use iterators: a generator consumed by `sum()` before `len(list(...))` or `max()`.
5. Async lost updates: any `await` between reading and writing shared state.
6. SQL built with f-strings or `+` from untrusted input; `fetchone()` when the prompt asks for all rows.
7. Path checks with `startswith(BASE)` on an unnormalized path: test `../`, an absolute path, and a sibling folder with the same prefix.
8. Money in `float` and `round()` (banker's rounding). Test `"0.125"` and `"1.005"`.
9. Sorting with `None` mixed into strings (`TypeError` in Python 3), and tie-breaks the prompt specifies.
10. Errors swallowed: a retry wrapper that returns `None` after the last failure.
11. Tests that cannot fail, like `assert f(x) == f(x)`.

## Step 3: Decide the verdict

Use exactly one of these:

- **Correct**: meets every requirement in Step 1.
- **Has bugs**: tries to do the right thing but gives a wrong result, crashes, hangs or is unsafe for some valid input.
- **Doesn't do what was asked**: ignores or contradicts an explicit requirement (for example, ignores a required tie-break or a required signature).

For two responses, say which is better and give each its own verdict and rating.

## Step 4: Prove every bug

For each bug give:

- **Line**: quote the exact line.
- **Proof input**: a concrete call, what it actually returns (or the exception, or "never returns"), and what it should return. Use exact values, for example `invoice_total(["1.005"])` returns `"1.00"`, expected `"1.01"`.
- **Fix**: the smallest code change that fixes it.

If you could not verify a value with certainty, write "Unverified" next to it instead of guessing. Never say you ran the code unless you actually did.

## Step 5: Rate it 1–5

- **5**: correct and clear.
- **4**: correct but surprising or fragile (a teammate could easily break it).
- **3**: works for normal input, with a real but minor gap (an unhandled edge case the prompt didn't stress).
- **2**: the core logic is right, but there is a real bug on a normal input.
- **1**: wrong for the prompt's own example, crashes or hangs on valid input, or has a security hole.

## Rules

- Lead with any security issue (injection, path traversal), even if there are other bugs.
- Don't invent bugs. A style preference ("could use `dict.fromkeys`") is not a defect. Calling a correct response buggy is a mistake, not caution.
- Credit what is correct in one line, for example "the zero check is right".
- Keep it short: no essays, no restating the code.

## Output format

```
Requirements: <bullets from Step 1>
Verdict: <Correct | Has bugs | Doesn't do what was asked>
Rating: <1–5>
Bug 1: <line> — <what goes wrong>
  Proof: <call> returns <actual>; expected <expected>
  Fix: <code>
(repeat for each bug, security first)
What's right: <one line>
Trap: <anything that looks like a bug but isn't, and why the prompt allows it, or "none">
```

For a correct response, skip the Bug lines. Always fill in the Trap line. Look for things a hasty reviewer would wrongly flag, for example "`round()` uses banker's rounding, which is fine because the prompt doesn't ask for half up", or "the mutable default is a deliberate memo cache".
