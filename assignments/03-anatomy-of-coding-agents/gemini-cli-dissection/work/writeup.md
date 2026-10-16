# Week 3 Writeup — Inside a Coding Agent

**Name:** Ezgi Turan
**Gemini CLI version (`gemini --version`):** 0.64.0
**OS / shell:** macOS + zsh
**Model(s) seen in trace:** `gemini-3.5-flash-lite` (all 16 calls)
**Session commit:** `c01f037`

> Evidence tags: `[OBSERVED]` = cite a timeline line, call number, or diff hunk.
> `[INFERRED]` = say what you're reasoning from.

---

## Part I — Session

**Narrative (one paragraph):** After the single prompt, the agent read `TASK.md`, ran
`python -m pytest` to see the two failing tests, and then read every source and test file
(`cart.py`, `pricing.py`, `test_cart.py`, `test_pricing.py`, `catalog.py`) before touching
anything. It searched for `.total(` callers, fixed `add_item` so repeated adds increment
the quantity, and rewrote `final_total` in `pricing.py` to apply the discount before tax
and add a new `shipping_fee` rule. It then updated three pre-shipping expected totals in
`test_cart.py`, added four shipping tests, re-ran pytest (all passing), checked
`git status`/`git diff`, and declared the task complete. Four `503 high demand` API errors
occurred along the way; the CLI retried each automatically. The whole run took about
2 minutes 20 seconds (07:01:51 → 07:04:09).

| Metric | Value |
|---|---|
| User prompts | 1 |
| Model calls | 16 successful (+4 failed `503` requests, auto-retried) |
| Tool calls (failed) | 26 (0) |
| Final `pytest` result | `18 passed` (was 2 failed, 12 passed before the agent) |
| Did you step in? When / what? | No |

---

## Part II — System prompt annotation

### Outline
1. TODO — label
2. TODO — label
3. …

### Annotations (≥6)
#### Section: TODO (tone and verbosity)
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

#### Section: TODO (when not to act / confirm)
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

#### Section: TODO (workflow)
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

#### Section: TODO (tool usage)
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

#### Section: TODO
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

#### Section: TODO
> quote
- **Behavior it produces:** TODO
- **Failure it prevents:** TODO

### Context injection (`GEMINI.md`)
TODO — where it enters, with evidence.

---

## Part III — Tool design

### Inventory
| Category | Tools | Used in session (count) |
|---|---|---|
| File | TODO | TODO |
| Search | TODO | TODO |
| Shell | TODO | TODO |
| Web | TODO | TODO |
| Memory / other | TODO | TODO |

### Tool A: TODO
- **Source file:** TODO
- **Schema:**
  ```
  TODO
  ```
- **Why each parameter:** TODO
- **Description line that corrects a model mistake:** > TODO. **Mistake:** TODO
- **Deliberately does not:** TODO

### Tool B: TODO
- **Source file:** TODO
- **Schema:**
  ```
  TODO
  ```
- **Why each parameter:** TODO
- **Description line that corrects a model mistake:** > TODO. **Mistake:** TODO
- **Deliberately does not:** TODO

### Why old/new string edits?
TODO — two reasons plus a trace moment.

### Per-model tool descriptions
TODO — tool compared, what differs between model families, why.

---

## Part IV — Behavioral analysis

### 1. The loop
```
TODO: call#1 → tool → call#2 → …
```
What ended it: TODO

### 2. Context growth
TODO — chart or table, then explanation (growth and caching).

### 3. Planning
TODO

### 4. Error recovery
> quoted failure
TODO — what happened next and whether it was good.

### 5. The tests
TODO — any edits to existing tests, quoted diff, allowed by TASK.md?

### 6. Models
TODO
