# Harness Engineering — Erläuterung zum Workflow

Dieses Dokument erklärt die Regeln aus `SKILL.md` und zeigt die zugehörigen Diagramme. `SKILL.md` ist verbindlich, dieses Dokument erklärt und begründet.

Grundgedanke: Nicht das Modell allein bestimmt das Ergebnis, sondern der Ablauf, in dem es arbeitet. Dasselbe Modell erreicht in einem gut gebauten Harness ein deutlich besseres Ergebnis als im Standardablauf.

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
    title "ARC-AGI-3 — der Harness entscheidet, nicht nur das Modell"
    x-axis ["Opus 5 | offiziell", "GPT-5.6 Sol | offiziell", "GPT-5.6 Sol | eigener Harness"]
    y-axis "Score in %" 0 --> 50
    bar [30.2, 7.8, 40]
```

Quelle: `references/diagramme/mmd/00-benchmark.mmd`, Bild: `references/diagramme/light/00-benchmark.png`

## 1. Graph-Grammatik

Ein Ablauf besteht aus Jobs als Knoten und Datenflüssen als Kanten. Verzweigungen sind ausdrücklich formulierte Bedingungen.

```mermaid
flowchart LR
    START([Start]) --> J("Job")
    J --> IF{"if"}
    IF -->|Fall A| A("Job A")
    IF -->|Fall B| B("Job B")
    IF -->|Fall C| C("Job C")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    class J,A,B,C agent
    class START term
    class IF dec
```

Quelle: `references/diagramme/mmd/01-graph-grammatik.mmd`, Bild: `references/diagramme/light/01-graph-grammatik.png`

Entscheidend ist die Rekursion: Jeder Knoten kann selbst ein Graph sein. Von außen ist „Implementierung“ ein Knoten, innen eine Kette oder ein Fächer aus mehreren Jobs mit anschließender Synthese.

```mermaid
flowchart LR
    subgraph JOB1["Job 1 — von außen ein einziger Knoten"]
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

Quelle: `references/diagramme/mmd/02-job-ist-selbst-graph.mmd`, Bild: `references/diagramme/light/02-job-ist-selbst-graph.png`

Praktische Folge für die Planung: Zuerst den Graphen auf oberster Ebene festlegen, dann nur die Knoten aufklappen, die tatsächlich mehrere Jobs brauchen. Ein Knoten, der sich nicht sinnvoll aufklappen lässt, ist ein einzelner Agentenauftrag.

## 2. Der vollständige Harness

Das folgende Diagramm zeigt den Referenzablauf. Er ist die Vorlage für Abschnitt 6 und 7 in `SKILL.md`.

```mermaid
flowchart TD
    START([neues GitHub-Issue]) --> BASE["Baseline-Tests<br/>welche Tests sind <b>vorher schon</b> rot?"]
    BASE --> READ("Issue-Reader<br/>gh issue view + Kommentar")
    READ --> RES("Recherche-Agent<br/>Codebase-Kontext sammeln")
    RES --> IMPL

    subgraph IMPL["Implementierung — von außen 1 Knoten, innen 4 Agents, strikt sequenziell"]
        direction LR
        TDD("1. TDD<br/>Tests zuerst") --> CODE("2. Implementieren")
        CODE --> COV("3. Coverage")
        COV --> DOCS("4. Doku")
    end

    IMPL --> QG["Quality Gates<br/>Tests + Linter + Build<br/>deterministisch, 0 Tokens"]
    QG --> QGD{"alles grün?<br/>gegen Baseline geprüft"}
    QGD -->|"nein — neu kaputt"| IMPL

    QGD -->|ja| RC("Review: Correctness")
    QGD -->|ja| RS("Review: Security")
    QGD -->|ja| RQ("Review: Code Quality")

    RC --> CTRL("Controller<br/>aggregiert alle Reviews")
    RS --> CTRL
    RQ --> CTRL

    CTRL --> VOTE{"mind. 2 von 3<br/>sagen OK?"}
    VOTE -->|ja| FIN
    VOTE -->|nein| SEV{"Severity<br/>über Schwelle?<br/>z.B. 80 %"}
    SEV -->|"nein — Kleinkram"| FIN
    SEV -->|ja| FU{"Follow-Up-Ticket<br/>sinnvoll möglich?"}
    FU -->|"ja — 90 % läuft"| FUA("Follow-Up-Agent<br/>Rest-Scope als neues Issue")
    FUA --> FIN
    FU -->|"nein — Code läuft gar nicht"| ATT{"Anzahl Versuche > N?"}
    ATT -->|nein| IMPL
    ATT -->|ja| SPLIT("Split-Agent<br/>Original-Ticket in 3-5 Sub-Tickets<br/>Original schließen, Code reverten")

    FIN("Finalize-Agent<br/>Commit, PR, Issue schließen") --> DONE([fertig])
    SPLIT --> RESTART([neue kleinere Issues<br/>Harness startet erneut])

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef det fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    class READ,RES,TDD,CODE,COV,DOCS,RC,RS,RQ,CTRL,FUA,SPLIT,FIN agent
    class BASE,QG det
    class QGD,VOTE,SEV,FU,ATT dec
    class START,DONE,RESTART term
```

Quelle: `references/diagramme/mmd/03-autodev-harness-full.mmd`, Bild: `references/diagramme/light/03-autodev-harness-full.png`

Kompaktere Fassung desselben Ablaufs:

```mermaid
flowchart LR
    START([Issue]) --> BASE["Baseline-<br/>Tests"] --> READ("Issue<br/>lesen") --> RES("Recherche")
    RES --> IMPL("Implementierung<br/><small>TDD -> Code -> Coverage -> Doku</small>")
    IMPL --> QG{"Quality Gates<br/>Tests + Linter<br/>0 Tokens"}
    QG -->|"rot"| IMPL

    QG -->|"grün"| RC("Correctness")
    QG -->|"grün"| RS("Security")
    QG -->|"grün"| RQ("Quality")
    RC --> CTRL("Controller")
    RS --> CTRL
    RQ --> CTRL

    CTRL --> D{"2/3 OK?<br/>sonst: Severity?<br/>sonst: Follow-Up?<br/>sonst: Versuche > N?"}
    D -->|"OK / Kleinkram"| FIN("Finalize")
    D -->|"Follow-Up"| FIN
    D -->|"nachbessern"| IMPL
    D -->|"Versuche > N"| SPLIT("Ticket<br/>aufteilen")
    FIN --> DONE([fertig])
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

Quelle: `references/diagramme/mmd/04-autodev-harness-16zu9.mmd`, Bild: `references/diagramme/light/04-autodev-harness-16zu9.png`

Die drei tragenden Ideen:

1. Baseline zuerst. Ohne den Vorzustand ist „Test rot“ keine Information.
2. Deterministische Gates vor teuren Agenten. Tests, Linter und Build kosten keine Tokens. Ein Review-Agent auf rotem Code kostet Tokens und liefert Befunde, die der Compiler schon kannte.
3. Kein unbegrenztes Nachbessern. Entweder Mehrheit, oder Kleinkram akzeptieren, oder Rest als Follow-Up abtrennen, oder nach dem Versuchszähler die Aufgabe neu zuschneiden.

## 3. Review sequenziell gegen Review parallel

Sequenziell erzeugt jeder Reviewer eine eigene Implementierungsrunde. Vier Runden für drei Sichtweisen:

```mermaid
flowchart TB
    I1("Impl v1") --> C1("Correctness")
    C1 -->|"'so nicht'"| I2("Impl v2")
    I2 --> C2("Correctness ok") --> S2("Security")
    S2 -->|"'so nicht'"| I3("Impl v3")
    I3 --> S3("Security ok") --> Q3("Quality")
    Q3 -->|"'kein Clean Code'"| I4("Impl v4")

    classDef bad fill:#D55E00,stroke:#7a3600,color:#ffffff
    class I1,I2,I3,I4,C1,C2,S2,S3,Q3 bad
```

Quelle: `references/diagramme/mmd/05-review-sequenziell.mmd`, Bild: `references/diagramme/light/05-review-sequenziell.png`

Parallel mit gebündeltem Feedback: eine Runde.

```mermaid
flowchart TB
    PI("Impl v1") --> PC("Correctness")
    PI --> PS("Security")
    PI --> PQ("Quality")
    PC --> PCT("Controller<br/>alle Feedbacks gebündelt")
    PS --> PCT
    PQ --> PCT
    PCT -->|"1x zurück, mit allem auf einmal"| PI2("Impl v2")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    class PI,PC,PS,PQ,PCT,PI2 agent
```

Quelle: `references/diagramme/mmd/06-review-parallel.mmd`, Bild: `references/diagramme/light/06-review-parallel.png`

Deshalb laufen die drei Reviewer in `SKILL.md` Abschnitt 7.1 gleichzeitig auf demselben Diff, und der Controller bündelt.

## 4. Controller

Der Controller entscheidet, er reviewt nicht selbst. Er bleibt im Hauptthread bei C1, weil hier die teuren Fehlentscheidungen entstehen.

```mermaid
flowchart TD
    RC("Correctness") --> CTRL
    RS("Security") --> CTRL
    RQ("Code Quality") --> CTRL
    CTRL("<b>Controller</b><br/>Reviews aggregieren, gewichten, entscheiden")

    CTRL --> V{"Mehrheit OK?<br/>2 von 3"}
    V -->|ja| FIN(["Finalize"])
    V -->|nein| SEV{"Severity > 80 %?"}
    SEV -->|nein| FIN
    SEV -->|ja| FU{"Follow-Up<br/>sinnvoll?"}
    FU -->|ja| FUT("Follow-Up-Ticket anlegen") --> FIN
    FU -->|nein| N{"Versuche > N?"}
    N -->|nein| IMPL(["zurück zur Implementierung"])
    N -->|ja| SPL("Ticket aufteilen<br/>N Sub-Tickets, Original zu, Code revert")

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef dec fill:#E69F00,stroke:#7a5300,color:#111111
    classDef term fill:#3f3f46,stroke:#18181b,color:#ffffff
    class RC,RS,RQ,CTRL,FUT,SPL agent
    class V,SEV,FU,N dec
    class FIN,IMPL term
```

Quelle: `references/diagramme/mmd/10-controller-detail.mmd`, Bild: `references/diagramme/light/10-controller-detail.png`

Die Reihenfolge der Prüfungen ist bewusst so gewählt:

- Mehrheit vor Einzelmeinung: Ein einzelner Reviewer mit strengem Maßstab darf keine Endlosschleife auslösen.
- Severity vor Vollständigkeit: Ein Formatierungshinweis blockiert keinen fertigen Fix.
- Follow-Up vor Neuanfang: Wenn neunzig Prozent laufen, wird der Rest ein eigener Punkt und nicht die Begründung, alles zu verwerfen.
- Versuchszähler vor Beharrlichkeit: Drei gescheiterte Runden sind ein Zuschnittproblem. Die Antwort ist ein kleinerer Aufgabenschnitt, kein vierter Versuch.

Abweichung von der Vorlage: Das Zerlegen und das Verwerfen des bisherigen Stands legt der Controller dem Nutzer vor. Er tut es nicht selbstständig.

## 5. Data Contracts

Jede Kante hat ein Schema. Der Empfänger weiß dadurch, was er bekommt, und der Sender weiß, was er liefern muss.

```mermaid
flowchart LR
    J1("Job 1<br/>Recherche") -->|"DO1"| J2("Job 2<br/>Verdichten")
    DO1[("DO1 — Quellenliste")]
    J1 -.->|"produziert"| DO1
    DO1 -.->|"konsumiert"| J2

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef data fill:#009E73,stroke:#00503a,color:#ffffff
    class J1,J2 agent
    class DO1 data
```

Quelle: `references/diagramme/mmd/07-data-contract-kante.mmd`, Bild: `references/diagramme/light/07-data-contract-kante.png`

Ein Schema besteht aus Objekten mit Pflichtfeldern. Entscheidend gegen Halluzination sind Identität und Belegstelle: eine `id`, ein wörtliches Zitat und ein Verweis vom abgeleiteten Fakt zurück auf seine Quelle.

```mermaid
classDiagram
    class DO1_Quellenliste {
        +Quelle[] quellen
        +string thema
    }
    class Quelle {
        +string id
        +string url
        +string zitat  «wörtlich von der Seite»
        +string relevanz
    }
    class DO2_Factsheet {
        +Fakt[] fakten
    }
    class Fakt {
        +string aussage
        +string quelle_id  «FK auf Quelle.id»
        +string zitat
    }
    DO1_Quellenliste "1" o-- "n" Quelle
    DO2_Factsheet "1" o-- "n" Fakt
    Quelle <.. Fakt : belegt durch
```

Quelle: `references/diagramme/mmd/08-data-contract-schema.mmd`, Bild: `references/diagramme/light/08-data-contract-schema.png`

Auf unseren Workflow übertragen:

- `Analyse.md` entspricht der Quellenliste. Jeder Befund hat `id`, `datei:zeile` und ein wörtliches Zitat.
- `Context.md` und Agentenberichte entsprechen dem Factsheet. Jede Aussage über den Code trägt eine `id` oder eine eigene Belegstelle.
- Review-Befunde tragen zusätzlich Schwere und Korrekturvorschlag.

Eine Aussage ohne Belegstelle ist keine Grundlage für eine Entscheidung. Das ist keine Formalität: Erfundene Fundstellen fallen beim Nachschlagen sofort auf, erfundene Prosa nicht.

## 6. Datenfluss und Ablauf getrennt planen

Zwei Fragen, zwei Graphen. Was fließt, und wer läuft wann. Vermischt man beide, entstehen Abhängigkeiten, die niemand kontrolliert.

```mermaid
flowchart TB
    subgraph DFG["DFG — Data Flow Graph: WAS fließt"]
        direction LR
        DO1[("DO1<br/>Quellenliste")] --> M(( ))
        DO2[("DO2<br/>Repo-/Domänen-Kontext")] --> M
        M --> DO3[("DO3<br/>Briefing")]
        M --> DO4[("DO4<br/>erweiterte Quellen")]
    end

    subgraph TSG["TSG — Task Schedule Graph: WER läuft WANN"]
        direction LR
        T1("T1 — Recherche") --> SYNC{{"Sync-Barriere<br/>wartet auf T1 UND T3"}}
        T3("T3 — Kontext laden") --> SYNC
        SYNC --> T2("T2 — Verdichten")
        T2 --> T4("T4 — Briefing schreiben")
        T2 --> T5("T5 — Quellen erweitern")
    end

    DO1 -. "erzeugt von" .- T1
    DO2 -. "erzeugt von" .- T3
    DO1 -. "Input für" .- T2
    DO2 -. "Input für" .- T2
    DO3 -. "erzeugt von" .- T4
    DO4 -. "erzeugt von" .- T5

    classDef agent fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef data fill:#009E73,stroke:#00503a,color:#ffffff
    classDef sync fill:#E69F00,stroke:#7a5300,color:#111111
    class T1,T2,T3,T4,T5 agent
    class DO1,DO2,DO3,DO4 data
    class SYNC sync
    style M fill:#009E73,stroke:#00503a,color:#ffffff
```

Quelle: `references/diagramme/mmd/09-dfg-ueber-tsg.mmd`, Bild: `references/diagramme/light/09-dfg-ueber-tsg.png`

Die Sync-Barriere ist der Punkt, an dem ein Job auf mehrere Vorgänger wartet. Sie muss ausdrücklich benannt werden, sonst läuft ein Agent mit halben Daten los.

## 7. Agent-Klassen

Klassen statt Modellnamen, damit die Zuordnung stabil bleibt, wenn sich die Modelle ändern.

```mermaid
flowchart LR
    subgraph K1["Class 1 — teuer, kompetent, wenig Fehler"]
        M1["Fable<br/>GPT-5.6 Sol"]
    end
    subgraph K2["Class 2 — Arbeitspferde"]
        M2["Opus 5<br/>Kimi K3<br/>Qwen 3.8"]
    end
    subgraph K3["Class 3 — billig, für Kleinkram"]
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

Quelle: `references/diagramme/mmd/11-agent-klassen.mmd`, Bild: `references/diagramme/light/11-agent-klassen.png`

In unserem Workflow: C1 ist Opus im Hauptthread, C2 ist Sonnet über das `Agent`-Tool, C3 ist Haiku über das `Agent`-Tool.

Die Klasse hängt am Job, nicht am Projekt:

```mermaid
flowchart LR
    J1("Job 1 — Planung<br/>C1") --> J2("Job 2<br/>C2")
    J1 --> J3("Job 3<br/>C2")
    J1 --> J4("Job 4<br/>C2")
    J2 --> J5("Job 5 — Synthese<br/>C1")
    J3 --> J5
    J4 --> J5
    J5 --> J6("Job 6 — Formatieren, Renaming, Commit-Message<br/>C3")

    classDef c1 fill:#D55E00,stroke:#7a3600,color:#ffffff
    classDef c2 fill:#0072B2,stroke:#023d5e,color:#ffffff
    classDef c3 fill:#009E73,stroke:#00503a,color:#ffffff
    class J1,J5 c1
    class J2,J3,J4 c2
    class J6 c3
```

Quelle: `references/diagramme/mmd/12-klassen-zuweisung.mmd`, Bild: `references/diagramme/light/12-klassen-zuweisung.png`

Planung und Synthese sind C1, weil dort widersprüchliche Informationen bewertet werden. Die parallelen Arbeitsschritte sind C2. Formatieren, Renaming und Commit-Nachricht sind C3.

## 8. Zusammenfassung

```mermaid
mindmap
  root)Harness Engineering(
    (Graph)
      Knoten = Job
      Kante = Flow
      Jeder Knoten kann selbst ein Graph sein
    (Scheduling)
      parallel wo möglich
      Sync-Barriere wo nötig
      deterministische Gates vor teure Agents
    (Data Contracts)
      festes Schema pro Kante
      Zitat + URL + ID gegen Halluzination
      DFG und TSG getrennt zeichnen
    (Kontrolle)
      Review-Panel statt Einzelmeinung
      Mehrheit + Severity statt Veto
      Follow-Up statt Reset
      harter Versuchszähler
    (Ökonomie)
      Agent-Klassen C1 C2 C3
      Klasse hängt am Job
      billigstes Modell das den Job noch schafft
```

Quelle: `references/diagramme/mmd/13-zusammenfassung.mmd`, Bild: `references/diagramme/light/13-zusammenfassung.png`

## 9. Was hier bewusst anders ist als in der Vorlage

Die Vorlage beschreibt einen vollautonomen Ablauf, der aus einem GitHub-Issue heraus committet, einen Pull Request eröffnet und das Issue schließt. Unser Workflow übernimmt die Struktur, nicht die Autonomie:

- Die Umsetzung beginnt erst nach der ausdrücklichen Freigabe durch den Nutzer.
- Sicherheitsentscheidungen, Datenbankmigrationen, Architekturänderungen und Änderungen an bestehenden Kundendaten bleiben bei C1 im Hauptthread und werden nicht per Mehrheit entschieden.
- Commit, Push und Pull Request geschehen nur auf ausdrückliche Anweisung.
- Ein Follow-Up ist ein Punkt in `TODO.md`, kein automatisch angelegtes Ticket.
- Das Verwerfen eines Arbeitsstands nach dem Versuchszähler wird vorgelegt, nicht ausgeführt.
