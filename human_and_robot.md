# The Human and the Robot - An organic model for use of AI 

## Overview

A minimal framework that helps the human use AI strategically as a programmable collaborator to get work done and improve workflows, rather than exclusively using AI to help perform otherwise manual work. It organizes work around **structured context (memory), reusable thinking (knowledge), and command**-triggered **pipelines**. The system captures context specific constraints, injects frameworks/templates, and produces artifacts for re-use.

**Slash commands** route tasks through lightweight pipelines using **contextual state cards (memory)** and **frameworks/templates (knowledge)** stored in a prescribed folder structure. Pipelines generate **artifacts** (docs, analyses, updates) and automatically update scope or add constraints to a specific context (such as a project, general task, or area of obligation).

The system is built as a walking skeleton, used to build itself, and emphasizes quality and deterministic outputs wherever possible.

### Definitions

- **opportunities.md**: a list of what you want to accomplish and hope AI can make easier. Format: trigger + acceptance criteria.
- **pipelines**: an orchestration of agents, humans, and automation to accomplish an objective.
- **tasks**: intrmediary outcomes using specific tools (e.g., convert this Excel, draft this email).
- **manifest.md**: a design contract for AI that includes input/output schema, invariants (must/not…), tools, an evaluation harness such as golden examples of ~5–20 tests, rubric scoring, or regression checks, ad governance. For humans it is a card-sized overview of the current objective, referencing or linking to resources like a save card for video game.
- **rules.md:** accumulation of one system-wide, framework-specific and context-specific one-liners that correct behavior and embed best practices for AI, and that guide human behavior for earlier, more informed decisions, and better habits.
- **agents**: AI task executors generalized across contexts and knowledge (preferences/methods).
- **humans**: did you pass the test?
- **artifacts**: outputs produced by tasks or pipelines that can be stored for reuse or reference, including deliverables, memories, and knowledge.
- **memory**: capture evolutionary changes within a context, such as:
    - **state:** objective, inputs, outputs, constraints, plan/steps, questions, status
    - **decisions:** that are approved by the user, and including rationale plus any structured outputs
    - **preferences:** e.g. tone, style, format, approach, etc.
- **state.md:** stores state, decisions, and preferences for a specific context in data fields plus an append log: date | type (scope_change, constraint) | change | reason | impact
- **knowledge**: AI and human focused stepwise frameworks, fillable templates, scoreable rubrics, targeted recipes, and anti-patterns/guards for consistent reasoning and decision making.
- **router**: AI agent that determines which pipeline runs, retrieves context (state, recent changes, preferences), and injects relevant items from the knowledge base (**templates, rubric**)
- **commands**: modeled after slash commands to initiate pipelines or tasks (meeting notes, critique, create deliverable), perform capture (opportunities, memories, learnings),

### Folder Structure:

- ops/opprtunities.md
- contexts/<c>/state.md + memories + artifacts + rules.md
- router/router_rules.md
- kb/frameworks/<f>.md + rules.md
- kb/recipes<r>.md (tasks, commands, etc.)
- kb/templates/<t>.md
- pipelines/<a>.file (agent backup), <p>_documentation.md (with links and change log) + <a>_manifest.md

### **LLM pipeline design layers**:

1. **skeleton / MVP** (minimal end-to-end flow)
2. **structured inputs** (convert messy user input into usable LLM structure)
3. **output normalization + packaging** (format, tone, structure)
4. **critique loop** (check invariants vs rubric; auto-revise if needed)
5. **retrieval augmentation** (pull state, decisions, templates, rubrics)
6. **instrumentation** (logs, decision/change analysis)
7. **automation hooks** (trigger memory updates, artifact storage, downstream pipelines)

### Three Contexts

**Design** — Scope what to build. Reasoning model, kept separate. This context accumulates your intent and produces contracts, plans, and skeletons. It never writes implementation code.

**Build** — Implement and test. Execution model, sustained across layers. This context fills in function bodies, fixes errors, and applies layer changes. It follows the plan. It doesn’t make design decisions.

**Review** — Fresh eyes on finished work. Any model, fresh context. Lists problems, simplifies, flags security issues. Does not fix — results go back to Build. Optional for personal tools. Recommended before anything user-facing ships.

## The Human

### Accumulate

Accumulate lessons learned, opportunities, and changes in opportunities.md. These get periodically reviewed and implemented as code or converted to knowledge as frameworks.

### Triggers

Every systemic change in approach, rabbit hole, human mistake, oe lesson learned gets added to triggers.md. These guide human behavior to make more informed decisions earlier and to correct habits in this quickly shifting landscape

EXAMPLE: 

- Reading code before running? Refactoring what works? Debating approaches? Edge cases before happy path? "Let me just understand..."? → **Stop and run the loop.**

### Manifest

For every project there will be an exit condition--acceptance criteria--a series of steps that need to happen next, and a current working surface comprised of knowledge, learnings, half completed steps, and a workspace. This is captured in the human manifest--a card sized (two sided) terse overview of the current step. 

## The Robot

### Accumulate 

Accumulate decisions, preferences, and changes to scope in a per project state.md. This gets reviewed periodically and implemented as part of the manifest or converted to domain knowledge or rules.md.

### Rules

These are the guardrails that govern behavior of the LLM at various stages. They correct behavior and embed best practices.

After every LLM mistake: (1) What did it do wrong? (2) Write a one-line rule that prevents it. (3) Add it permanently.

Rules.md non-negotiable rules are included in the system prompt / persona / agent instructions (processed every turn). Project-specific rules in the Manifest (uploaded per session and incorporated into prompt).

### Manifest 

The manifest tells the LLM what exists (behaviors, signatures, states, assumptions) so it can scope or implement without seeing the full project, and serves as a design contract for implementation and quality control. The human collaborates with a planner LLM to modify the manifest, and the manifest represents the knobs and dials for the human.

## License
GPLv3