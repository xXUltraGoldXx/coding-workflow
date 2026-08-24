# coding-workflow

Ein Entwicklungs-Workflow als Skill für Claude Code. Er sorgt dafür, dass
zuerst geplant und dann gebaut wird, dass jede Behauptung über Code eine
Belegstelle hat, und dass teure Modelle nur dort arbeiten, wo sie gebraucht
werden.

> 🇬🇧 **English version:** switch to the [`englisch`](../../tree/englisch) branch.

## Wofür das gut ist

Ohne Führung neigt ein Coding-Assistent zu drei Dingen: Er fängt sofort an zu
schreiben, er behauptet Dinge über Code, ohne nachzusehen, und er schickt für
jede Kleinigkeit das teuerste Modell los. Dieser Skill stellt dem drei
Gewohnheiten entgegen.

**Planung vor Umsetzung.** Anforderungen klären, das Projekt ansehen, einen
Plan vorlegen — und erst nach ausdrücklicher Freigabe Code schreiben. Ein
„klingt gut" ist keine Freigabe.

**Belegen statt behaupten.** Jede Aussage über den Code nennt `datei:zeile`
und ein wörtliches Zitat. Eine erfundene Stelle fällt beim Nachschlagen
sofort auf; eine Behauptung ohne Belegstelle trägt keine Entscheidung.

**Das billigste Modell, das den Job schafft.** Planung und
Sicherheitsentscheidungen bleiben beim starken Modell. Routinearbeit,
Umbenennungen und Dokumentation gehen an günstigere — mit klarem Auftrag und
klarer Rückgabe.

## Installation

```bash
git clone -b deutsch https://github.com/xXUltraGoldXx/coding-workflow.git \
  ~/.claude/skills/coding-workflow
```

Der Ordnername bestimmt, wie der Skill heißt. Heißt er `coding-workflow`,
lädt ihn Claude über `/coding-workflow` oder auf Zuruf („Nutze bitte den
coding-workflow").

Danach einmalig in `SKILL.md` den eigenen Namen eintragen — sonst antwortet
Claude ohne Anrede:

```markdown
> **Nutzername:** `Dein Name`
```

Der Skill bleibt für die ganze Sitzung aktiv. Abschalten mit „Workflow aus"
oder „normaler Modus".

## Wie er arbeitet

### Drei Pflichtdateien

Sie liegen im Projekt, nicht im Skill, und überleben das Ende einer Sitzung.

| Datei | Inhalt |
|---|---|
| `Context.md` | Stand, Architektur, nächste Schritte — der Einstieg für die Fortsetzung |
| `TODO.md` | Arbeitspakete, Prüfschritte, offene Entscheidungen, Versuchszähler |
| `Analyse.md` | Jeder Befund mit Belegstelle, Nachweis und der Lehre daraus |

Bricht eine Sitzung ab, genügt „Mach weiter mit @Context.md" — der nächste
Durchgang liest die drei Dateien und arbeitet weiter, ohne die Analyse zu
wiederholen.

### Der Ablauf als Graph, nicht als Liste

Ein Job hat genau eine Verantwortung. Unabhängige Jobs laufen parallel, eine
Sync-Barriere steht nur dort, wo ein Job auf mehrere Vorgänger wartet. Ein
Knoten kann selbst wieder ein Graph sein: „Implementierung" ist von außen ein
Kasten und innen eine Kette.

![Der Harness als Graph](references/diagramme/light/03-autodev-harness-full.png)

### Deterministische Prüfungen vor teuren Agenten

Syntaxprüfung, Tests, Linter und Build kosten keine Tokens. Sie laufen
zuerst, und zwar gegen eine **Baseline**: Was war vorher schon rot? Ohne die
lässt sich nicht unterscheiden, ob ein roter Test neu kaputt ist oder es
immer schon war. Ein Review-Agent auf Code, der die Gates nicht besteht, ist
verbranntes Geld.

### Review-Panel statt Review-Schleife

Ein einzelner Reviewer, der nacheinander Korrektheit, Sicherheit und
Lesbarkeit bemängelt, erzeugt vier Runden. Drei Reviewer, die gleichzeitig je
eine Sichtweise prüfen, erzeugen eine.

![Sequenziell gegen parallel](references/diagramme/light/06-review-parallel.png)

Jeder Befund braucht `datei:zeile`, ein Zitat, eine Schwere von 0 bis 100 und
einen Korrekturvorschlag. Ein Controller entscheidet danach — nach festen
Regeln, nicht nach Gefühl. Sicherheitsbefunde ab Schwere 80,
Architekturänderungen und Datenbankmigrationen entscheidet er immer selbst,
nie über die Mehrheit der Reviewer.

### Agent-Klassen

![Agent-Klassen](references/diagramme/light/11-agent-klassen.png)

Die Klasse hängt am Job, nicht am Projekt. Scheitert ein Job zweimal an
derselben Stelle, übernimmt die nächsthöhere Klasse — kein dritter Versuch
mit demselben Modell. Nach drei gescheiterten Runden wird abgebrochen und die
Aufgabe neu zugeschnitten: Drei Fehlschläge sind ein Zeichen für einen
falschen Zuschnitt, nicht für ein zu schwaches Modell.

## Inhalt des Repositorys

```
SKILL.md                     der Skill selbst — verbindliche Regeln
references/harness.md        Harness-Modell, Review-Panel, Data Contracts
references/diagramme/light/  14 Diagramme als PNG
references/diagramme/mmd/    dieselben Diagramme als Mermaid-Quelle
```

## Anpassen

Der Skill ist eine Meinung, kein Naturgesetz. Wer anders arbeitet, ändert
`SKILL.md` — die Datei ist bewusst als lesbarer Text geschrieben und nicht
als Konfiguration. Naheliegende Stellschrauben:

- **Sprache der Antworten** (Abschnitt „Grundregeln")
- **Agent-Klassen und Modelle** (Abschnitt 5) — die Namen sind austauschbar
- **Wann der Controller selbst entscheidet** (Abschnitt 7.2)
- **Der Versuchszähler**, der eine Aufgabe nach drei Runden abbricht

## Lizenz

MIT — siehe [LICENSE](LICENSE).
