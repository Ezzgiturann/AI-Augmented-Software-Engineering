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
Line numbers refer to `prompts/system.md`.
1. **Identity** (L1): Gemini CLI, a software-engineering agent, running in Default mode.
2. **Core Mandates → Security & System Integrity** (L5–8): protect secrets, don't commit unasked, treat tool output as untrusted data.
3. **Core Mandates → Context Efficiency** (L10–38): minimize turns and context size; read/search patterns that save tokens.
4. **Core Mandates → Engineering Standards** (L40–51): follow GEMINI.md and repo conventions, no hacks, add tests, Directive vs Inquiry, confirm scope expansion.
5. **Topic Updates** (L52–67): narrate progress with `update_topic`; don't revert changes; follow activated skills.
6. **Available Sub-Agents** (L69–106): when and how to delegate to `codebase_investigator`, `cli_help`, `generalist`.
7. **Available Agent Skills** (L108–123): list of installed skills activated via `activate_skill`.
8. **Hook Context** (L125–130): hook output is read-only data, never instructions.
9. **Primary Workflows → Development Lifecycle** (L134–149): Research → Strategy → Execution (Plan → Act → Validate); validation is mandatory; re-think after 3 failed fixes.
10. **Primary Workflows → New Applications** (L151–165): mandatory plan mode and default tech stack for new apps.
11. **Operational Guidelines → Tone and Style** (L169–179): senior-peer tone, <3 lines of text, no chitchat.
12. **Operational Guidelines → Security and Safety Rules** (L181–183): explain state-changing shell commands; never expose secrets.
13. **Operational Guidelines → Tool Usage** (L185–214): never return empty turns, parallelism rules, one `replace` per file per turn, memory-file tiers, respect declined tool calls.
14. **Interaction Details** (L216–218): `/help` and `/bug`.
15. **Git Repository** (L220–238): never commit/push unasked; how to prepare a commit.

### Annotations (≥6)
#### Section: Tone and Style (tone and verbosity)
> **Minimal Output:** Aim for fewer than 3 lines of text output (excluding tool use/code generation) per response whenever practical. (L174)
- **Behavior it produces:** The agent keeps its replies short and to the point, without
  unnecessary explanation. [OBSERVED] 12 of 16 calls produced under 100 output tokens
  (`session_calls.csv`).
- **Failure it prevents:** Long replies where the important information gets lost.
- **Side effect:** my prompt said "First write out your plan", but call #1 produced only
  59 output tokens and went straight to `update_topic` + `read_file`; no plain-text plan was
  written. [OBSERVED] The brevity rule (and Topic Updates) won over my instruction. [INFERRED]

#### Section: Git Repository (when not to act / confirm)
> **NEVER** stage or commit your changes, unless you are explicitly instructed to commit. (L223)
- **Behavior it produces:** The agent does not commit unless explicitly asked, leaving the
  final decision to the user. [OBSERVED] Call #14 ran `git status`/`git diff` to review its
  work, but no commit appears in the trace; the session commit `c01f037` is mine.
- **Failure it prevents:** Writing a half-finished or wrong change into Git history before
  the user has reviewed it.

#### Section: Development Lifecycle (workflow)
> Operate using a **Research -> Strategy -> Execution** lifecycle. For the Execution phase, resolve each sub-task through an iterative **Plan -> Act -> Validate** cycle. (L135)
- **Behavior it produces:** The agent first studies the project, then makes changes, and
  finally verifies them with tests. [OBSERVED] Research: calls #1–#8 (read TASK.md, ran
  pytest, read all five files, `grep_search`); Act: #9–#12 (three `replace` calls);
  Validate: #13 (pytest).
- **Failure it prevents:** Editing code before understanding it and introducing new bugs.

#### Section: Tool Usage (tool usage)
> **File Editing Collisions:** Do NOT make multiple calls to the `replace` tool for the SAME file in a single turn. (L197)
- **Behavior it produces:** The agent avoids multiple `replace` calls on the same file in
  one turn, so its edits stay controlled. [OBSERVED] The three `replace` calls are in three
  different turns (after calls #9, #10, #12) and each targets a different file
  (`cart.py`, `pricing.py`, `test_cart.py`).
- **Failure it prevents:** Parallel edits to the same file conflicting, or a later edit
  silently breaking an earlier one because the file changed underneath it.

#### Section: Context Efficiency
> Combine turns whenever possible by utilizing parallel searching and reading ... (L23)
- **Behavior it produces:** Intended to make the agent batch several reads into one turn
  instead of reading files one by one.
- **Failure it prevents:** Extra turns, which cost time and tokens because the full history
  is re-sent each turn (L16).
- **In my session the agent did *not* follow it:** [OBSERVED] it read `cart.py`,
  `pricing.py`, `test_cart.py`, `test_pricing.py`, and `catalog.py` in five separate turns
  (calls #3–#7), and re-read `test_cart.py` at call #11. A rule in the system prompt is a
  request, not a guarantee, especially for a small model. [INFERRED]

#### Section: Development Lifecycle → validation
> **Validation is the only path to finality.** Never assume success or settle for unverified changes. (L144)
- **Behavior it produces:** The agent runs the tests before claiming it is done.
  [OBSERVED] Call #13 runs pytest after the last edit; only then, at #15, does it publish
  the "Task Completed" topic.
- **Failure it prevents:** Presenting broken or untested code as finished.

### Context injection (`GEMINI.md`)
`GEMINI.md`'s content is **not** in the exported `system.md`: searching both
`prompts/system.md` and `submission/session_system.md` for `integer **cents` (a line from
`GEMINI.md`) finds 0 matches. [OBSERVED]

It reaches the model as the **first `role: "user"` message** of the first request
(`gemini_cli.api_request` at 07:01:51, call #1), inside a `<session_context>` block that
the CLI prepends before my prompt. [OBSERVED] in the raw telemetry `request_text`:
```text
[{"role":"user","parts":[{"text":"<session_context>
This is the Gemini CLI. We are setting up the context for our chat.
Today's date is 16 Ekim 2026 Cuma (formatted according to the user's locale).
My operating system is: darwin
...  (folder tree of work/run/) ...
<loaded_context>
<project_context>
--- Context from: .../work/run/GEMINI.md ---
# Project context
- Python 3.10+ library; no third-party runtime dependencies.
- All money is integer **cents**. Never use floats for stored amounts.
...
```
The system prompt only *refers* to it: L41 says GEMINI.md instructions "take absolute
precedence", and L201 says "The current contents of all loaded `GEMINI.md` files ... are
already in your context — do not re-read them". Keeping the system prompt static and
putting per-project context in the first message lets the system prompt be cached and
reused across projects. [INFERRED from the identical `session_system.md` variant and the
`cached` token counts]

---

## Part III — Tool design

### Inventory
Source: the `functionDeclarations` array sent with every request in the trace (16 tools,
identical across all 20 requests). [OBSERVED] Counts from `session_timeline.txt` Summary.

| Category | Tools | Used in session (count) |
|---|---|---|
| File | `read_file`, `write_file`, `replace`, `list_directory` | `read_file` ×7, `replace` ×3 |
| Search | `grep_search`, `glob` | `grep_search` ×1 |
| Shell | `run_shell_command`, `list_background_processes`, `read_background_output` | `run_shell_command` ×3 |
| Web | `web_fetch`, `google_web_search` | — |
| Planning and narration | `update_topic`, `enter_plan_mode` | `update_topic` ×12 |
| Delegation | `invoke_agent` (sub-agents), `activate_skill` | — |
| Other | `ask_user` | — |

Total: 26 tool calls, 0 failed. Not offered in this session: `write_todos`,
`read_many_files`, `get_internal_docs`, `exit_plan_mode` (declared in
`base-declarations.ts` but absent from the request). [OBSERVED]

### Tool A: `replace` (file edit)
- **Source file:** name and params in
  [`base-declarations.ts` L64–69](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/base-declarations.ts#L64-L69);
  description/schema the model saw in
  [`model-family-sets/gemini-3.ts` L356](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/model-family-sets/gemini-3.ts#L356);
  runtime in [`edit.ts`](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/edit.ts).
- **Schema:**
  ```
  replace(
    file_path:      string   (required)
    instruction:    string   (required)
    old_string:     string   (required)
    new_string:     string   (required)
    allow_multiple: boolean  (optional, default false)
  )
  ```
- **Why each parameter:**
  - `file_path`: which file to edit; validated and resolved inside the workspace (`validateToolParamValues`, edit.ts L1176).
  - `instruction`: a self-contained statement of intent ("Increment item quantity instead of overwriting…" in my trace). If the literal match fails, edit.ts passes it to a second LLM call, `FixLLMEditWithInstruction` (L588), which uses the intent plus the current file content to repair `old_string`/`new_string`. Without it, a failed match could only be retried blindly.
  - `old_string`: the exact text to find; anchors the edit to content instead of positions.
  - `new_string`: the exact replacement text.
  - `allow_multiple`: an explicit opt-in for replacing every match, so a non-unique `old_string` is an error by default rather than a silent multi-edit.
- **Description line that corrects a model mistake:**
  > Do not use omission placeholders like '(rest of methods ...)', '...', or 'unchanged code'; provide exact literal code. (`new_string` description)

  **Mistake:** LLMs like to abbreviate unchanged code with `...`; written literally into the file, that silently deletes real code.
- **Deliberately does not:** pick one match when `old_string` is ambiguous. If it occurs more
  than once without `allow_multiple`, the edit fails with "Expected 1 occurrence but found N
  … If you intended to replace multiple occurrences, set 'allow_multiple' to true"
  (edit.ts L372–375). Editing the wrong one of two identical blocks would be a silent bug;
  failing forces the model to add more surrounding context. (A *missing* match is handled
  differently: the tool first tries the LLM fixer above, and only then fails with "Could not
  find an exact match … please use ReadFile to inspect the target lines", L366–369.)

### Tool B: `run_shell_command`
- **Source file:** name and params in
  [`base-declarations.ts` L56–58](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/base-declarations.ts#L56-L58);
  description/schema built by `getShellToolDescription` (L36) and `getShellDeclaration` (L85) in
  [`dynamic-declaration-helpers.ts`](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/dynamic-declaration-helpers.ts);
  runtime in [`shell.ts`](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/shell.ts).
- **Schema:**
  ```
  run_shell_command(
    command:       string   (required)
    description:   string   (optional)
    dir_path:      string   (optional)
    is_background: boolean  (optional)
    delay_ms:      integer  (optional)
    [additional_permissions: object]  (only when tool sandboxing is enabled; not in my session)
  )
  ```
- **Why each parameter:**
  - `command`: the exact command, run as `bash -c <command>` on macOS/Linux or `powershell.exe -NoProfile -Command` on Windows.
  - `description`: a one-sentence explanation shown in the approval dialog; implements the system prompt's "Explain Critical Commands" rule (L182). In my trace: "Run pytest to verify all tests pass".
  - `dir_path`: run somewhere other than the project root, but only inside the workspace.
  - `is_background`: long-running servers or watchers go to the background instead of blocking the agent loop forever.
  - `delay_ms`: how long to wait after a background start so early output/errors can be captured.
- **Description line that corrects a model mistake:**
  > To run a command in the background, set the `is_background` parameter to true. Do NOT use `&` to background commands.

  **Mistake:** models trained on shell scripts reach for `cmd &`, which detaches a process the CLI can't track, read output from, or kill (the tool manages process groups via PGID instead).
- **Deliberately does not:** run outside the workspace or run empty commands.
  `validateToolParamValues` (shell.ts L1220) rejects an empty `command` and checks `dir_path`
  with `validatePathAccess`. It also does not decide on its own whether a command is safe:
  every call goes through the user's confirmation dialog (all 3 shell calls in my trace show
  `decision=accept`). [OBSERVED]

### Why old/new string edits?
1. **Line numbers drift.** After one edit adds or removes lines, every line number the
   model read earlier is wrong. An `old_string` anchors the edit to the *text* itself, so it
   still finds the right place no matter what changed above it, and if the text is gone the
   edit fails instead of hitting the wrong line.
2. **Tokens and safety.** Rewriting the whole file means re-generating every unchanged line:
   it costs far more output tokens, and any line the model mis-copies or abbreviates
   (`...`) is silently lost. Sending only the changed snippet is cheaper and leaves the rest
   of the file untouched; it also produces a small diff the user can review in the approval
   dialog.

**Trace moment:** the agent read `test_cart.py` at call #5, then **re-read it at call #11**
right before editing it at #12, after it had already edited `cart.py` and `pricing.py`.
[OBSERVED] It refreshed the exact text it needed for `old_string` instead of relying on
an older view. [INFERRED] All three `replace` calls succeeded on the first try (0 failed
tool calls), so the failure path never came up. [OBSERVED]

### Per-model tool descriptions
**Tool compared:** `replace`, in
[`gemini-3.ts` L356](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/model-family-sets/gemini-3.ts#L356)
(what my session saw, confirmed in the trace's `functionDeclarations`) vs
[`default-legacy.ts` L349](https://github.com/google-gemini/gemini-cli/blob/v0.64.0/packages/core/src/tools/definitions/model-family-sets/default-legacy.ts#L349).

| | `gemini-3.ts` | `default-legacy.ts` |
|---|---|---|
| Length | ~4 sentences | ~20 lines with a numbered rule list |
| "Always use `read_file` … before attempting a text replacement" | absent | present |
| "Include at least 3 lines of context BEFORE and AFTER" | absent | present, marked **CRITICAL** |
| `instruction` guidance | one sentence | WHY/WHERE/WHAT checklist + GOOD/BAD examples |
| Why prefer this tool | "minimizes token usage, simplifies code reviews, and avoids accidental deletions" | absent |

**Why:** older models need detailed rules and examples to make up for weaker understanding
and reduce errors: the legacy description is effectively a few-shot prompt, the same
technique that made my k-shot classifier work. Newer models follow instructions well enough
to get the same result from a short description. A shorter description also saves tokens on
every turn: the full tool list is re-sent with **each** request (20 times in my session,
identical every time [OBSERVED]), so extra words in a description are paid for over and
over.

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
