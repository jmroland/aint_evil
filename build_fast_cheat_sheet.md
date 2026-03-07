# Build Fast | AI Powered Development - Cheat Sheet for Non-Programmers

The core principles:

- Design with lightweight contracts (Given → When → Then), not architecture documents.
- Build ugly first, prove it works, then refactor to amend with code, security, and other refinements.
- Iterate in disciplined layers: happy path → real data → sad paths → edge cases → polish.
- Avoid editing LLM-generated code for as long as possible. Describe needed changes.
- Accumulate every LLM mistake into living constraints according to leading practice such as Claude.md.

The workflow is designed to scale from a chatbot copy-paste approach to fully integrated Claude Code with parallel agents (see Boris Cherny's published workflow - creator of Claude Code at Anthropic).

What follows: a constraints template (Rules.md), prompt templates for each phase, workflow optimizations for chatbots and agentic scenarios, and examples.

### The Loop

```other
Design → Build → layer done?
  → next layer → Design → Build → ...
  → all layers done → Review (optional) → Ship
```

NOTE: Every layer starts in Design (to scope) then moves to Build (to implement). The loop is always the same. If Design says “nothing to scope, just build it,” you’re in Build in one turn.

| **Layer**    | **Design scopes**                        | **Build implements**               |
| ------------ | ---------------------------------------- | ---------------------------------- |
| 0 Design     | flow contract, plan, skeleton with tests | —                                  |
| 1 Implement  | (skeleton just came from Design)         | fill in functions until tests pass |
| 2 Real data  | what changes when we swap sources?       | swap them                          |
| 3 Sad paths  | which failures matter? new GWT rows      | implement handling                 |
| 4 Edge cases | which guards matter? new GWT rows        | enforce them                       |
| 5 Refactor   | structural risks? data model changes?    | refactor                           |
| 6 Polish     | anything before shipping?                | types, split files                 |
| 7 Review     | —                                        | — (separate context)               |

## Templates

### LAYER 0 — DESIGN

#### Human Instructions 

- Design context. One prompt, two steps with a pause.

```other
You turn prose project descriptions into flow contracts, plans,
and skeletons. You never write implementation code.

Follow the steps below exactly. Pause where indicated.

# Step 1 — Flow Contract and Plan

I want to build: [prose description, as messy as you want]

Convert this into a flow contract grouped by concern:

## [Concern Name]

Given [precondition] → When [trigger] → Then [outcome + what user sees]
One row per step. Happy path only.

Then output:

SMOKE TEST: [one sentence — what you'd show someone to prove it works]

STATES (if applicable):
| State | Action/Guard | Next State |

KEY ASSUMPTIONS:

- [any data shape, external API, or dependency the plan relies on]

Do NOT write any code.

**PAUSE. Output the above. Ask: "Approve this design? (y/n)"**
**Wait for my response before proceeding to Step 2.**

# Step 2 — Skeleton, Tests, and Manifest

Generate a skeleton and tests from the approved design.
Do NOT implement any logic.

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

MANIFEST (place as comment block at top of skeleton file):
The manifest tells the LLM what exists (behaviors, signatures,
states, assumptions) so it can scope or implement without seeing
the full codebase. Use this exact format:

## [Concern Name]

Given → When → Then
function_name(inputs: types) -> output_type
Tests: [property description]

## [Another Concern]

IMPORTS: function_name -> output_type
Given → When → Then
function_name(inputs: types) -> output_type
Tests: [property description]

## Orchestration

IMPORTS: [all routed functions]
run_command(store, cmd, args) -> tuple(store, str)
Tests: [routing + unknown command properties]

## States

state → [action] → state

## Assumptions

[one line per assumption]

IMPORTS rules:

- If a concern uses functions defined in another concern,
list only their signature as an IMPORTS line.
- IMPORTS shows the dependency shape, not the full GWT/tests.

# Process Instructions

- Be critical. Be creative.
- Answer your own questions with the simplest way that works,
then ask if still unclear.
- Note self-resolved questions under "Q/A (self-resolved):"
Example: Delimiter/encoding? Assume UTF-8, comma.
- Explore 1-2 alternatives. Pick the best. Note alternatives as:
If you [approach], turn to page 42.
BEWARE: [specific downside].
```

### LAYER 1 — IMPLEMENT

#### Human Instructions 

- Build context.
- Repeat for each concern until all tests pass and behavior matches the smoke test. Then proceed to Layer 2.

```other
You implement functions from skeletons until tests pass. You fix
your own errors. You change only what is asked.

[upload/paste Rules.md + skeleton]

All tests are currently failing. Implement [concern — paste the
concern block from your manifest]:

## [Concern Name]

Given [row]
Given [row]

Do not change signatures, orchestration, or tests.
Run tests. If any fail, fix only the failing functions and rerun.
Repeat until green or stuck. If stuck, tell me what's wrong.

After implementation, update the manifest comment at the top of
the file to reflect any changes.

CODE CONSTRAINTS:

- Single file, under 100 lines. If it can't fit, ask me to cut scope.
- No error handling, logging, or types unless I ask.
- No comments unless the logic is non-obvious.
- If unclear, ask — don't guess.
- Never change code I didn't ask to change.
- Do not change signatures unless Design context approved it.
If a signature needs changing, stop and tell me.
- Provide a terse commit message referencing functions updated.
```

### LAYERS 2–6 — LAYER PROGRESSION

#### Human Instructions 

- Each layer follows the same two-step pattern: scope in Design, implement in Build. The manifest grows with each layer.
- See Layer-specific scoping guidance

**Layer-specific scoping guidance:**

| **Layer**    | **Ask Design context to scope**                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| 2 Real data  | “What changes when I replace [fake source] with [real source]? New failure modes? Signature changes?”                    |
| 3 Sad paths  | “Which specific failure cases matter for this manifest? Output new Given-When-Then rows for each.”                       |
| 4 Edge cases | “Which state transition edge cases and guard conditions matter? Output new Given-When-Then rows.”                        |
| 5 Refactor   | “Any structural risks in refactoring to [functional style / new data model / etc.]? What should change, what must stay?” |
| 6 Polish     | “Anything to address before shipping? Types, file splitting, naming?”                                                    |

**Step 1 — Scope (Design context):**

````other
You turn prose project descriptions into flow contracts, plans,
and skeletons. You never write implementation code.

Here is the current manifest:
[paste manifest comment block from top of code file]

I want to add: [layer concern — see guidance table below]

Output:

NEW ROWS (grouped under existing or new concern headings):

## [Concern]

Given → When → Then

NEW FUNCTIONS:
function_name(inputs: types) -> output_type — purpose ≤10 words

NEW TESTS:
assertion description, one per line

TRANSITION TABLE CHANGES:
new states or transitions, if any

MODIFICATIONS TO EXISTING (if necessary):
For each change to an existing row, signature, or transition:
MODIFIED: [what it was] → [what it becomes]
REASON: [why this layer requires it]

If no modifications needed, state: "No changes to existing."

Existing rows, signatures, and transitions are STABLE.
Modify only if genuinely required. New additions need no approval.
Modifications do — wait for my confirmation before proceeding.

```other

```

**Step 2 — Implement (Build context):**

```other
You implement functions from skeletons until tests pass. You fix
your own errors. You change only what is asked.
```

[upload/paste Rules.md + current code]

Implement these approved changes:
[paste Design's output — new rows, signatures, tests,
and any approved modifications]

Do not change signatures unless Design approved a modification.
If a signature needs changing, stop and tell me.
No other changes. Run tests. If any fail, fix and rerun.
Repeat until green or stuck.

After implementation, update the manifest comment at the top of
the file to reflect any changes.

Provide a terse commit message referencing functions updated.
````

### LAYER 7 — REVIEW (optional)

#### Human Instructions 

- Review context. Fresh chat. Any model. One prompt per concern.
- Feed results back to Build context for fixes.

**Review:**

```other
You review finished code against its flow contract.
List problems only. Do not fix them.

[upload Rules.md + code (manifest is at the top of the code)]

Does this enforce the states and transitions in the manifest?
Are there gaps between the flow contract and the implementation?
```

**Simplify:**

```other
You simplify code while keeping behavior identical.

[upload Rules.md + code]

Simplify. Keep behavior identical. Target under 100 lines.
```

**Security:**

```other
You audit code for security vulnerabilities.

[upload Rules.md + code]

Flag vulnerabilities. Do not fix them.
```

---

# Example: Bookmark Manager

**Manifest (lives as comment block at top of code file):**

````other
## Bookmarking
Given no bookmarks exist → When user saves a URL → Then bookmark created with status "unsorted"
Given bookmarks exist → When user adds tags → Then bookmark status becomes "sorted"
  create_bookmark(url: str) -> dict
  add_tags(bookmark: dict, tags: list[str]) -> dict
  Tests: output always has {url, tags, status, created_at}; add_tags never mutates input

## Search

IMPORTS: create_bookmark -> dict
Given bookmarks exist → When user searches by tag → Then matching bookmarks returned
search_by_tag(bookmarks: list[dict], tag: str) -> list[dict]
Tests: empty list when no match; only matching tags returned

## Digest

IMPORTS: create_bookmark -> dict
Given unsorted bookmarks older than 1 day → When digest runs → Then unsorted bookmarks returned
get_unsorted_digest(bookmarks: list[dict], now: datetime) -> list[dict]
Tests: excludes sorted; excludes fresh unsorted

## Orchestration

IMPORTS: create_bookmark, add_tags, search_by_tag, get_unsorted_digest
run_command(store: list, cmd: str, args: dict) -> tuple(list, str)
Tests: routes valid commands; unknown command returns error + unchanged store

## States

unsorted → [add_tags] → sorted

## Assumptions

SQLite storage, no auth, timestamps via datetime

```other

```

**Smoke test:** Save a URL, tag it, search by tag, see unsorted ones in a digest.

**Skeleton (abbreviated — manifest comment block sits above this):**

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
# dict, list[str] -> dict (new bookmark, updated status)
raise NotImplementedError

def search_by_tag(bookmarks, tag):
# list[dict], str -> list[dict]
raise NotImplementedError

def get_unsorted_digest(bookmarks, now):
# list[dict], datetime -> list[dict]
raise NotImplementedError

def run_command(store, command, args):
# list, str, dict -> tuple(list, str)
# Routes to correct function. NO IO here.
raise NotImplementedError

def main():
# IO only: read input, call run_command, print, persist
raise NotImplementedError
```
````

**Layer progression:**

| **Layer** | **Design scopes (using manifest)**                         | **Build implements**                              |
| --------- | ---------------------------------------------------------- | ------------------------------------------------- |
| 0         | Flow contract + manifest + skeleton above                  | —                                                 |
| 1         | —                                                          | Implement concerns one at a time until tests pass |
| 2         | “What changes with SQLite?”                                | Add save/load edge functions, swap store          |
| 3         | “Which failures? → duplicate URL, empty tags, invalid URL” | Handle each, new GWT rows added to manifest       |
| 4         | “Edge cases? → tag already-sorted bookmark”                | Merge tags, no status change                      |
| 5         | “Risks in functional refactor?”                            | Pure functions, immutable updates                 |
| 6         | “Ready to ship?”                                           | Type hints, split files if needed                 |
| 7         | Review / simplify / security                               | Apply fixes                                       |

## License
GPLv3