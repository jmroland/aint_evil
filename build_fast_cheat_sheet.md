# Build Fast | AI Powered Development - Cheat Sheet for Non-Programmers

A minimal Human - AI framework that helps the human collaborate with AI as a programmable collaborator to get work done and improve workflows, rather than exclusively using AI to help perform otherwise manual work. It organizes work around **structured context (memory), reusable thinking (knowledge), and command**-triggered **pipelines**. The system captures context specific constraints, injects frameworks/templates, and produces artifacts for re-use.

**Slash commands** route tasks through lightweight pipelines using **contextual state cards (memory)** and **frameworks/templates (knowledge)** stored in a prescribed folder structure. Pipelines generate **artifacts** (docs, analyses, updates) and automatically update scope or add constraints to a specific context (such as a project, general task, or area of obligation).

The system is built as a walking skeleton, used to build itself, and emphasizes quality and deterministic outputs wherever possible.

It is built on the the **Development Framework** below.

### Definitions

- **objectives & opportunities**: what you want to accomplish (trigger + acceptance criteria)
- **pipelines**: orchestrate how the objective is accomplished
- **tasks**: context-specific steps using specific tools (e.g., Excel, email) and preferences
- **flow contracts**: input/output schema, invariants (must…), tools, prohibited behaviors (do NOT…), evaluation harness (golden examples ~5–20 tests, rubric scoring, regression checks), governance
- **agents**: task executors generalized across contexts, knowledge (preferences/methods), and tools
- **artifacts**: outputs produced by tasks or pipelines; stored for reuse or reference
- **memory**:
    - **state** (objective, inputs, outputs, constraints, plan/steps, questions, stakeholders, deliverable, phase if needed)
    - **decisions** (with rationale and any structured outputs such as prompts)
    - **preferences** (tone, stakeholders, style)
    - stored in a **state card** with append log: date | type (scope_change, constraint) | change | reason | impact
- **knowledge**: stepwise frameworks, fillable templates, scoreable rubrics, and anti-patterns/guards for consistent reasoning
- **router**: determines which pipeline runs, retrieves context (**state → recent changes → preferences**), and injects KB (**template + rubric**)
    - **pipeline retrieval contract** (JSON/YAML): state targets by field, recent changes, preferences, knowledge
- **commands**: capture (opportunity, scope_change, constraint, preference), meeting, critique (rubric-scored revision loop), brand <x> (convert x input to branded y output)

### Folder Structure:

- ops/opprtunities.md
- memory/projects/<p>/state.md + decisions.md + preferences.md
- router/router_rules.md
- kb/frameworks/<f>.md
- kb/templates/<t>.md
- kb/rubrics.md
- temeletry/log.md (date, command used, manual edits, outcome)
- agents/<a>.file (backup), description.md (with links and change log)

### **LLM pipeline design layers**:

1. **skeleton / MVP** (minimal end-to-end flow)
2. **structured inputs** (convert messy user input into usable LLM structure)
3. **output normalization + packaging** (format, tone, structure)
4. **critique loop** (check invariants vs rubric; auto-revise if needed)
5. **retrieval augmentation** (pull state, decisions, templates, rubrics)
6. **instrumentation** (logs, decision/change analysis)
7. **automation hooks** (trigger memory updates, artifact storage, downstream pipelines)

# Development Framework

This is the working draft of a flow—built as a component of the framework, for building real software using LLMs as your hands. You control taste, direction, and scope. The LLM holds the complexity. I am not a developer but I design solutions and often work with code. This was born out of my frustration with not being able to work quickly enough despite the promises of AI: I can see the solution, now I want it.

The core principles:

- Design with lightweight contracts (Given → When → Then), not architecture documents.
- Build ugly first, prove it works, then refactor to amend with code, security, and other refinements.
- Iterate in disciplined layers: happy path → real data → sad paths → edge cases → polish.
- Avoid editing LLM-generated code for as long as possible. Describe needed changes.
- Accumulate every LLM mistake into living constraints according to leading practice such as Claude.md.

The workflow is designed to scale from a chatbot copy-paste approach to fully integrated Claude Code with parallel agents (see Boris Cherny's published workflow - creator of Claude Code at Anthropic).

What follows: a constraints template (Rules.md), prompt templates for each phase, workflow optimizations for chatbots and agentic scenarios, and examples.

## The Human

Am I reading code before running it? Refactoring something that works? Debating approaches? Handling edge cases before the happy path? "Let me just understand this one thing..."?

**Stop. Run the loop.**

---

## The Robot

**For chatbots:** Put non-negotiable rules in the system prompt / persona (processed every turn). Project-specific rules stay in Rules.md (uploaded per session).

**For Agents:** Rules go in agent instructions (applied every turn). Reference material goes in knowledge sources (RAG-retrieved, not guaranteed every turn). Never put rules in knowledge sources.

---

## RULES.md | Living Constraints - Every Mistake Becomes a Rule

These are the guardrails that govern behavior of the LLM at various stages. They correct behavior and embed best practices.

These can be accompanied by agent- or stage-specific rules files.

After every LLM mistake:

1. Ask what did it do wrong?
2. Write a one-line rule that prevents it.
3. Add it to your constraints block permanently.

---

## Base Workflow

```other
Plan Agent (reasoning model, Chat 1):
  1A: prose → flow contract (design, no code)
  1B: approved design → skeleton + tests (structural code, no implementation)

Build Agent (execution model, Chat 2):
  Handoff: restate scenarios, states, top 3 constraints → user confirms
  Execute: implement concerns → run tests → fix → repeat until green
  Iterate: real data → sad paths → edge cases → refactor → polish

Plan Agent returns for Layers 1-3:
  Scope each layer's concerns before Build Agent implements

Review Agents (any model, fresh chats, Chat 3+):
  Review, simplify, security → results fed back to Build Agent
```

---

## Workflow Optimizations

### Chatbot Workflow

1. Frontier model (Claude, ChatGPT) for all code generation and reasoning.
2. 3+ sessions: plan, build, review/simplify/security.
3. Upload Rules.md + code files. Paste critical constraints inline.
4. Start by planning. Iterate until the plan is right. Then run code without reading it.
5. Copy/paste results back. Use local LLMs (Ollama) for formatting output, diffing, changelog — never for code.
6. Running changelog for rollback.

### Claude Code Workflow (Boris Method)

1. Opus with thinking for everything. Slower but less steering.
2. 5 terminal tabs + 5-10 web agents, hand off via teleport.
3. Claude.md checked into repo. Every mistake becomes a rule. Whole team contributes.
4. Start in Plan mode, iterate until right, switch to auto-accept.
5. Slash commands in `.claude/commands/` for repeated workflows.
6. MCP configs checked into repo so Claude uses your tools directly.
7. Subagents for post-build: simplify, verify, security. Pre-allow safe bash via `/permissions`.
8. Verification loop: Claude tests own work via browser, test suites, or bash.

### Example Aliases and Local LLM Prompts

```bash
alias t="clear && $TEST_CMD 2>&1 | tee /tmp/last-test.txt && cat /tmp/last-test.txt | pbcopy"
cat prompt.txt | ollama run mistral "Does this prompt ask for exactly one change? Does it include constraints? Flag anything vague."
diff old.js new.js | ollama run mistral "Write a one-line changelog entry for this change." >> changelog.txt
$TEST_CMD 2>&1 | ollama run mistral "Format this test output as a bug report: what failed, what was expected, what actually happened. Nothing else."
```

---

## Agent Identities

Use these as the opening line of each agent's instructions. One sentence of identity, one of boundary.

**Plan Agent:** You turn prose project descriptions into flow contracts, plans, and skeletons. You never write implementation code.

**Build Agent:** You implement functions from skeletons until tests pass. You fix your own errors. You change only what is asked.

**Review Agent:** You review finished code against its flow contract. You list problems only. You do not fix them.

---

## Templates

### PHASE 1A — DESIGN (no code)

*Plan Agent. Reasoning model. Chat 1.*

```other
[upload Rules.md]

I want to build: [prose description, as messy as you want]

Convert this into a flow contract using this format:
Given [precondition] → When [trigger] → Then [outcome + what user sees]

One row per step. Happy path only. Ask if unclear.

Then output the plan:

SMOKE TEST: [one sentence — what you'd show someone to prove it works]

STATES (if applicable):
| State | Action/Guard | Next State |

KEY ASSUMPTIONS:
- [any data shape, external API, or dependency the plan relies on]

Do NOT write code, signatures, or tests.
```

**Process instructions (include in agent instructions or paste inline):**

```other
- Be critical. Be creative.
- Try to answer your own questions with the simplest way that works,
  then ask if still unclear.
- Briefly note self-resolved questions under "Q/A (self-resolved):"
  Example: Delimiter/encoding? Assume UTF-8, comma.
- Explore 1-2 alternative approaches. Decide on the best.
  Note alternatives as:
    If you [approach], turn to page 42.
    BEWARE: [specific downside].
```

*User reviews and approves design before proceeding.*

---

### PHASE 1B — SKELETON (structural code, no implementation)

*Plan Agent. Same chat as 1A.*

```other
Here is the approved design:
[paste or reference from context]

Generate a skeleton and tests. Do NOT implement any logic.

SKELETON:
- Pure function signatures: data in → data out, no side effects
- Input/output types noted in comments
- Every function body: throw "not implemented"
- Orchestration as a pure run_command function that routes input
  to correct functions and returns results. No IO in run_command.
  IO (input, print, DB) stays in main() only.
  run_command has its own tests like any other function.
- Transition table (if applicable) as enforced data structure
- Use language idioms where significantly more robust or readable

TESTS:
- 1-2 property-based tests per function: what should ALWAYS be true
  about its output, regardless of input values
- Orchestration: test routing for valid commands and unknown commands
- Transition table: test every valid + invalid transition
```

*User runs skeleton. All tests should fail. This is correct.*

---

### PHASE 2 — HANDOFF (once, at start of Chat 2)

*Build Agent. Execution model. Chat 2.*

```other
[upload Rules.md + skeleton file]

Before writing any code, restate:

SCENARIOS (happy path only):
Given → When → Then

STATES (if applicable):
State → [transition] → State

CONSTRAINTS:
Top 3 from Rules.md that affect this build

Flag any gaps. Wait for my confirmation before implementing.
```

*User verifies Build Agent understood design. Proceed only after confirmation.*

---

### PHASE 3 — EXECUTE

*Build Agent. Same chat as handoff.*

**Implement a concern:**

```other
All tests are currently failing. Implement [concern — paste the
Given-When-Then row(s) from your flow contract]:

Given [row from flow contract]

Do not change signatures, orchestration, or tests.
Run tests when done.
```

**Fix failures (manual — for chatbot without code execution):**

```other
Tests failed. Here is the output:

[paste test output]

Fix only the failing functions. Do not change:
- Function signatures
- Orchestration
- Tests
- Any passing code

Run tests after fixing. If still failing, explain what you
tried and what you think is wrong.
```

**Fix failures (agentic — for Copilot Studio with code interpreter or Claude Code):**

Add to Build Agent instructions:

```other
After implementing any concern, run tests immediately.
If tests fail:
1. Read the output. Fix only the failing functions.
2. Do not change signatures, orchestration, tests, or passing code.
3. Run tests again.
4. Repeat until all tests pass or the same test fails 3 times.
5. If stuck after 3 attempts, stop. Report: what failed, what you
   tried, your best guess at root cause. Wait for user decision.
```

**Code constraints (include in Build Agent instructions or paste inline):**

```other
- Single file, under 100 lines. If it can't fit, ask me to cut scope.
- No error handling, logging, or types unless I ask.
- No comments unless the logic is non-obvious.
- If unclear, ask — don't guess.
- Never change code I didn't ask to change.
- Implement one CONCERN per response (group related functions).
- Provide a terse commit message referencing specific functions updated.
```

*Repeat until all tests pass and behavior matches the smoke test.*

---

### PHASE 3+ — LAYER SCOPING (Plan Agent returns)

*Plan Agent. Reasoning model. Chat 1 or fresh chat.*

Before Layers 1-3, ask the Plan Agent to scope the concern:

```other
Here is the current flow contract and passing code:
[paste or attach]

I want to add [Layer concern: e.g., error handling / real data / edge cases].

Output:
1. Which specific cases matter for this flow contract?
2. For each case: Given → When → Then (new rows)
3. Which existing functions are affected?
4. Any new functions needed? (signature only)
5. Any new tests needed? (assertion description only)

Do NOT write implementation code.
```

*User approves new rows, then takes them to the Build Agent.*

---

### PHASE 4 — ITERATE (Layers 1-5)

*Build Agent. Same chat. One layer per pass.*

Each layer is one prompt. Tests must pass before moving to the next.

**Layer 1 — Real data:**

```other
Replace [fake data source] with [real source].
Add side-effect functions at the edge if needed:
  save_to_db(db, item) and load_from_db(db)
Orchestration calls these. Core functions stay pure.
No other changes. Run tests.
```

**Layer 2 — Sad paths** (use rows from Phase 3+ scoping):

```other
Add error handling for these specific cases:
[paste Given-When-Then rows from Plan Agent]
No other changes. Run tests.
```

**Layer 3 — Edge cases** (use rows from Phase 3+ scoping):

```other
Enforce these state transition edge cases:
[paste Given-When-Then rows from Plan Agent]
Add tests for these cases. No other changes. Run tests.
```

**Layer 4 — Refactor to style:**

```other
Refactor to functional style:
- All core functions pure: data in, data out, no mutation
- Use dict/object unpacking for immutable updates
- Replace loops with comprehensions or map/filter
- No variable reassignment in core functions
- Side effects (DB, print) only in orchestration
Behavior must remain identical. Run tests.
```

**Layer 5 — Polish:**

```other
Add type hints to all function signatures.
Split into separate files if over 150 lines.
No behavior changes. Run tests.
```

---

### PHASE 5 — REFINE

*Fresh chats. Any model. Chat 3+.*

**Review:**

```other
[upload Rules.md + code]

You review finished code against its flow contract.
Does this enforce these states and transitions?
List problems only. Do not fix them.
```

**Simplify:**

```other
[upload Rules.md + code]

Simplify this code. Keep behavior identical.
Target: under 100 lines. Run tests.
```

**Security:**

```other
[upload Rules.md + code]

Flag security vulnerabilities. Do not fix them.
```

*Feed results back to Build Agent in Chat 2 for fixes.*

---

## Cross-Language Prototyping

Prototype in Python always (LLMs generate it most reliably, Copilot Studio code interpreter runs it natively). When porting to another language:

1. **Keep the Python prototype as your executable spec** — it proves the design works.
2. **Hand the target-language agent the plan + flow contract + passing test descriptions** (not the Python source). Let it build a fresh skeleton using that language's idioms.
3. **Port the test assertions, not the implementation.** "Given X, when Y, then Z" doesn't change. The test bodies do.

This re-enters Phase 1B → Phase 2 → Phase 3 with a different language target. The flow contract is the durable artifact; the code is disposable.

**Heuristic:** ≥3 Given-When-Then rows or a state machine → Python-first. ≤2 rows, no state → go direct.

---

## Example: Bookmark Manager

**Flow contract:**

```other
Given no bookmarks exist → When user saves a URL → Then bookmark created with status "unsorted"
Given bookmarks exist → When user searches by tag → Then matching bookmarks returned
Given bookmarks exist → When user adds tags to a bookmark → Then bookmark status becomes "sorted"
Given unsorted bookmarks older than 1 day exist → When digest runs → Then list of unsorted bookmarks returned
```

**Smoke test:** I can save a URL, tag it, search by tag, and see unsorted ones in a digest.

**States:** unsorted → [add tags] → sorted

**Skeleton (abbreviated):**

```python
TRANSITIONS = {
    "unsorted": {"add_tags": "sorted"},
}

def valid_transition(current, action):
    return action in TRANSITIONS.get(current, {})

def create_bookmark(url):
    # str -> dict {url, tags, status, created_at}
    raise NotImplementedError

def add_tags(bookmark, tags):
    # dict, list[str] -> dict (new bookmark with tags + updated status)
    raise NotImplementedError

def search_by_tag(bookmarks, tag):
    # list[dict], str -> list[dict]
    raise NotImplementedError

def get_unsorted_digest(bookmarks, now):
    # list[dict], datetime -> list[dict]
    raise NotImplementedError

def run_command(store, command, args):
    # list, str, dict -> tuple(list, str)
    # Routes to correct function, returns (new_store, output)
    # NO IO here
    raise NotImplementedError

def main():
    # IO only: read input, call run_command, print, persist
    raise NotImplementedError
```

**Layer progression:** Layer 0 (implement until tests pass) → Layer 1 (SQLite) → Layer 2 (duplicate URL, empty tags, invalid URL) → Layer 3 (tag already-sorted bookmark) → Layer 4 (functional refactor) → Layer 5 (type hints, split files).

