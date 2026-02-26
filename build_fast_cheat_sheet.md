# Build Fast | AI Powered Development - Cheat Sheet for Non-Programmers

This is the working draft of a flow for building real software using LLMs as your hands. You control taste, direction, and scope. The LLM holds the complexity. I am not a developer but I design solutions and often work with code. This was born out of my frustration with not being able to work quickly enough despite the promises of AI: I can see the solution, now I want it.

The core principles:

- Design with lightweight contracts (Given → When → Then), not architecture documents.
- Build ugly first, prove it works, then refactor into your preferred style.
- Never edit LLM-generated code. Describe what's wrong.
- Accumulate every mistake into living constraints that make the LLM smarter over time.
- Iterate in disciplined layers: happy path → real data → sad paths → edge cases → polish.

The workflow is designed to scale  from chatbot copy-paste (based on my experiences) to Claude Code with parallel agents (Boris Cherny's published workflow - creator of Claude Code at Anthropic).

What follows: a constraints template (Rules.md), prompt templates for each phase, workflow optimizations for chatbots and Claude Code, and example aliases for local automation.

## RULES.md | Living Constraints - Every Mistake Becomes a Rule

These are the guardrails that govern behavior of the LLM. They define scope, enable focus, correct behavior, and embed best practices.

These can be accompanied by agent-specific rules files.

After every LLM mistake:

1. What did it do wrong?
2. Write a one-line rule that prevents it.
3. Add it to your constraints block permanently.

## Base Workflow

Plan → describe objective →  create given when then flow contract →  create development plan →  create skeleton with function signatures →  create property based tests →  hand off

execute → implement one or more given when then →  implement entire skeleton → run →  errors →  revise →  run →  next →  complete skeleton

# Workflow Optimizations

### Chatbot Workflow

Essentials:

1. Frontier model (Claude, ChatGPT) for all code generation and reasoning.
2. 3+ sessions open: plan, build, subagents (review, simplify, verify, security).
3. Upload Rules.md + code files. Paste critical constraints inline. Check it into repo as living constraints file.
Every mistake becomes a rule. Whole team contributes.
4. Start by planning. Iterate until the plan is right. Then run the code without reading it.
5. Clipboard-friendly test and other aliases.
6. Running changelog for rollback.
7. Local model (Ollama) for glue: format test output, describe diffs,
auto-changelog, connecting to tools. Never for code.
8. Give Claude a way to prove the code works, then generate test
suites that allow it to check its own work.

#### Layer 0 - "Happy Path"

1. Write a spec, plan, or acceptance criteria. EXAMPLE:

`Given [precondition / current state] → When [user action or trigger] → Then [what happens + what the user sees]`

1. Write a skeleton with function signatures thet have input and output types in comments, and a transition table as a data structure if used, plus orchestration wiring. Every function currently throws "Not implemented".  Write property based tests. For each function signature, generate 1-2 property-based tests that describe what should ALWAYS be true about its output, regardless of specific input values.

For the transition table (if applicable), test every valid and every invalid transition.

1. Embed plan in prompt template with `Rules.md`.
2. Have LLM restate requirements (prompt in template).
3. Ask for a **single-file solution** (<100 lines) and **run it immediately** — don't read it first.
4. Talk don't code: have the LLM fix errors or address behavior gaps.
5. Ship it; next slice (real data, error checking, etc.)
6. Answer questions about implementation details with "the simplest way that works."

### Claude Code Workflow (Boris Method)

1. Opus with thinking for everything. Slower but less steering.
2. 5 numbered terminal tabs with notifications + 5-10 web agents
via Chrome extension, hand off between them using teleport.
3. Rules.md checked into repo as living constraints file.
Every mistake becomes a rule. Whole team contributes.
4. Start every task in Plan mode. Iterate until the plan is right.
Then switch to auto-accept and let Claude one-shot it.
5. Slash commands in .claude/commands/ for repeated workflows
(commit-push-pr, format, etc). Ask Claude to create them.
6. MCP configs checked into repo so Claude can use your tools
(Slack, Sentry, BigQuery, etc) directly.
7. Subagents for post-build passes: simplify, verify, security.
Pre-allow safe bash commands via /permissions instead of
skipping permissions entirely. Check into .claude/settings.json.
8. Verification loop: give Claude a way to prove the code works,
and have it test its own work via browser, test suites, or bash.

### EXAMPLE Slash Commands, Aliases, and Local LLM Prompts

```markdown
alias t="clear && $TEST_CMD 2>&1 | tee /tmp/last-test.txt && cat /tmp/last-test.txt | pbcopy"
cat prompt.txt | ollama run mistral "Does this prompt ask for exactly one change? Does it include constraints? Flag anything vague."
diff old.js new.js | ollama run mistral "Write a one-line changelog entry for this change." >> changelog.txt
$TEST_CMD 2>&1 | ollama run mistral "Format this test output as a bug report: what failed, what was expected, what actually happened. Nothing else."
```

## Templates

### PHASE 1 — PLAN (no code yet)

```markdown
[upload Rules.md]
```

```markdown
See attached Rules.md for full rules.
Critical reminders: single file, under 100 lines, no error handling unless asked.

See attached Rules.md.

I want to build: [prose description, as messy as you want]

Convert this into a flow contract using this format:
Given [precondition] → When [trigger] → Then [outcome + what user sees]

One row per step. Happy path only. Ask if unclear. Do NOT write code.

```

```javascript
Here's the agreed flow contract:
[paste the approved Given-When-Then rows]

Output the plan in this exact format:

DONE WHEN: [observable behaviors that prove it works]

STATES (if applicable):
| State | Action/Guard | Next State | Enforced by |

FUNCTIONS:
1. [function] — [purpose ≤10 words] — [inputs → outputs] — [side effects, if any] — [what user sees]

KEY ASSUMPTIONS: [any data shape, external API, or dependency the plan relies on]

Happy path only. Ask if unclear. Do NOT write code.
```

```markdown
Below is the agreed plan. Write a skeleton. Do NOT implement any logic.

### Skeleton Structure
- Pure function signatures: data in → data out, no side effects
- Input/output types noted in comments
- Every function body: throw "not implemented"
- Transition table (if applicable) as a data structure, enforced
- Orchestration at top level as a pipeline: input → fn1 → fn2 → output (all side effects here only)
- Use language idioms where significantly more robust, readable, or simple

### Tests (generate alongside skeleton)
- 1-2 property-based tests per function: what should ALWAYS
  be true about its output, regardless of input values
- Transition table (if applicable): test every valid transition
  succeeds and every invalid transition is rejected

### Plan
[paste plan here]

```

### PHASE 2 — SKELETON VERIFICATION (first time LLM sees this design)

```markdown
Belown is the agreed plan. Restate my requirements before coding (happy path only).

### Agreed plan:
[paste the approved plan from Phase 1]

### Requirements

SCENARIOS:
* Given → When → Then

STATES:
* State → [transition] → State

CONSTRAINTS:
* Top 3 from Rules.md that affect this plan

No error handling, no edge cases. Flag any gaps.
```

### PHASE 3 — EXECUTE

```markdown
[upload Rules.md + current code (signatures + code to target)]
```

```markdown
See attached Rules.md for full rules.
Critical reminders: single file, under 100 lines, no error handling unless asked.

See attached current.js.

Implement step [N] only.
```

```markdown
Tests failed after implementing [concern]. Here is the output:

[paste test output]

Fix only the failing functions. Do not change:
- Function signatures
- Orchestration
- Tests
- Any passing code

Run tests after fixing. If still failing, explain what you
tried and what you think is wrong.


```

#### Agentic

```markdown
After implementing any concern, run tests immediately.
If tests fail:
1. Read the output. Fix only the failing functions.
2. Do not change signatures, orchestration, tests, or passing code.
3. Run tests again.
4. Repeat until all tests pass or the same test fails 3 times.
5. If stuck after 3 attempts, stop. Report: what failed, what you
   tried, your best guess at root cause. Wait for user decision.


```

### PHASE 4 — REFINE (per agent — use dedicated review, simplify, verify, or security agent)

### RULES.md

```markdown
## CODE INSTRUCTIONS
- Single file, under 100 lines. If it can't fit, ask me to cut scope.
- No error handling, logging, or types unless I ask for them.
- No comments unless the logic is non-obvious.
- If unclear, ask me — don't guess.
- Provide a terse commit message only with the changes made. Reference specific functions or code sections updated. EXAMPLE: "Enrich state objects (Timestamp, RowNumber, FileName), switch merge loops to ForEach-Object, and add 'file downloaded' to transition table."


## PROCESS INSTRUCTIONS
- Review your output against the agreed plan and constraints.
- Be CRITICAL — pull no punches.
- Be CREATIVE.
- Explore 1-2 alternative approaches and decide on the best approach.
  Briefly note at the end which alternatives you explored using this format:
    - If you decide to [wire up a SQLite database hoping its query power
      would pay off later], say "turn to page 42".
      BEWARE: [you'll find yourself debugging schema migrations].
    - If you decide to [go with browser localStorage betting on zero
      back-end], say "turn to page 71".
      BEWARE: [data vanishes when you clear the cache.]
- Ask and answer any questions in terse bullets without large headings.
```

### System prompt for scenarios with limitations

```markdown
You are a code generator working under strict constraints.

CRITICAL RULES:
- Single file, under 100 lines. If it can't fit, ask me to cut scope.
- No error handling, logging, types, or comments unless asked.
- No abstractions "for later." Minimal dependencies.
- If unclear, ask — don't guess.
- Never change code I didn't ask you to change.

PROCESS:
- refer to the provided table of contents for the uploaded rules.md file to identify and use any relevant sections as you generate your response. Ensure your output aligns with the rules and guidelines in this document below as a table of contents.

Rules.md - TABLE OF CONTENTS
```