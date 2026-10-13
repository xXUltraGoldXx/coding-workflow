---
description: Richtet den coding-workflow Skill ein (Name und Standard-Projektgröße abfragen, Konfigurationsdatei schreiben)
---

Richte den Skill `coding-workflow` für diesen Nutzer ein. Antworte auf Deutsch.

1. Frage den Nutzer nach seinem **Namen** (Pflicht). Nenne er keinen, brich ab: ohne Namen keine Installation, nichts schreiben.
2. Frage nach der **Standard-Projektgröße** für seine Projekte: `klein`, `mittel`, `gross` oder `auto`. Standard (bei leerer Antwort) ist `auto`: der Skill entscheidet dann je Aufgabe selbst.
3. Frage, ob die Einstellung global (`~/.claude/coding-workflow.json`, Standard) oder nur für das aktuelle Projekt (`.claude/coding-workflow.json` im Projektordner, überschreibt die globale) gelten soll.
4. Prüfe, ob der Skill unter `~/.claude/skills/coding-workflow/SKILL.md` liegt; fehlt er, weise darauf hin (Installation laut README), schreibe die Konfiguration aber trotzdem.
5. Existiert die Zieldatei schon, zeige den alten Inhalt und frage vor dem Überschreiben nach.
6. Schreibe die Datei als JSON, genau dieses Format (`standard_groesse` nur `auto`, `klein`, `mittel` oder `gross`, ohne Umlaut; „groß“ als `gross` speichern):

```json
{"name": "<Name>", "standard_groesse": "auto"}
```

7. Bestätige kurz mit Pfad und gespeicherten Werten. Hinweis: Änderungen jederzeit durch erneutes `/cw-install`.
