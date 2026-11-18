# Claude Code: `managed-settings.json` steuern

> [!IMPORTANT]
> Die `managed-settings.json` hat **dasselbe Format** wie jede andere Claude-Code-`settings.json`. Der einzige Unterschied: Was hier steht, können Benutzer- und Projekteinstellungen **nicht überschreiben**.

## Inhalt

- [Location](#location)
- [Aufbau](#aufbau)
- [1. `permissions`: die drei Listen](#1-permissions-die-drei-listen)
- [2. Regel-Syntax](#2-regel-syntax)
  - [Pfade in `Read`- und `Edit`-Regeln](#pfade-in-read-und-edit-regeln)
- [3. Weitere nützliche Schalter in `permissions`](#3-weitere-nützliche-schalter-in-permissions)
- [4. Beispiel](#4-beispiel)
  - [Was einzelne `deny`-Regeln schützen](#was-einzelne-deny-regeln-schützen)
- [Grenze der Bash-Regeln](#grenze-der-bash-regeln)
- [Als Nächstes](#als-nächstes)

## Location

```bash
C:\Program Files\ClaudeCode\managed-settings.json
```

## Aufbau

```jsonc
{
  "permissions": { ... },   // Was darf Claude?
  "hooks": { ... },         // Eigene Skripte vor/nach Aktionen
  "env": { ... },           // Umgebungsvariablen
  "sandbox": { ... }        // Wirkt unter Windows nicht
}
```

Wichtig für dich sind vor allem **`permissions`** und eventuell **`hooks`**.

## 1. `permissions`: die drei Listen

```jsonc
"permissions": {
  "allow": [],   // ohne Nachfrage erlaubt
  "ask":   [],   // immer nachfragen
  "deny":  []    // immer verboten
}
```

> [!WARNING]
> Bei Konflikten gilt: **`deny` vor `ask` vor `allow`**.
> Steht `Bash(git *)` in `allow` und `Bash(git push *)` in `ask`, fragt Claude Code bei `git push` nach.

## 2. Regel-Syntax

| Regel                           | Bedeutung                               |
| ------------------------------- | --------------------------------------- |
| `Bash(git status)`            | genau dieser Befehl                     |
| `Bash(git *)`                 | alles, was mit `git` anfängt         |
| `Read(**/.env)`               | `.env`-Dateien in allen Ordnern lesen |
| `Edit(**/*.json)`             | JSON-Dateien ändern                    |
| `WebFetch`                    | jeder Webabruf                          |
| `WebFetch(domain:github.com)` | nur diese Domain                        |
| `WebSearch`                   | Websuche                                |
| `mcp__tia-portal`             | alle Tools eines MCP-Servers            |
| `mcp__tia-portal__export`     | ein einzelnes MCP-Tool                  |

### Pfade in `Read`- und `Edit`-Regeln

| Muster       | Bedeutung           |
| ------------ | ------------------- |
| `**/datei` | überall            |
| `~/…`     | dein Benutzerordner |
| `//…`     | absoluter Pfad      |

> [!NOTE]
> Wie Windows-Laufwerke genau geschrieben werden (z. B. `//K:/…`), solltest du mit dem `TEST-VERBOTEN.txt`-Trick prüfen.

## 3. Weitere nützliche Schalter in `permissions`

```jsonc
"defaultMode": "acceptEdits",
"disableBypassPermissionsMode": "disable",
"additionalDirectories": []
```

- **`defaultMode`** bestimmt, wie Claude Code startet:
  - `default`: fragt bei allem
  - `acceptEdits`: ändert Dateien ohne Nachfrage, fragt bei Befehlen
  - `plan`: liest nur und plant
- **`disableBypassPermissionsMode`**: Mit `"disable"` lässt sich der Modus „alles erlauben“ nicht einschalten.
- **`additionalDirectories`**: zusätzliche Ordner außerhalb des Projekts. Leer lassen, dann bleibt Claude Code im Projekt.

## 4. Beispiel

```jsonc
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable",
    "allow": [
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)","Bash(git add *)", "Bash(git commit *)",
      "Bash(dotnet build *)", "Bash(dotnet test *)", "Bash(npm run *)",
      "WebFetch(domain:github.com)", "WebFetch(domain:raw.githubusercontent.com)",  "WebFetch(domain:api.github.com)"
    ],
    "ask": [
      "WebSearch",
      "Bash(git push *)", "Bash(git clone *)", "Bash(git pull *)",
      "Bash(curl *)", "Bash(wget *)", "Bash(Invoke-WebRequest *)",
      "Bash(npm install *)", "Bash(pip install *)", "Bash(dotnet add *)",
      "Bash(winget *)"
    ],
    "deny": [
      "Read(**/.env)", "Read(**/*.pfx)", "Read(**/*.key)",
      "Read(~/.ssh/**)",
      "Bash(git push --force *)",
      "Bash(Remove-Item -Recurse *)", "Bash(rm -rf *)",
      "Bash(format *)", "Bash(reg *)",
      "Edit(**/.git/**)",
      "Edit(**/.claude/settings*.json)",
      "Edit(**/.vscode/settings.json)",
      "Read(**/*.kdbx)",
      "Bash(git reset --hard *)", "Bash(git clean *)"
    ]
  }
}
```

Was die Listen bewirken:

- **`allow`**: Routine im Projekt läuft ohne Rückfrage.
- **`ask`**: Alles, was ins Internet geht oder Pakete holt, fragt vorher.
- **`deny`**: Geheimnisse lesen und gefährliches Löschen ist verboten.

### Was einzelne `deny`-Regeln schützen

| Regel                    | Zweck                                                                           |
| ------------------------ | ------------------------------------------------------------------------------- |
| `.git`                 | Die Git-Historie bleibt unangetastet, damit Fehler rückgängig zu machen sind. |
| `.claude`, `.vscode` | Die KI kann ihre eigenen Regeln im Projekt nicht lockern.                       |
| `*.kdbx`               | Keine KeePass-Datenbanken lesen.                                                |
| `reset`, `clean`     | Keine Änderungen ohne Rückweg verwerfen.                                      |

## Grenze der Bash-Regeln

> [!CAUTION]
> Die `Bash`-Regeln prüfen nur den **Befehlstext**. `python -c "..."` oder ein Skript im Projekt umgeht sie.
> Die Regeln für `Read`, `Edit` und `WebFetch` sind dagegen verlässlich, weil sie die eingebauten Tools selbst betreffen.

## Als Nächstes

Als Nächstes könnten wir `hooks` anschauen, also eigene Prüfskripte, z. B. eine PowerShell-Version des Papierkorb-Wächters. Oder willst du erst diese Regeln testen?
