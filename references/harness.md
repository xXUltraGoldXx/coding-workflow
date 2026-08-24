# Harness Engineering — Explanation of the Workflow

This document explains the rules from `SKILL.md` and shows the diagrams that go with them. `SKILL.md` is binding; this document explains and justifies.

Core idea: the model alone doesn't determine the result — the flow it operates in does. The same model reaches a noticeably better result in a well-built harness than in the default flow.

```mermaid
---
config:
  xyChart:
    width: 1100
    height: 500
  themeVariables:
    xyChart:
      plotColorPalette: "#0072B2"
---
xychart-beta
    title "ARC-AGI-3 — the harness decides, not just the model"
    x-axis ["Opus 5 | official", "GPT-5.6 Sol | official", "GPT-5.6 Sol | custom harness"]
    y-axis "Score in %" 0 --> 50
    bar [30.2, 7.8, 40]
```

Source: `references/diagramme/mmd/00-benchmark.mmd`, image: `references/diagramme/light/00-benchmark.png`

## 1. Graph grammar

A flow consists of jobs as nodes and data flows as edges. Branches are explicitly stated conditions.

```mermaid
flowchart LR
    START([Start]) --> J("Job")
    J --> IF{"if"}
    IF -->|Case A| A("Job A")
    IF -->|Case B| B("Job B")
    IF -->|Case C| C("Job C")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    class J,A,B,C agent
    class START term
    class IF dec
```

Source: `references/diagramme/mmd/01-graph-grammatik.mmd`, image: `references/diagramme/light/01-graph-grammatik.png`

The key point is recursion: any node can itself be a graph. From the outside, "implementation" is one node; on the inside it's a chain or a fan-out of several jobs followed by synthesis.

```mermaid
flowchart LR
    subgraph JOB1["Job 1 — a single node from the outside"]
        direction LR
        IN("Job 1") --> P1("Job 1'")
        IN --> P2("Job 1''")
        IN --> P3("Job 1'''")
        P1 --> SY("Job 1 Synth")
        P2 --> SY
        P3 --> SY
    end

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    class IN,P1,P2,P3,SY agent
```

Source: `references/diagramme/mmd/02-job-ist-selbst-graph.mmd`, image: `references/diagramme/light/02-job-ist-selbst-graph.png`

Practical consequence for planning: settle the top-level graph first, then only expand the nodes that genuinely need several jobs. A node that can't meaningfully be expanded is a single agent task.

## 2. The full harness

The following diagram shows the reference flow. It is the template for sections 6 and 7 of `SKILL.md`.

```mermaid
flowchart TD
    START([new GitHub issue]) --> BASE["Baseline tests<br/>which tests are <b>already</b> red beforehand?"]
    BASE --> READ("Issue reader<br/>gh issue view + comment")
    READ --> RES("Research agent<br/>gather codebase context")
    RES --> IMPL

    subgraph IMPL["Implementation — 1 node from the outside, 4 agents inside, strictly sequential"]
        direction LR
        TDD("1. TDD<br/>tests first") --> CODE("2. Implement")
        CODE --> COV("3. Coverage")
        COV --> DOCS("4. Docs")
    end

    IMPL --> QG["Quality gates<br/>tests + linter + build<br/>deterministic, 0 tokens"]
    QG --> QGD{"all green?<br/>checked against baseline"}
    QGD -->|"no — newly broken"| IMPL

    QGD -->|yes| RC("Review: Correctness")
    QGD -->|yes| RS("Review: Security")
    QGD -->|yes| RQ("Review: Code Quality")

    RC --> CTRL("Controller<br/>aggregates all reviews")
    RS --> CTRL
    RQ --> CTRL

    CTRL --> VOTE{"at least 2 of 3<br/>say OK?"}
    VOTE -->|yes| FIN
    VOTE -->|no| SEV{"severity<br/>above threshold?<br/>e.g. 80%"}
    SEV -->|"no — minor stuff"| FIN
    SEV -->|yes| FU{"follow-up ticket<br/>feasible?"}
    FU -->|"yes — 90% works"| FUA("Follow-up agent<br/>remaining scope as new issue")
    FUA --> FIN
    FU -->|"no — code doesn't run at all"| ATT{"attempt count > N?"}
    ATT -->|no| IMPL
    ATT -->|yes| SPLIT("Split agent<br/>original ticket into 3-5 sub-tickets<br/>close original, revert code")

    FIN("Finalize agent<br/>commit, PR, close issue") --> DONE([done])
    SPLIT --> RESTART([new smaller issues<br/>harness starts over])

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef det fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    class READ,RES,TDD,CODE,COV,DOCS,RC,RS,RQ,CTRL,FUA,SPLIT,FIN agent
    class BASE,QG det
    class QGD,VOTE,SEV,FU,ATT dec
    class START,DONE,RESTART term
```

Source: `references/diagramme/mmd/03-autodev-harness-full.mmd`, image: `references/diagramme/light/03-autodev-harness-full.png`

A more compact version of the same flow:

```mermaid
flowchart LR
    START([Issue]) --> BASE["Baseline<br/>tests"] --> READ("Read<br/>issue") --> RES("Research")
    RES --> IMPL("Implementation<br/><small>TDD -> Code -> Coverage -> Docs</small>")
    IMPL --> QG{"Quality gates<br/>tests + linter<br/>0 tokens"}
    QG -->|"red"| IMPL

    QG -->|"green"| RC("Correctness")
    QG -->|"green"| RS("Security")
    QG -->|"green"| RQ("Quality")
    RC --> CTRL("Controller")
    RS --> CTRL
    RQ --> CTRL

    CTRL --> D{"2/3 OK?<br/>else: severity?<br/>else: follow-up?<br/>else: attempts > N?"}
    D -->|"OK / minor"| FIN("Finalize")
    D -->|"follow-up"| FIN
    D -->|"rework"| IMPL
    D -->|"attempts > N"| SPLIT("Split<br/>ticket")
    FIN --> DONE([done])
    SPLIT --> DONE

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef det fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    class READ,RES,IMPL,RC,RS,RQ,CTRL,FIN,SPLIT agent
    class BASE det
    class QG,D dec
    class START,DONE term
```

Source: `references/diagramme/mmd/04-autodev-harness-16zu9.mmd`, image: `references/diagramme/light/04-autodev-harness-16zu9.png`

The three load-bearing ideas:

1. Baseline first. Without knowing the prior state, "test is red" is not information.
2. Deterministic gates before expensive agents. Tests, linter, and build cost no tokens. A review agent on red code costs tokens and delivers findings the compiler already knew about.
3. No unbounded rework. Either a majority, or accept minor issues, or split off the remainder as a follow-up, or, once the attempt counter is hit, re-scope the task.

## 3. Sequential review vs. parallel review

Sequential: every reviewer produces its own implementation round. Four rounds for three perspectives:

```mermaid
flowchart TB
    I1("Impl v1") --> C1("Correctness")
    C1 -->|"'not like this'"| I2("Impl v2")
    I2 --> C2("Correctness ok") --> S2("Security")
    S2 -->|"'not like this'"| I3("Impl v3")
    I3 --> S3("Security ok") --> Q3("Quality")
    Q3 -->|"'not clean code'"| I4("Impl v4")

    classDef bad fill:#D55E00,stroke:#7a3600,color:#ffffff
    class I1,I2,I3,I4,C1,C2,S2,S3,Q3 bad
```

Source: `references/diagramme/mmd/05-review-sequenziell.mmd`, image: `references/diagramme/light/05-review-sequenziell.png`

Parallel with bundled feedback: one round.

```mermaid
flowchart TB
    PI("Impl v1") --> PC("Correctness")
    PI --> PS("Security")
    PI --> PQ("Quality")
    PC --> PCT("Controller<br/>all feedback bundled")
    PS --> PCT
    PQ --> PCT
    PCT -->|"once, back with everything at once"| PI2("Impl v2")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    class PI,PC,PS,PQ,PCT,PI2 agent
```

Source: `references/diagramme/mmd/06-review-parallel.mmd`, image: `references/diagramme/light/06-review-parallel.png`

That's why the three reviewers in `SKILL.md` section 7.1 run at the same time on the same diff, and the controller bundles the results.

## 4. Controller

The controller decides; it doesn't review itself. It stays in the main thread with C1, because that's where the expensive wrong decisions happen.

```mermaid
flowchart TD
    RC("Correctness") --> CTRL
    RS("Security") --> CTRL
    RQ("Code Quality") --> CTRL
    CTRL("<b>Controller</b><br/>aggregate, weigh, decide on reviews")

    CTRL --> V{"majority OK?<br/>2 of 3"}
    V -->|yes| FIN(["Finalize"])
    V -->|no| SEV{"severity > 80%?"}
    SEV -->|no| FIN
    SEV -->|yes| FU{"follow-up<br/>feasible?"}
    FU -->|yes| FUT("create follow-up ticket") --> FIN
    FU -->|no| N{"attempts > N?"}
    N -->|no| IMPL(["back to implementation"])
    N -->|yes| SPL("split ticket<br/>N sub-tickets, close original, revert code")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    class RC,RS,RQ,CTRL,FUT,SPL agent
    class V,SEV,FU,N dec
    class FIN,IMPL term
```

Source: `references/diagramme/mmd/10-controller-detail.mmd`, image: `references/diagramme/light/10-controller-detail.png`

The order of checks is chosen deliberately:

- Majority before a single opinion: one reviewer with a strict standard must not be able to trigger an endless loop.
- Severity before completeness: a formatting note doesn't block a finished fix.
- Follow-up before starting over: if ninety percent works, the rest becomes its own item, not the justification for discarding everything.
- Attempt counter before persistence: three failed rounds are a scoping problem. The answer is a smaller task cut, not a fourth attempt.

Deviation from the template: the controller presents splitting the task and discarding the current state to the user. It doesn't do either on its own.

## 5. Data contracts

Every edge has a schema. That way the receiver knows what it's getting, and the sender knows what it has to deliver.

```mermaid
flowchart LR
    J1("Job 1<br/>Research") -->|"DO1"| J2("Job 2<br/>Condense")
    DO1[("DO1 — source list")]
    J1 -.->|"produces"| DO1
    DO1 -.->|"consumed by"| J2

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef data fill:#009E73,stroke:#00503a,color:#ffffff
    class J1,J2 agent
    class DO1 data
```

Source: `references/diagramme/mmd/07-data-contract-kante.mmd`, image: `references/diagramme/light/07-data-contract-kante.png`

A schema consists of objects with required fields. What matters against hallucination is identity and citation: an `id`, a verbatim quote, and a pointer from a derived fact back to its source.

```mermaid
classDiagram
    class DO1_SourceList {
        +Source[] sources
        +string topic
    }
    class Source {
        +string id
        +string url
        +string quote  «verbatim from the page»
        +string relevance
    }
    class DO2_Factsheet {
        +Fact[] facts
    }
    class Fact {
        +string statement
        +string source_id  «FK to Source.id»
        +string quote
    }
    DO1_SourceList "1" o-- "n" Source
    DO2_Factsheet "1" o-- "n" Fact
    Source <.. Fact : substantiated by
```

Source: `references/diagramme/mmd/08-data-contract-schema.mmd`, image: `references/diagramme/light/08-data-contract-schema.png`

Applied to our workflow:

- `Analyse.md` corresponds to the source list. Every finding has an `id`, `file:line`, and a verbatim quote.
- `Context.md` and agent reports correspond to the factsheet. Every statement about the code carries an `id` or its own citation.
- Review findings additionally carry severity and a fix suggestion.

A statement without a citation is not a basis for a decision. This isn't a formality: a made-up citation gets caught immediately when looked up; made-up prose doesn't.

## 6. Plan data flow and schedule separately

Two questions, two graphs. What flows, and who runs when. Mixing the two creates dependencies nobody controls.

```mermaid
flowchart TB
    subgraph DFG["DFG — Data Flow Graph: WHAT flows"]
        direction LR
        DO1[("DO1<br/>source list")] --> M(( ))
        DO2[("DO2<br/>repo/domain context")] --> M
        M --> DO3[("DO3<br/>briefing")]
        M --> DO4[("DO4<br/>expanded sources")]
    end

    subgraph TSG["TSG — Task Schedule Graph: WHO runs WHEN"]
        direction LR
        T1("T1 — research") --> SYNC{{"sync barrier<br/>waits for T1 AND T3"}}
        T3("T3 — load context") --> SYNC
        SYNC --> T2("T2 — condense")
        T2 --> T4("T4 — write briefing")
        T2 --> T5("T5 — expand sources")
    end

    DO1 -. "produced by" .- T1
    DO2 -. "produced by" .- T3
    DO1 -. "input to" .- T2
    DO2 -. "input to" .- T2
    DO3 -. "produced by" .- T4
    DO4 -. "produced by" .- T5

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef data fill:#009E73,stroke:#00503a,color:#ffffff
    classDef sync fill:#E69F00,stroke:#7a5300,color:#111111
    class T1,T2,T3,T4,T5 agent
    class DO1,DO2,DO3,DO4 data
    class SYNC sync
    style M fill:#009E73,stroke:#00503a,color:#ffffff
```

Source: `references/diagramme/mmd/09-dfg-ueber-tsg.mmd`, image: `references/diagramme/light/09-dfg-ueber-tsg.png`

The sync barrier is the point where a job waits on several predecessors. It must be named explicitly, otherwise an agent starts off with half the data.

## 7. Agent classes

Classes instead of model names, so the assignment stays stable when the models change.

```mermaid
flowchart LR
    subgraph K1["Class 1 — expensive, capable, few errors"]
        M1["Fable<br/>GPT-5.6 Sol"]
    end
    subgraph K2["Class 2 — workhorses"]
        M2["Opus 5<br/>Kimi K3<br/>Qwen 3.8"]
    end
    subgraph K3["Class 3 — cheap, for small stuff"]
        M3["Minimax M3<br/>DeepSeek V4 Flash<br/>GPT-5.6 T.<br/>Sonnet 5"]
    end
    K1 --> K2 --> K3

    classDef c1 fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef c2 fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef c3 fill:#009E73,stroke:#00503a,color:#ffffff
    class M1 c1
    class M2 c2
    class M3 c3
```

Source: `references/diagramme/mmd/11-agent-klassen.mmd`, image: `references/diagramme/light/11-agent-klassen.png`

In our workflow: C1 is Opus in the main thread, C2 is Sonnet via the `Agent` tool, C3 is Haiku via the `Agent` tool.

The class is attached to the job, not the project:

```mermaid
flowchart LR
    J1("Job 1 — planning<br/>C1") --> J2("Job 2<br/>C2")
    J1 --> J3("Job 3<br/>C2")
    J1 --> J4("Job 4<br/>C2")
    J2 --> J5("Job 5 — synthesis<br/>C1")
    J3 --> J5
    J4 --> J5
    J5 --> J6("Job 6 — formatting, renaming, commit message<br/>C3")

    classDef c1 fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef c2 fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef c3 fill:#009E73,stroke:#00503a,color:#ffffff
    class J1,J5 c1
    class J2,J3,J4 c2
    class J6 c3
```

Source: `references/diagramme/mmd/12-klassen-zuweisung.mmd`, image: `references/diagramme/light/12-klassen-zuweisung.png`

Planning and synthesis are C1, because that's where conflicting information gets weighed. The parallel work steps are C2. Formatting, renaming, and commit messages are C3.

## 8. Summary

```mermaid
mindmap
  root)Harness Engineering(
    (Graph)
      Node = job
      Edge = flow
      Any node can itself be a graph
    (Scheduling)
      parallel where possible
      sync barrier where needed
      deterministic gates before expensive agents
    (Data contracts)
      fixed schema per edge
      quote + URL + ID against hallucination
      draw DFG and TSG separately
    (Control)
      review panel instead of a single opinion
      majority + severity instead of veto
      follow-up instead of reset
      hard attempt counter
    (Economics)
      agent classes C1 C2 C3
      class is attached to the job
      cheapest model that can still do the job
```

Source: `references/diagramme/mmd/13-zusammenfassung.mmd`, image: `references/diagramme/light/13-zusammenfassung.png`

## 9. What's deliberately different here from the template

The template describes a fully autonomous flow that commits straight from a GitHub issue, opens a pull request, and closes the issue. Our workflow adopts the structure, not the autonomy:

- Implementation only begins after explicit approval from the user.
- Security decisions, database migrations, architecture changes, and changes to existing customer data stay with C1 in the main thread and are never decided by majority vote.
- Commit, push, and pull request happen only on explicit instruction.
- A follow-up is an item in `TODO.md`, not an automatically created ticket.
- Discarding a work state after the attempt counter is hit gets presented to the user, not carried out.
