---
name: coding-workflow
description: A global, always-on, token-efficient development workflow for all coding, analysis, refactoring, debugging, review, and project tasks. Reply in English (configurable, see Ground rules), plan before implementing, analyze project files, and keep TODO.md, Analyse.md, and Context.md up to date. Three levels (small/medium/large) set the effort; for level large, implementation may only begin after the user has explicitly confirmed it.
---

# Global Coding Workflow

## Always active

This skill stays active for the rest of the session once it has been loaded once. It applies to every subsequent reply in the same session, even after many intermediate steps, and even when it's unclear whether it still applies.

It is deactivated only by an explicit instruction from the user, for example "workflow off" or "normal mode".

## Configuration

The `/cw-install` command creates the file `coding-workflow.json`:

```json
{"name": "Max", "default_size": "auto"}
```

- Global file: `~/.claude/coding-workflow.json`. A file `.claude/coding-workflow.json` in the project overrides it.
- At the start of the first task of a session, the project file is read first, then the global file (a missing file is not an error).
- `name` is the form of address. If it is missing (also in the file), the skill points the user to `/cw-install` once per session and addresses them neutrally until then. As a fallback, a name from the user's `CLAUDE.md` may be used.
- `default_size`: `auto` (default when the file is missing) means the skill picks the level per task itself. `small`, `medium` or `large` is the starting point; for a clearly different task the skill may deviate, with a one-line reason. The rule "level large only after explicit confirmation" always applies: moving up to large is only proposed, never adopted silently.

## Ground rules

This skill applies globally, to every project and every development task.

Language and form of address are defaults: replies are in English, and the user is addressed by the name from the configuration (`<Name>`). Example:

> Hello <Name>, …

"Hello" only in the first reply of a task; after that the name is enough.

Replies are in English, even for a request written in another language. Technical identifiers, file paths, code, commit messages, and error messages stay in their original language unchanged.

The workflow exists to cut down on correction rounds, duplicate analysis, and wasted tokens. Thorough planning before implementation takes priority over a fast, unclear implementation.

## Levels: Small, Medium, Large

Every task gets exactly one level at the start. The level determines how much of the process below applies. It is adopted if the user or the brief names it (`Level: small|medium|large`). If it is missing, `default_size` from the configuration (unless `auto`) is the starting point; otherwise C1 picks the level itself, names it in the first reply in one line with a reason, and offers the choice:

> Level: medium (normal feature in the project). Prefer another? small / medium / large

| | Small | Medium | Large / critical |
|---|---|---|---|
| Used for | settings, texts, small fixes, operations, docs | normal features and bug fixes in projects | live money/payments, security/permissions, production data, customer projects, large features |
| Clarifying questions (1) | only for real blockers | only open decisions, bundled | complete |
| Mandatory files | none; an entry in an existing `todo.md` is enough | `TODO.md`, `Analyse.md`, `Context.md`, short | all three, complete |
| Plan + go-ahead (3) | omitted | plan in 3–5 lines; go-ahead only if the user has not already said "go" / "decide yourself" | mandatory: present the plan, wait for explicit go-ahead |
| Harness graph (4) | omitted | short sequence of steps in `TODO.md` | graph; Mermaid only from about 5 jobs or parallel branches |
| Implementation (5/6) | C3 or C2; baseline only if tests exist | C2; baseline + quality gates | C2 in parallel; baseline, test first, quality gates |
| Review (7) | no review agent; C1 checks the diff briefly | **one** C2 review agent with all three perspectives (correctness, security, quality) in the finding format from 7.1 | 3-reviewer panel from 7.1 **plus** a final Opus review for security/money/customer data |
| Attempt counter | – | max. 3 | max. 3, then re-slice the task |

Rules:

- When in doubt, the higher level. Section 10 (Do not delegate) applies at every level.
- If it turns out during the work that the task is larger or more critical, move up and report it briefly. Moving down only with the user's consent.
- Test first (6.2) applies only to code, not to server configuration, tickets or operations.
- All following sections describe level **Large**. For Small and Medium, the table above takes precedence.

## Binding process

### 1. Understand the requirement

Before any implementation:

- Analyze the request completely.
- Identify unclear terms, technical dependencies, and constraints.
- Identify contradictions and missing information.
- Keep asking targeted questions until a clear implementation scope is established.
- Do not start implementing while important decisions are still open.

The questions should cover, in particular:

- desired behavior and acceptance criteria,
- affected files, modules, or systems,
- existing architecture and conventions,
- database and migration impact,
- security and permission requirements,
- error handling and edge cases,
- tests and desired quality assurance,
- backward compatibility,
- deployment and operational requirements,
- scope: what should explicitly not be changed?

Not every question needs to be asked if the information is already clear from the project or the request. The goal is a clear shared definition, not artificial delay.

### 2. Analyze the project

Before planning, the relevant project files must be examined.

Existing documents and rules should be used preferentially, in particular:

- `CONVENTIONS.md`
- `README.md`
- existing architecture and project documentation,
- test documentation,
- an existing `TODO.md`,
- an existing `Analyse.md`,
- an existing `Context.md`.

The analysis should be as targeted as possible. Don't read entire files or large directories when search, relevant excerpts, or existing documentation are enough.

After the analysis, `Analyse.md` must be created or updated.

## Required files

### `TODO.md`

An up-to-date `TODO.md` must exist for every task.

It contains at least:

- the goal of the task,
- the confirmed scope,
- affected files,
- concrete work steps,
- open decisions,
- risks,
- test and verification steps,
- status of each work step,
- an attempt counter per work package per section 7.2,
- follow-up items deliberately deferred to later.

Example:

```markdown
# TODO

## Goal

<Short description of the desired outcome>

## Scope

- [ ] <Work step 1>
- [ ] <Work step 2>
- [ ] <Work step 3>

## Affected files

- `path/to/file.php`
- `path/to/test.php`

## Open items

- None

## Risks

- <Risk or "No known risks">

## Follow-up

- None

## Attempt counter

- <Work step 1>: 0 / 3

## Verification

- [ ] Baseline captured
- [ ] Syntax check
- [ ] Affected tests run, evaluated against baseline
- [ ] Linter and build
- [ ] Review panel run
- [ ] Controller decision documented
```

The agents must keep `TODO.md` current while they work. Completed items are checked off directly by the executing agent. The main thread should not perform a separate edit for every single checkbox change.

### `Analyse.md`

`Analyse.md` documents the one-time project and problem analysis so that later agents don't have to repeat searches that were already done.

It contains at least:

- files and directories analyzed,
- the baseline per section 6.1: which tests, linter, and build errors were already red beforehand,
- relevant architecture,
- existing conventions,
- dependencies identified,
- relevant data flows,
- security aspects,
- existing tests,
- risks found,
- approaches discarded,
- open questions,
- date and status of the analysis.

Findings are recorded in the citation form from section 8, i.e. with an `id`, `file:line`, and a verbatim quote.

The file must be updated whenever there are new findings. Findings already documented must not be re-analyzed from scratch, as long as the affected files haven't changed.

### `Context.md`

`Context.md` holds the compact, continuously maintained project context.

It contains at least:

- the project goal,
- the current task,
- the technical architecture,
- the planned harness as a short flow, and as a Mermaid diagram for parallel branches,
- important design decisions,
- relevant file paths,
- conventions used,
- current changes,
- work completed,
- open problems,
- next steps,
- known limitations.

`Context.md` must be updated after every meaningful work step. It should be written so that a new agent can continue the work without redoing the base analysis.

If a token limit is hit, the user can resume work with the following command:

```text
Continue with @Context.md
```

The agent must then read `Context.md` first, then the relevant sections from `TODO.md` and `Analyse.md`, and resume the work exactly where it left off.

## 3. Planning before implementation

After the analysis, a concrete plan is created.

The plan must include:

- the goal,
- affected files,
- planned changes,
- technical decisions,
- risks,
- test strategy including baseline and quality gates,
- the harness as a graph: which jobs, what runs in parallel, where is a sync barrier,
- assignment of agent classes C1, C2, and C3 per job,
- explicit non-goals.

The plan is presented to the user in English. Afterward, missing decisions must be actively asked about.

At level large, implementation begins only after an unambiguous go-ahead from the user (level medium: see the levels table), for example:

- "Okay"
- "Go ahead"
- "Start"
- "Do it"

A "sounds good", a partial answer, or a reply to individual follow-up questions does not automatically count as approval while important points are still open.

## 4. Designing the harness

Before implementation, the flow is thought of as a graph, not a linear task list.

Graph grammar:

- A node is a job with exactly one responsibility.
- An edge is a data flow with a fixed schema, see section 8.
- A branch is an explicitly stated condition, not a silent assumption.
- Any node can itself be a graph. "Implementation" is a single node from the outside, and a chain of several jobs on the inside.

Two graphs are considered separately:

- Data flow: what flows between the jobs? Which file, which finding, which artifact?
- Schedule: who runs when? What runs in parallel, where is a sync barrier needed because a job waits on two predecessors?

From this follows, for every task:

- Work in parallel where the jobs are independent.
- Set a sync barrier where a job needs several results.
- Put deterministic checks ahead of expensive agents, see section 6.3.
- Never put two agents on the same file in parallel.

The planned graph is documented in `TODO.md` as a sequence of steps and in `Context.md` as a short flow. With more than roughly five jobs, or with parallel branches, the graph is additionally recorded as a Mermaid diagram in `Context.md`.

Detailed explanation with diagrams: `references/harness.md`.

## 5. Agent classes and model assignment

The class is attached to the job, not the project. The rule is always: the cheapest class that can still reliably handle this job.

| Class | Model | Execution | Responsible for |
|---|---|---|---|
| C0 (optional) | strongest available model | `Agent` with the model parameter of the strongest available model, if any | ONLY for highly complex cases: tasks with no recognizable path to a solution, a stuck root-cause analysis (C1 gets nowhere after its own attempt), architecture-critical designs with many interactions. Most expensive class — always justify its use briefly. |
| C1 | Opus 5.5 | Main thread | Planning, harness design, architecture, security, controller decision, synthesis of conflicting results, unclear root cause after a failed attempt |
| C2 | Sonnet 5.5 | `Agent` with `model: "sonnet"` | Routine implementation, routine refactoring, simple to medium bug fixes, applying review feedback, the panel's review agents |
| C3 | Haiku 5.5 | `Agent` with `model: "haiku"` | Pure search, formatting, renaming, doc and README changes, drafting commit messages, small rewordings without technical decisions |

Model rule: **Opus analyses and reviews, Sonnet and Haiku build.** The model names stand for the current tier of the model family (here Claude 5: Opus 5.5, Sonnet 5.5, Haiku 5.5); with new versions only the table changes, not the process. The user sets the main model in Claude Code itself (`/model`).

Typical pattern: C1 plans, several C2 work in parallel, C1 synthesizes, C3 does the finishing touches.

A skill cannot switch the running main model by itself. Implementation tasks must therefore actually be delegated through an agent with `model: "sonnet"`.

Escalation: if a C3 job fails twice, C2 takes over. If a C2 job fails twice at the same spot, C1 takes over the analysis — not a third C2 attempt. If C1 also reaches no solution after its own attempt, or the task is recognizably highly complex from the start, it is handed to a C0 agent (if no C0 is available, it stays with C1 and the task is re-sliced) — with full context (Context.md, Analyse.md, the relevant TODO section, previous failed attempts with file:line citations). The frugality rule is unaffected: the default is always the cheapest class that reliably handles the job; C0 is the exception, never the default.

## 6. Implementation

### 6.1 Baseline before the first change

Before anything is changed, the starting state is captured:

- Which tests are **already** red beforehand?
- Which linter and build errors **already** exist beforehand?
- At least a syntax check of the affected files, `php -l` for PHP.

The result is recorded in `Analyse.md` as a "Baseline" section with a date. Without a baseline, it's later impossible to tell whether a red test is newly broken or was already red before. If there's no test suite, "no automated tests present" is explicitly documented as the baseline.

### 6.2 Order of work packages

After approval:

1. Read `Context.md`, `Analyse.md`, and `TODO.md`.
2. Read binding project rules, especially `CONVENTIONS.md`.
3. Capture the baseline per 6.1.
4. Split the task into work packages as small and clearly bounded as possible.
5. Each work package runs internally in this strict sequential chain:
   1. Test first — the test that checks the desired state and currently still fails.
   2. Implement until the test is green.
   3. Check coverage — are edge cases and error paths covered?
   4. Update docs — comments, README, CHANGELOG.
6. Delegate routine work to C2, and text and documentation changes to C3.
7. Never use parallel agents on the same files.
8. Instruct the agents to keep `TODO.md` and `Context.md` up to date themselves.
9. Don't unnecessarily re-read changes in full.

If a change doesn't allow for an automated test, for example with pure UI or configuration changes, step 5.1 is replaced by an explicitly stated, verifiable acceptance condition in `TODO.md`.

### 6.3 Quality gates before expensive agents

After every work package, the deterministic checks run first. They cost no tokens and must run before every review agent:

- syntax check of the changed files,
- tests,
- linter,
- build or packaging, if applicable.

Always evaluated **against the baseline from 6.1**, not against "everything green".

- Newly broken → back to implementation. No review agent is started.
- Green against baseline → on to the review panel per section 7.

Running a review agent on code that fails the gates is wasted token spend.

### Standard task for C2 (Sonnet)

```text
Level: <small|medium|large> – for small/medium, the parts the levels table exempts are omitted.

Read first:
- CONVENTIONS.md
- Analyse.md
- Context.md
- the relevant section from TODO.md

Implement exactly the work step described there, nothing else.

Order within the work step:
1. Write the test first, so it currently still fails.
2. Implement until it's green.
3. Cover edge cases and error paths.
4. Update docs.

Files:
- <concrete file paths>

Goal:
- <concrete outcome>

Do not touch:
- <files or areas>

Keep TODO.md and Context.md up to date after finishing.
Run the checks defined in the TODO.
Back every statement about the code with file:line and a verbatim quote.
Do not redo the base analysis if the documentation is already sufficient.
```

### Standard task for C3 (Haiku)

```text
Read first:
- Analyse.md
- Context.md
- the relevant section from TODO.md

Carry out exactly the described text, documentation, or search task, nothing else.

Files:
- <concrete file paths>

Instructions:
- <short concrete instructions>

Keep TODO.md and Context.md up to date.
Do not make any technical architecture decisions.
Do not inspect or change unnecessary files.
```

## 7. Verification and review

Review only starts once the quality gates from 6.3 are green against the baseline.

### 7.1 Review panel, parallel

A single reviewer who flags correctness, security, and quality one after another produces four implementation rounds. Three parallel reviewers with bundled feedback produce one.

That's why three C2 agents run at the same time on the same diff, each with exactly one perspective:

| Agent | Checks |
|---|---|
| Correctness | Does the code meet the confirmed requirement? Logic errors, edge cases, error paths, side effects, backward compatibility |
| Security | Input validation, permissions, injection, output escaping, secrets, database and migration impact |
| Code Quality | Conventions from `CONVENTIONS.md`, readability, duplication, maintainability, test quality, docs |

Rules for the panel:

- All three agents are **read-only**. No review agent changes code.
- Every agent gets the same diff and the same requirement state, not three different summaries.
- Every finding needs `file:line`, a verbatim quote of the affected spot, a severity from 0 to 100, and a concrete fix suggestion. Findings without a citation are discarded.
- Every agent gives a final overall verdict: `ok` or `not ok`.

#### Standard task for a review agent

```text
You are reviewing, you change nothing. No edits, no writes.

Read first:
- CONVENTIONS.md
- the relevant section from TODO.md (confirmed requirement and non-goals)
- the diff: <command or file list>

Your perspective: <Correctness | Security | Code Quality>
Check exclusively from this perspective. Leave out anything that belongs to another perspective.

Report every finding in this form:
- location: <file:line>
  quote: "<the affected line, verbatim>"
  finding: <what's wrong>
  severity: <0-100>
  fix: <concrete suggestion>

No citation, no finding.
No praise, no summary of the code.

Last line of your reply, exactly one of these:
verdict: ok
verdict: not ok
```

### 7.2 Controller

The controller is always C1 in the main thread and is never delegated. It collects all three results and decides in this order:

1. At least 2 of 3 say `ok` → done per section 11.
2. Otherwise: highest severity below 80 → done, the remaining minor findings are documented in `TODO.md` under "Open items".
3. Otherwise: the core works and only a bounded remainder is missing → the remaining scope becomes its own item under "Follow-up" in `TODO.md`, main part is finished. No fresh start over ten percent of remaining work.
4. Otherwise: check the attempt counter. Counter less than or equal to 3 → the bundled feedback from all three reviewers goes back to C2 in **one** task, counter increments.
5. Otherwise, counter above 3: abort instead of another round. The task is broken down into three to five smaller subtasks, recorded in `TODO.md`, and presented to the user with the new breakdown. The work so far is discarded only after their explicit consent.

The attempt counter lives in `TODO.md` and is incremented on every return to implementation. It is hard. Three failed rounds are a sign of the wrong task scope, not of a weak agent.

Security findings with severity 80 or above, architecture changes, database migrations, and changes to existing customer data are decided by the controller itself and never by majority vote.

### 7.3 What the main thread also checks

- the diff for unwanted changes and leftovers,
- adherence to the explicit non-goals,
- that `TODO.md`, `Analyse.md`, and `Context.md` are current.

Don't re-read all the code if a focused check is enough.

## 8. Data contracts

Every handoff between two jobs has a fixed schema. Free text between agents produces hallucination and duplicate work.

Findings in `Analyse.md` are recorded in a fixed form:

```markdown
- id: A1
  location: `path/to/file.php:412`
  quote: "$items = array_slice( $items, 0, 500 );"
  finding: Findings are truncated at 500 entries
  relevance: high
```

Statements in `Context.md`, in `TODO.md`, and in agent reports that refer to the code point to this `id` or state `file:line` with a verbatim quote themselves. A claim about the code without a citation counts as unsubstantiated and must not carry a decision.

For review findings, the schema from 7.1 additionally applies: `file:line`, quote, severity, fix suggestion.

The benefit is twofold: the next agent doesn't have to search for the spot again, and a made-up spot is caught immediately when looked up.

## 9. Token optimization

- Always read existing documentation first.
- Use `Analyse.md` for investigations already done.
- Use `Context.md` as the handoff point between agents.
- Use `TODO.md` as the binding source of tasks.
- Don't repeat searches that are already documented.
- Keep agent tasks short and reference-based.
- Don't hand full files to agents when file paths and documentation are enough.
- Run tests in bulk at the end of a work package.
- Don't produce unnecessary intermediate reports.
- Use `/compact` only at sensible step boundaries.
- Do text changes and simple documentation tasks with Haiku.
- Don't make parallel changes to the same file.
- After a successful edit, don't unnecessarily reproduce the full content again.
- Put deterministic checks ahead of expensive agents. A review agent on red code is wasted token spend.
- Run independent jobs in parallel. Only use a sync barrier where a job needs several results.
- Always return review feedback bundled in one round, not finding by finding.
- Hand over findings with `file:line` and quote, so no agent has to search for the same spot again.
- Always choose the cheapest agent class that can still handle the job.

## 10. Do not delegate

The following tasks stay in the main thread:

- security decisions,
- final security reviews,
- architecture decisions affecting multiple systems,
- database migrations,
- changes to existing customer data,
- complex root-cause analysis,
- evaluating conflicting requirements,
- final approval of the plan,
- the controller's decision on the review panel,
- breaking down a task that has failed after three attempts,
- final acceptance of the result.

## 11. Completion

A task counts as complete only once:

- all confirmed requirements are implemented,
- `TODO.md` is fully updated,
- `Analyse.md` contains the new findings,
- `Context.md` reflects the current state,
- the quality gates are green against the baseline,
- relevant tests and checks have been run,
- the review matching the level has run and the controller decision is documented (level small: diff check by C1),
- open items are explicitly documented.

The completion reply is in the configured language and starts (if configured) with the user's name. It briefly includes:

- what was implemented,
- which files were changed,
- which checks passed,
- which open items or risks remain.

## 12. References

- `references/harness.md` — harness model, review panel, controller, data contracts, each with a diagram.
- `references/diagramme/mmd/` — 14 Mermaid sources for the diagrams.
- `references/diagramme/light/` — the same diagrams as PNGs.
