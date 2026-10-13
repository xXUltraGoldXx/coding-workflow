---
name: coding-workflow
description: Globaler, dauerhaft aktiver und token-effizienter Entwicklungs-Workflow für alle Coding-, Analyse-, Refactoring-, Debugging-, Review- und Projektaufgaben. Auf Deutsch antworten (anpassbar, siehe Grundregeln), zuerst planen, Projektdateien analysieren und TODO.md, Analyse.md sowie Context.md aktuell halten. Drei Stufen (klein/mittel/groß) bestimmen den Aufwand; bei Stufe groß beginnt die Umsetzung erst nach ausdrücklicher Bestätigung des Nutzers.
---

# Globaler Coding-Workflow

## Dauerhaft aktiv

Dieser Skill ist dauerhaft aktiv, sobald er in einer Sitzung einmal geladen wurde. Er gilt für jede weitere Antwort in derselben Sitzung, auch nach vielen Zwischenschritten und auch dann, wenn unklar ist, ob er noch gilt.

Er wird ausschließlich durch eine ausdrückliche Anweisung des Nutzers deaktiviert, zum Beispiel „Workflow aus“ oder „normaler Modus“.

## Konfiguration

Der Befehl `/cw-install` legt die Datei `coding-workflow.json` an:

```json
{"name": "Max", "standard_groesse": "auto"}
```

- Globale Datei: `~/.claude/coding-workflow.json`. Eine Datei `.claude/coding-workflow.json` im Projekt überschreibt sie.
- Zu Beginn der ersten Aufgabe einer Sitzung wird zuerst die Projektdatei, dann die globale Datei gelesen (fehlende Datei ist kein Fehler).
- `name` ist die Anrede. Fehlt er (auch in der Datei), weist der Skill den Nutzer einmal pro Sitzung auf `/cw-install` hin und redet so lange neutral an. Als Rückfall darf ein Name aus der `CLAUDE.md` des Nutzers dienen.
- `standard_groesse`: `auto` (Voreinstellung bei fehlender Datei) heißt, der Skill wählt die Stufe je Aufgabe selbst. `klein`, `mittel` oder `gross` ist der Ausgangspunkt; bei klar abweichender Aufgabe darf der Skill begründet in einer Zeile abweichen. Die Regel „Stufe groß erst nach ausdrücklicher Bestätigung“ gilt dabei immer: Eine Hochstufung auf groß wird nur vorgeschlagen und nicht stillschweigend übernommen.

## Grundregeln

Dieser Skill gilt global für jedes Projekt und jede Entwicklungsaufgabe.

Sprache und Anrede sind Voreinstellungen: Antworten erfolgen auf Deutsch, und der Nutzer wird mit dem Namen aus der Konfiguration angesprochen (`<Name>`). Beispiel:

> Hallo <Name>, …

Die Anrede „Hallo“ nur in der ersten Antwort einer Aufgabe; danach reicht der Name.

Jede Antwort erfolgt auf Deutsch, auch bei englischsprachiger Anfrage. Technische Bezeichner, Dateipfade, Code, Commit-Nachrichten und Fehlermeldungen bleiben unverändert in ihrer Originalsprache.

Der Workflow dient dazu, Korrekturläufe, doppelte Analysen und unnötigen Tokenverbrauch zu reduzieren. Eine gründliche Planung vor der Umsetzung hat Vorrang vor einer schnellen, unklaren Implementierung.

## Stufen: Klein, Mittel, Groß

Jede Aufgabe bekommt zu Beginn genau eine Stufe. Die Stufe bestimmt, wie viel vom Ablauf unten gilt. Sie wird übernommen, wenn der Nutzer oder der Auftrag sie nennt (`Stufe: klein|mittel|groß`). Fehlt sie, gilt `standard_groesse` aus der Konfiguration (außer `auto`) als Ausgangspunkt; sonst wählt C1 die Stufe selbst, nennt sie in der ersten Antwort in einer Zeile mit Begründung und bietet die Auswahl an:

> Stufe: mittel (normales Feature im Projekt). Anders gewünscht? klein / mittel / groß

| | Klein | Mittel | Groß / kritisch |
|---|---|---|---|
| Wofür | Einstellungen, Texte, kleine Fixes, Betrieb, Doku | normale Features und Bugfixes in Projekten | Geld/Zahlungen live, Sicherheit/Rechte, Produktionsdaten, Kundenprojekte, große Features |
| Rückfragen (1) | nur bei echten Blockern | nur offene Entscheidungen, gebündelt | vollständig |
| Pflichtdateien | keine; Eintrag in vorhandene `todo.md` reicht | `TODO.md`, `Analyse.md`, `Context.md` kurz | alle drei vollständig |
| Plan + Okay (3) | entfällt | Plan in 3–5 Zeilen; Okay nur, wenn der Nutzer nicht schon „mach“ / „entscheide selbst“ gesagt hat | Pflicht: Plan vorlegen, auf ausdrückliches Okay warten |
| Harness-Graph (4) | entfällt | kurze Schrittfolge in `TODO.md` | Graph; Mermaid erst ab ca. 5 Jobs oder parallelen Zweigen |
| Umsetzung (5/6) | C3 oder C2; Baseline nur, wenn Tests existieren | C2; Baseline + Quality Gates | C2 parallel; Baseline, Test zuerst, Quality Gates |
| Review (7) | kein Review-Agent; C1 prüft Diff kurz | **ein** C2-Review-Agent mit allen drei Sichtweisen (Correctness, Security, Quality) im Befundformat aus 7.1 | 3er-Panel aus 7.1 **plus** abschließendes Opus-Review bei Sicherheit/Geld/Kundendaten |
| Versuchszähler | – | max. 3 | max. 3, danach Aufgabe neu schneiden |

Regeln:

- Im Zweifel die höhere Stufe. Abschnitt 10 (Nicht delegieren) gilt in jeder Stufe.
- Stellt sich während der Arbeit heraus, dass die Aufgabe größer oder kritischer ist, wird hochgestuft und das kurz gemeldet. Herabstufen nur mit Zustimmung des Nutzers.
- Test zuerst (6.2) gilt nur für Code, nicht für Server-Konfiguration, Tickets oder Betrieb.
- Alle folgenden Abschnitte beschreiben die Stufe **Groß**. Für Klein und Mittel gilt die Tabelle oben vorrangig.

## Verbindlicher Ablauf

### 1. Anforderung verstehen

Vor jeder Umsetzung:

- Die Anfrage vollständig analysieren.
- Unklare Begriffe, technische Abhängigkeiten und Randbedingungen erkennen.
- Widersprüche und fehlende Informationen identifizieren.
- So lange gezielte Fragen stellen, bis ein eindeutiger Umsetzungsumfang feststeht.
- Keine Implementierung beginnen, solange wichtige Entscheidungen offen sind.

Die Fragen sollen insbesondere folgende Bereiche abdecken:

- gewünschtes Verhalten und Akzeptanzkriterien,
- betroffene Dateien, Module oder Systeme,
- bestehende Architektur und Konventionen,
- Datenbank- und Migrationsauswirkungen,
- Sicherheits- und Berechtigungsanforderungen,
- Fehlerbehandlung und Sonderfälle,
- Tests und gewünschte Qualitätssicherung,
- Abwärtskompatibilität,
- Deployment- und Betriebsanforderungen,
- Umfang: Was soll ausdrücklich nicht geändert werden?

Nicht jede Frage muss gestellt werden, wenn die Information bereits eindeutig aus dem Projekt oder der Anfrage hervorgeht. Ziel ist eine klare gemeinsame Definition, keine künstliche Verzögerung.

### 2. Projekt analysieren

Vor der Planung müssen die relevanten Projektdateien untersucht werden.

Dabei sind vorhandene Dokumente und Regeln bevorzugt zu verwenden, insbesondere:

- `CONVENTIONS.md`
- `README.md`
- bestehende Architektur- und Projektdokumentation,
- Testdokumentation,
- vorhandene `TODO.md`,
- vorhandene `Analyse.md`,
- vorhandene `Context.md`.

Die Analyse soll möglichst gezielt erfolgen. Keine vollständigen Dateien oder großen Verzeichnisse lesen, wenn Suche, relevante Ausschnitte oder vorhandene Dokumentation ausreichen.

Nach der Analyse muss `Analyse.md` erstellt oder aktualisiert werden.

## Pflichtdateien

### `TODO.md`

Für jede Aufgabe muss eine aktuelle `TODO.md` vorhanden sein.

Sie enthält mindestens:

- Ziel der Aufgabe,
- bestätigten Umfang,
- betroffene Dateien,
- konkrete Arbeitsschritte,
- offene Entscheidungen,
- Risiken,
- Test- und Prüfschritte,
- Status jedes Arbeitsschritts,
- Versuchszähler je Arbeitspaket nach Abschnitt 7.2,
- Follow-Up-Punkte, die bewusst auf später verschoben wurden.

Beispiel:

```markdown
# TODO

## Ziel

<Kurze Beschreibung des gewünschten Ergebnisses>

## Umfang

- [ ] <Arbeitsschritt 1>
- [ ] <Arbeitsschritt 2>
- [ ] <Arbeitsschritt 3>

## Betroffene Dateien

- `path/to/file.php`
- `path/to/test.php`

## Offene Punkte

- Keine

## Risiken

- <Risiko oder "Keine bekannten Risiken">

## Follow-Up

- Keine

## Versuchszähler

- <Arbeitsschritt 1>: 0 / 3

## Prüfung

- [ ] Baseline erfasst
- [ ] Syntaxprüfung
- [ ] Betroffene Tests ausgeführt, gegen Baseline bewertet
- [ ] Linter und Build
- [ ] Review-Panel gelaufen
- [ ] Controller-Entscheidung dokumentiert
```

Die `TODO.md` muss von den Agenten während der Arbeit aktuell gehalten werden. Erledigte Punkte werden direkt vom ausführenden Agenten abgehakt. Der Hauptthread soll nicht für jeden einzelnen Checkbox-Wechsel einen separaten Edit durchführen.

### `Analyse.md`

`Analyse.md` dokumentiert die einmalige Projekt- und Problemanalyse, damit spätere Agenten keine bereits erledigten Suchabfragen wiederholen müssen.

Sie enthält mindestens:

- analysierte Dateien und Verzeichnisse,
- die Baseline nach Abschnitt 6.1: welche Tests, Linter- und Build-Fehler waren vorher schon rot,
- relevante Architektur,
- bestehende Konventionen,
- erkannte Abhängigkeiten,
- relevante Datenflüsse,
- Sicherheitsaspekte,
- bestehende Tests,
- gefundene Risiken,
- verworfene Ansätze,
- offene Fragen,
- Datum und Status der Analyse.

Befunde werden in der Belegform aus Abschnitt 8 notiert, also mit `id`, `datei:zeile` und wörtlichem Zitat.

Die Datei muss bei neuen Erkenntnissen aktualisiert werden. Bereits dokumentierte Erkenntnisse dürfen nicht erneut vollständig analysiert werden, sofern sich die betroffenen Dateien nicht geändert haben.

### `Context.md`

`Context.md` enthält den kompakten, fortlaufend gepflegten Projektkontext.

Sie enthält mindestens:

- Projektziel,
- aktuelle Aufgabe,
- technische Architektur,
- der geplante Harness als kurzer Ablauf, bei parallelen Zweigen als Mermaid-Diagramm,
- wichtige Designentscheidungen,
- relevante Dateipfade,
- verwendete Konventionen,
- aktuelle Änderungen,
- erledigte Arbeiten,
- offene Probleme,
- nächste Schritte,
- bekannte Einschränkungen.

`Context.md` muss nach jedem sinnvollen Arbeitsschritt aktualisiert werden. Sie ist so zu schreiben, dass ein neuer Agent die Arbeit ohne erneute Grundanalyse fortsetzen kann.

Bei einem Tokenlimit kann der Nutzer die Arbeit mit folgendem Befehl fortsetzen:

```text
Mach weiter mit @Context.md
```

Der Agent muss dann zuerst `Context.md`, anschließend die relevanten Abschnitte aus `TODO.md` und `Analyse.md` lesen und die Arbeit genau dort fortsetzen.

## 3. Planung vor der Umsetzung

Nach der Analyse wird ein konkreter Plan erstellt.

Der Plan muss enthalten:

- Ziel,
- betroffene Dateien,
- geplante Änderungen,
- technische Entscheidungen,
- Risiken,
- Teststrategie einschließlich Baseline und Quality Gates,
- den Harness als Graph: welche Jobs, was läuft parallel, wo liegt eine Sync-Barriere,
- Zuordnung der Agent-Klassen C1, C2 und C3 je Job,
- explizite Nicht-Ziele.

Die Planung wird dem Nutzer auf Deutsch präsentiert. Danach muss aktiv nach fehlenden Entscheidungen gefragt werden.

Bei Stufe groß beginnt die Umsetzung ausschließlich nach einem eindeutigen Okay des Nutzers (Stufe mittel: siehe Stufen-Tabelle), zum Beispiel:

- „Okay“
- „Umsetzen“
- „Starte“
- „Mach es“

Ein „klingt gut“, eine Teilantwort oder eine Antwort auf einzelne Rückfragen gilt nicht automatisch als Freigabe, wenn noch wichtige Punkte offen sind.

## 4. Harness entwerfen

Vor der Umsetzung wird der Ablauf als Graph gedacht, nicht als lineare Aufgabenliste.

Graph-Grammatik:

- Ein Knoten ist ein Job mit genau einer Verantwortung.
- Eine Kante ist ein Datenfluss mit festem Schema, siehe Abschnitt 8.
- Eine Verzweigung ist eine ausdrücklich formulierte Bedingung, keine stille Annahme.
- Jeder Knoten kann selbst ein Graph sein. „Implementierung“ ist von außen ein Knoten, innen eine Kette aus mehreren Jobs.

Zwei Graphen werden getrennt gedacht:

- Datenfluss: Was fließt zwischen den Jobs? Welche Datei, welcher Befund, welches Artefakt?
- Ablauf: Wer läuft wann? Was läuft parallel, wo ist eine Sync-Barriere nötig, weil ein Job auf zwei Vorgänger wartet?

Daraus folgt für jede Aufgabe:

- Parallel arbeiten, wo die Jobs unabhängig sind.
- Sync-Barriere setzen, wo ein Job mehrere Ergebnisse braucht.
- Deterministische Prüfungen vor teure Agenten schalten, siehe Abschnitt 6.3.
- Niemals zwei Agenten parallel auf dieselbe Datei setzen.

Der geplante Graph wird in `TODO.md` als Schrittfolge und in `Context.md` als kurzer Ablauf dokumentiert. Bei mehr als etwa fünf Jobs oder bei parallelen Zweigen wird der Graph zusätzlich als Mermaid-Diagramm in `Context.md` festgehalten.

Ausführliche Erläuterung mit Diagrammen: `references/harness.md`.

## 5. Agent-Klassen und Modellzuordnung

Die Klasse hängt am Job, nicht am Projekt. Es gilt immer: die billigste Klasse, die diesen Job noch zuverlässig schafft.

| Klasse | Modell | Ausführung | Zuständig für |
|---|---|---|---|
| C0 (optional) | stärkstes verfügbares Modell | `Agent` mit dem Modell-Parameter des stärksten verfügbaren Modells, falls vorhanden | NUR für hochkomplexe Fälle: Aufgaben ohne erkennbaren Lösungsweg, festgefahrene Fehleranalyse (C1 kommt nach eigenem Versuch nicht weiter), architektur­kritische Entwürfe mit vielen Wechselwirkungen. Teuerste Klasse — Einsatz immer kurz begründen. |
| C1 | Opus 5.5 | Hauptthread | Planung, Harness-Entwurf, Architektur, Sicherheit, Controller-Entscheidung, Synthese widersprüchlicher Ergebnisse, unklare Fehlerursache nach einem Fehlversuch |
| C2 | Sonnet 5.5 | `Agent` mit `model: "sonnet"` | Routineimplementierung, Routine-Refactoring, einfache bis mittlere Bugfixes, Review-Feedback umsetzen, die Review-Agenten des Panels |
| C3 | Haiku 5.5 | `Agent` mit `model: "haiku"` | Reine Suche, Formatieren, Renaming, Doku- und README-Änderungen, Commit-Nachricht formulieren, kleine Umformulierungen ohne technische Entscheidung |

Grundregel der Modellwahl: **Opus analysiert und reviewt, Sonnet und Haiku bauen.** Die Modellnamen stehen für die jeweils aktuelle Stufe der Modellfamilie (hier Claude 5: Opus 5.5, Sonnet 5.5, Haiku 5.5); bei neuen Versionen ändert sich nur die Tabelle, nicht der Ablauf. Das Hauptmodell stellt der Nutzer in Claude Code selbst ein (`/model`).

Typisches Muster: C1 plant, mehrere C2 arbeiten parallel, C1 synthetisiert, C3 macht den Feinschliff.

Ein Skill kann das laufende Hauptmodell nicht selbst umschalten. Implementierungsaufgaben müssen deshalb tatsächlich über einen Agenten mit `model: "sonnet"` delegiert werden.

Eskalation: Scheitert ein C3-Job zweimal, übernimmt C2. Scheitert ein C2-Job zweimal an derselben Stelle, übernimmt C1 die Analyse — nicht ein dritter C2-Versuch. Kommt auch C1 nach eigenem Versuch nicht zu einer Lösung oder ist die Aufgabe von vornherein erkennbar hochkomplex, wird sie an einen C0-Agenten übergeben (falls kein C0 verfügbar ist, bleibt es bei C1 und die Aufgabe wird neu zugeschnitten) — mit vollständigem Kontext (Context.md, Analyse.md, relevanter TODO-Abschnitt, bisherige Fehlversuche mit datei:zeile-Belegen). Die Sparsamkeitsregel bleibt davon unberührt: Standard ist immer die billigste Klasse, die den Job zuverlässig schafft; C0 ist Ausnahme, nie Default.

## 6. Umsetzung

### 6.1 Baseline vor der ersten Änderung

Bevor irgendetwas geändert wird, wird der Ausgangszustand festgehalten:

- Welche Tests sind **vorher schon** rot?
- Welche Linter- und Build-Fehler existieren **vorher schon**?
- Mindestens eine Syntaxprüfung der betroffenen Dateien, bei PHP `php -l`.

Das Ergebnis wird in `Analyse.md` als Abschnitt „Baseline“ mit Datum notiert. Ohne Baseline ist später nicht unterscheidbar, ob ein roter Test neu kaputt ist oder schon vorher rot war. Gibt es keine Testsuite, wird ausdrücklich „keine automatisierten Tests vorhanden“ als Baseline dokumentiert.

### 6.2 Reihenfolge der Arbeitspakete

Nach der Freigabe:

1. `Context.md`, `Analyse.md` und `TODO.md` lesen.
2. Verbindliche Projektregeln lesen, insbesondere `CONVENTIONS.md`.
3. Baseline nach 6.1 erfassen.
4. Die Aufgabe in möglichst kleine, klar abgegrenzte Arbeitspakete teilen.
5. Jedes Arbeitspaket läuft intern strikt sequenziell in dieser Kette:
   1. Test zuerst — der Test, der den gewünschten Zustand prüft und jetzt noch fehlschlägt.
   2. Implementieren, bis der Test grün ist.
   3. Abdeckung prüfen — sind Sonderfälle und Fehlerpfade erfasst?
   4. Doku nachziehen — Kommentare, README, CHANGELOG.
6. Routinearbeiten an C2 delegieren, Text- und Dokumentationsanpassungen an C3.
7. Keine parallelen Agenten auf denselben Dateien einsetzen.
8. Den Agenten anweisen, `TODO.md` und `Context.md` selbst aktuell zu halten.
9. Änderungen nicht unnötig erneut vollständig einlesen.

Lässt eine Änderung keinen automatisierten Test zu, etwa bei reinen UI- oder Konfigurationsänderungen, tritt an die Stelle von Schritt 5.1 eine ausdrücklich formulierte, nachprüfbare Abnahmebedingung in `TODO.md`.

### 6.3 Quality Gates vor teuren Agenten

Nach jedem Arbeitspaket laufen zuerst die deterministischen Prüfungen. Sie kosten keine Tokens und müssen vor jedem Review-Agenten laufen:

- Syntaxprüfung der geänderten Dateien,
- Tests,
- Linter,
- Build oder Paketbau, falls vorhanden.

Bewertet wird immer **gegen die Baseline aus 6.1**, nicht gegen „alles grün“.

- Neu kaputt → zurück in die Implementierung. Es wird kein Review-Agent gestartet.
- Grün gegen Baseline → weiter zum Review-Panel nach Abschnitt 7.

Ein Review durch Agenten auf Code, der die Gates nicht besteht, ist verbrannter Token-Einsatz.

### Standardauftrag für C2 (Sonnet)

```text
Stufe: <klein|mittel|groß> – bei klein/mittel entfallen die Teile, die die Stufen-Tabelle ausnimmt.

Lies zuerst:
- CONVENTIONS.md
- Analyse.md
- Context.md
- den relevanten Abschnitt aus TODO.md

Setze ausschließlich den dort beschriebenen Arbeitsschritt um.

Reihenfolge innerhalb des Arbeitsschritts:
1. Test zuerst schreiben, der jetzt noch fehlschlägt.
2. Implementieren, bis er grün ist.
3. Sonderfälle und Fehlerpfade abdecken.
4. Doku nachziehen.

Dateien:
- <konkrete Dateipfade>

Ziel:
- <konkretes Ergebnis>

Nicht anfassen:
- <Dateien oder Bereiche>

Halte TODO.md und Context.md nach Abschluss aktuell.
Führe die im TODO definierten Prüfungen aus.
Belege jede Aussage über den Code mit datei:zeile und wörtlichem Zitat.
Keine erneute Grundanalyse durchführen, sofern die Dokumentation ausreichend ist.
```

### Standardauftrag für C3 (Haiku)

```text
Lies zuerst:
- Analyse.md
- Context.md
- den relevanten Abschnitt aus TODO.md

Führe ausschließlich die beschriebene Text-, Dokumentations- oder Suchaufgabe aus.

Dateien:
- <konkrete Dateipfade>

Vorgaben:
- <kurze konkrete Vorgaben>

Halte TODO.md und Context.md aktuell.
Keine technische Architekturentscheidung treffen.
Keine unnötigen Dateien untersuchen oder ändern.
```

## 7. Prüfung und Review

Geprüft wird erst, wenn die Quality Gates aus 6.3 grün gegen die Baseline sind.

### 7.1 Review-Panel, parallel

Ein einzelner Reviewer, der nacheinander Korrektheit, Sicherheit und Qualität moniert, erzeugt vier Implementierungsrunden. Drei parallele Reviewer mit gebündeltem Feedback erzeugen eine.

Deshalb laufen drei C2-Agenten gleichzeitig auf demselben Diff, jeder mit genau einer Sichtweise:

| Agent | Prüft |
|---|---|
| Correctness | Erfüllt der Code die bestätigte Anforderung? Logikfehler, Sonderfälle, Fehlerpfade, Seiteneffekte, Abwärtskompatibilität |
| Security | Eingabevalidierung, Berechtigungen, Injection, Ausgabe-Escaping, Geheimnisse, Datenbank- und Migrationsauswirkungen |
| Code Quality | Konventionen aus `CONVENTIONS.md`, Lesbarkeit, Duplikate, Wartbarkeit, Testqualität, Doku |

Regeln für das Panel:

- Alle drei Agenten sind **read-only**. Kein Review-Agent ändert Code.
- Jeder Agent bekommt denselben Diff und denselben Anforderungsstand, nicht drei verschiedene Zusammenfassungen.
- Jeder Befund braucht `datei:zeile`, ein wörtliches Zitat der betroffenen Stelle, eine Schwere von 0 bis 100 und einen konkreten Korrekturvorschlag. Befunde ohne Belegstelle werden verworfen.
- Jeder Agent gibt am Ende ein Gesamturteil: `ok` oder `nicht ok`.

#### Standardauftrag für einen Review-Agenten

```text
Du reviewst, du änderst nichts. Keine Edits, keine Writes.

Lies zuerst:
- CONVENTIONS.md
- den relevanten Abschnitt aus TODO.md (bestätigte Anforderung und Nicht-Ziele)
- den Diff: <Befehl oder Dateiliste>

Deine Sichtweise: <Correctness | Security | Code Quality>
Prüfe ausschließlich diese Sichtweise. Was in eine andere Sichtweise gehört, lässt du weg.

Gib jeden Befund in dieser Form aus:
- ort: <datei:zeile>
  zitat: "<wörtlich die betroffene Zeile>"
  befund: <was ist falsch>
  schwere: <0-100>
  korrektur: <konkreter Vorschlag>

Ohne Belegstelle kein Befund.
Kein Lob, keine Zusammenfassung des Codes.

Letzte Zeile deiner Antwort, genau eines davon:
urteil: ok
urteil: nicht ok
```

### 7.2 Controller

Der Controller ist immer C1 im Hauptthread und wird nicht delegiert. Er sammelt alle drei Ergebnisse und entscheidet in dieser Reihenfolge:

1. Mindestens 2 von 3 sagen `ok` → Abschluss nach Abschnitt 11.
2. Sonst: höchste Schwere unter 80 → Abschluss, die offenen Kleinbefunde werden in `TODO.md` unter „Offene Punkte“ dokumentiert.
3. Sonst: Der Kern funktioniert und nur ein abgrenzbarer Rest fehlt → Rest-Scope als eigener Punkt unter „Follow-Up“ in `TODO.md`, Hauptteil abschließen. Kein Neuanfang für zehn Prozent Restarbeit.
4. Sonst: Versuchszähler prüfen. Zähler kleiner oder gleich 3 → das gebündelte Feedback aller drei Reviewer in **einem** Auftrag zurück an C2, Zähler erhöhen.
5. Sonst, Zähler über 3: Abbruch statt weiterer Runde. Die Aufgabe wird in drei bis fünf kleinere Teilaufgaben zerlegt, in `TODO.md` festgehalten und dem Nutzer mit dem neuen Zuschnitt vorgelegt. Der bisherige Stand wird nur nach dessen ausdrücklicher Zustimmung verworfen.

Der Versuchszähler steht in `TODO.md` und wird bei jeder Rückgabe an die Implementierung erhöht. Er ist hart. Drei gescheiterte Runden sind ein Zeichen für einen falschen Aufgabenzuschnitt, nicht für einen zu schwachen Agenten.

Sicherheitsbefunde ab Schwere 80, Architekturänderungen, Datenbankmigrationen und Änderungen an bestehenden Kundendaten entscheidet der Controller selbst und niemals über die Mehrheitsregel.

### 7.3 Was der Hauptthread zusätzlich prüft

- Diff auf ungewollte Änderungen und Reste,
- Einhaltung der ausdrücklichen Nicht-Ziele,
- Aktualität von `TODO.md`, `Analyse.md` und `Context.md`.

Nicht den gesamten Code erneut lesen, wenn eine fokussierte Prüfung genügt.

## 8. Data Contracts

Jede Übergabe zwischen zwei Jobs hat ein festes Schema. Freitext zwischen Agenten erzeugt Halluzination und doppelte Arbeit.

Befunde in `Analyse.md` werden in fester Form notiert:

```markdown
- id: A1
  ort: `pfad/zur/datei.php:412`
  zitat: "$items = array_slice( $items, 0, 500 );"
  befund: Findings werden bei 500 Einträgen abgeschnitten
  relevanz: hoch
```

Aussagen in `Context.md`, in `TODO.md` und in Agentenberichten, die sich auf den Code beziehen, verweisen auf diese `id` oder nennen selbst `datei:zeile` mit wörtlichem Zitat. Eine Behauptung über den Code ohne Belegstelle gilt als nicht belegt und darf keine Entscheidung tragen.

Für Review-Befunde gilt zusätzlich das Schema aus 7.1: `datei:zeile`, Zitat, Schwere, Korrekturvorschlag.

Der Nutzen ist doppelt: Der nächste Agent muss die Stelle nicht erneut suchen, und eine erfundene Stelle fällt beim Nachschlagen sofort auf.

## 9. Token-Optimierung

- Vorhandene Dokumentation immer zuerst lesen.
- `Analyse.md` für bereits erledigte Untersuchungen verwenden.
- `Context.md` als Übergabepunkt zwischen Agenten nutzen.
- `TODO.md` als verbindliche Aufgabenquelle verwenden.
- Keine bereits dokumentierten Suchabfragen wiederholen.
- Agentenaufträge kurz und referenzbasiert formulieren.
- Keine vollständigen Dateien an Agenten übergeben, wenn Dateipfade und Dokumentation ausreichen.
- Tests gebündelt am Ende eines Arbeitspakets ausführen.
- Keine unnötigen Zwischenberichte erzeugen.
- `/compact` nur an sinnvollen Schrittgrenzen verwenden.
- Textänderungen und einfache Dokumentationsaufgaben mit Haiku durchführen.
- Keine parallelen Änderungen an derselben Datei durchführen.
- Nach erfolgreichem Edit keine unnötige vollständige Wiederholung des Inhalts erzeugen.
- Deterministische Prüfungen vor teure Agenten schalten. Ein Review-Agent auf rotem Code ist verlorener Token-Einsatz.
- Unabhängige Jobs parallel laufen lassen. Sync-Barriere nur dort, wo ein Job mehrere Ergebnisse braucht.
- Review-Feedback immer gebündelt in einer Runde zurückgeben, nicht Befund für Befund.
- Befunde mit `datei:zeile` und Zitat übergeben, damit kein Agent dieselbe Stelle erneut sucht.
- Immer die billigste Agent-Klasse wählen, die den Job noch schafft.

## 10. Nicht delegieren

Folgende Aufgaben bleiben im Hauptthread:

- Sicherheitsentscheidungen,
- finale Sicherheitsreviews,
- Architekturentscheidungen mit Auswirkungen auf mehrere Systeme,
- Datenbankmigrationen,
- Änderungen an bestehenden Kundendaten,
- komplexe Fehleranalyse,
- Bewertung widersprüchlicher Anforderungen,
- finale Freigabe des Plans,
- die Controller-Entscheidung über das Review-Panel,
- das Zerlegen einer nach drei Versuchen gescheiterten Aufgabe,
- finale Abnahme des Ergebnisses.

## 11. Abschluss

Eine Aufgabe gilt erst als abgeschlossen, wenn:

- alle bestätigten Anforderungen umgesetzt sind,
- `TODO.md` vollständig aktualisiert ist,
- `Analyse.md` neue Erkenntnisse enthält,
- `Context.md` den aktuellen Stand widerspiegelt,
- die Quality Gates grün gegen die Baseline sind,
- relevante Tests und Prüfungen ausgeführt wurden,
- das zur Stufe passende Review gelaufen und die Controller-Entscheidung dokumentiert ist (Stufe klein: Diff-Sichtung durch C1),
- offene Punkte ausdrücklich dokumentiert sind.

Die Abschlussantwort erfolgt in der eingestellten Sprache und beginnt (falls konfiguriert) mit dem Namen des Nutzers. Sie enthält kurz:

- was umgesetzt wurde,
- welche Dateien geändert wurden,
- welche Prüfungen erfolgreich waren,
- welche offenen Punkte oder Risiken bestehen.

## 12. Referenzen

- `references/harness.md` — Harness-Modell, Review-Panel, Controller, Data Contracts, jeweils mit Diagramm.
- `references/diagramme/mmd/` — 14 Mermaid-Quellen der Diagramme.
- `references/diagramme/light/` — dieselben Diagramme als PNG.
