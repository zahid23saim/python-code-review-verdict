# python-code-review-verdict

A skill that reviews an AI model's Python answer against the prompt it was given and returns a verdict, the buggy line, a proof input (actual vs expected), a minimal fix and a 1–5 rating. Use it when you grade AI-written code, compare two responses, or practise for code-review assessments.

I review AI-written Python as part of AI-training work, and I kept re-explaining the same checklist: read the prompt first, prove every bug with a concrete input, lead with security, and don't invent bugs in code that's correct. This skill writes that down once.

## Example (a real run)

**Input**

> Prompt: Write `last_n_lines(path, n)` that returns the last n lines of a text file as a list of strings without trailing newlines. n can be 0.
>
> ```python
> def last_n_lines(path, n):
>     with open(path) as f:
>         lines = f.read().splitlines()
>     return lines[-n:]
> ```

**Output**

```
Requirements:
- Define last_n_lines(path, n).
- Read the last n lines of a text file.
- Return a list of strings.
- Remove trailing newline characters.
- Support n = 0.
Verdict: Has bugs
Rating: 2/5
Bug 1: return lines[-n:] — when n is zero, -0 evaluates to 0, so the slice returns the entire list instead of an empty list.
Proof: If the file contains "first\nsecond\n":
last_n_lines(path, 0) returns ["first", "second"].
Expected: [].
Fix:
return lines[-n:] if n else []
What's right: splitlines() correctly removes line-ending characters, and slicing correctly selects the last n lines when n is positive.
Trap: splitlines() handles both \n and \r\n; this is not a bug. The function also correctly returns all lines when n exceeds the file's line count.
```

The bug is easy to miss: `lines[-n:]` looks right and passes every test except `n = 0`, which the prompt explicitly allows. I checked the proof by running it: `lines[-0:]` returns the whole list.

## What it checks

1. Lists the prompt's requirements before reading the code, including easily missed words like "all", "any iterable" and "rounding half up".
2. Traces the code on the prompt's example, empty input, boundaries and one hostile input.
3. Checks 11 bug families that look correct at a glance: off-by-one, loops that never shrink, mutable defaults, single-use generators, async lost updates, SQL injection, path traversal, float money, `None` in sorts, swallowed errors, and tests that can't fail.
4. Proves every bug with a concrete call, the actual result and the expected result, and gives the smallest fix.
5. Rates 1–5 on a fixed scale, leads with security issues, and names traps a hasty reviewer would wrongly flag.

## How I tested it

I ran it in fresh chats on three new cases I didn't use while writing it, and checked each proof in Python:

| Case | Verdict | Notes |
|---|---|---|
| `dedupe(emails)` built with a set | Doesn't do what was asked, 1/5 | Lost order and original casing, with a correct fix |
| `average_rating` using `round(..., 1)` | Correct, 5/5 | No bug invented. The first run didn't name the banker's-rounding trap, so I added a required **Trap** line to the skill. |
| `last_n_lines` with `lines[-n:]` | Has bugs, 2/5 | Caught the `n = 0` case (the example above) |

## Install

- **Any tool that reads skills** (ChatGPT Skills on paid plans, Codex, Cursor, Copilot): copy the `python-code-review-verdict` folder into your skills folder, or install it from this repo's link.
- **Any chat assistant**: paste the contents of [`SKILL.md`](python-code-review-verdict/SKILL.md) into a new chat, add "Follow this SKILL.md exactly", then paste the prompt and the response you want reviewed.

Suggestions welcome. Open an issue or a pull request against `SKILL.md`.
