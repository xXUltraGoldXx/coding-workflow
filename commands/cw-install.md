---
description: Sets up the coding-workflow skill (asks for name and default project size, writes the configuration file)
---

Set up the `coding-workflow` skill for this user. Reply in English.

1. Ask the user for their **name** (required). If they give none, abort: no name, no installation, write nothing.
2. Ask for the **default project size** for their projects: `small`, `medium`, `large` or `auto`. The default (on an empty answer) is `auto`: the skill then decides per task itself.
3. Ask whether the setting should apply globally (`~/.claude/coding-workflow.json`, default) or only to the current project (`.claude/coding-workflow.json` in the project folder, overrides the global one).
4. Check whether the skill exists at `~/.claude/skills/coding-workflow/SKILL.md`; if it is missing, point that out (installation per the README), but write the configuration anyway.
5. If the target file already exists, show its old content and ask before overwriting.
6. Write the file as JSON, in exactly this format (`default_size` only `auto`, `small`, `medium` or `large`):

```json
{"name": "<Name>", "default_size": "auto"}
```

7. Confirm briefly with the path and the stored values. Note: change it any time by running `/cw-install` again.
