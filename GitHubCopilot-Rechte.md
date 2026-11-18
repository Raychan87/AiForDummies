# GitHub Copilot in VS Code – Rechte und Schutz

Gilt für alle Modelle im Copilot-Chat (auch lokale und OpenAI-kompatible).
Einstellungen: `Strg+Shift+P` → **Preferences: Open User Settings (JSON)**

> Stand: November 2026, Windows. Wenn VS Code einen Einstellungsnamen grau unterkringelt, ist er in deiner Version unbekannt.

---

## 1. Die Stellschrauben

| Einstellung                                 | Was sie steuert                                                                       |
| ------------------------------------------- | ------------------------------------------------------------------------------------- |
| `chat.tools.global.autoApprove`           | Alles ohne Nachfrage (YOLO). Gilt für alle Workspaces.                               |
| `chat.tools.edits.autoApprove`            | Welche Dateien ohne Nachfrage geändert werden                                        |
| `chat.tools.terminal.autoApprove`         | Welche Terminalbefehle ohne Nachfrage laufen (`true`) oder immer fragen (`false`) |
| `chat.tools.urls.autoApprove`             | Welche URLs ohne Nachfrage abgerufen werden                                           |
| `chat.tools.eligibleForAutoApproval`      | Welche Tools überhaupt automatisch freigegeben werden dürfen                        |
| `chat.additionalReadAccessFolders`        | Zusätzliche Ordner nur zum Lesen                                                     |
| `chat.agent.sandbox.enabledWindows`       | Sandbox für Terminalbefehle (Windows: experimentell)                                 |
| `chat.agent.sandbox.fileSystem.windows`   | Sandbox-Pfade: `allowRead`, `allowWrite`, `denyRead`                            |
| `chat.agent.sandbox.network.allowNetwork` | Internet für Terminalbefehle in der Sandbox (`true`/`false`)                     |
| `chat.agent.maxRequests`                  | Wie viele Schritte der Agent am Stück machen darf                                    |

**Wichtig:**

- Die eingebauten Datei-Tools arbeiten nur im geöffneten Workspace.
- `terminal.autoApprove` prüft nur den Befehlstext. Ein Skript kann trotzdem alles tun.
- Echte Grenze für Terminalbefehle ist nur die Sandbox.
- Die Windows-Sandbox kennt keine Freigabe pro Domain: Internet ganz oder gar nicht.

### Regel-Syntax

```jsonc
"chat.tools.terminal.autoApprove": {
  "git status": true,                    // Befehl beginnt mit "git status"
  "/^dotnet (build|test)\\b/": true,     // Regex zwischen / /
  "rm": false                            // immer fragen
}

"chat.tools.edits.autoApprove": {
  "**/*": true,                          // alle Dateien im Workspace
  "**/.env": false                       // diese immer fragen
}

"chat.tools.urls.autoApprove": {
  "https://github.com/*": true,
  "https://example.com/api/*": { "approveRequest": true, "approveResponse": false }
}
```

---

## 2. Szenarien

### Stufe 1 – Nur zuschauen (fremder Code, Analyse)

Copilot fragt bei **jeder** Aktion.

```jsonc
{
  "chat.tools.global.autoApprove": false,
  "chat.tools.edits.autoApprove": { "**/*": false },
  "chat.tools.terminal.autoApprove": { "/.*/": false },
  "chat.tools.urls.autoApprove": {}
}
```

Tipp: Für reines Lesen den **Ask**-Modus im Chat statt **Agent** nehmen.

---

### Stufe 2 – Vorsichtig (Standard für den Alltag)

Dateien ändern und Lesebefehle frei, alles andere fragen.

```jsonc
{
  "chat.tools.global.autoApprove": false,
  "chat.tools.edits.autoApprove": {
    "**/*": true,
    "**/.git/**": false,
    "**/.vscode/**": false,
    "**/.env": false,
    "**/*.json": false
  },
  "chat.tools.terminal.autoApprove": {
    "/^git (status|diff|log)\\b/": true,
    "/^dir\\b|^ls\\b|^Get-ChildItem\\b/": true
  },
  "chat.tools.urls.autoApprove": {
    "https://github.com/*": true,
    "https://learn.microsoft.com/*": true
  }
}
```

---

### Stufe 3 – Autonom im Projekt, Internet fragen

Copilot arbeitet im Projekt frei, Git lokal frei. Push, Pull und Downloads fragen.

```jsonc
{
  "chat.tools.global.autoApprove": false,
  "chat.tools.edits.autoApprove": {
    "**/*": true,
    "**/.git/**": false,
    "**/.vscode/**": false,
    "**/.env": false
  },
  "chat.tools.terminal.autoApprove": {
    "/^git (status|diff|log|add|commit|branch|checkout|switch|stash)\\b/": true,
    "/^dotnet (build|test|run)\\b/": true,
    "/^npm (run|test)\\b/": true,
    "/^python \\.\\//": true,

    "git push": false, "git pull": false, "git clone": false, "git fetch": false,
    "git reset --hard": false, "git clean": false,
    "curl": false, "wget": false, "Invoke-WebRequest": false, "Invoke-RestMethod": false,
    "npm install": false, "pip": false, "dotnet add": false, "winget": false,
    "Remove-Item": false, "rm": false, "del": false, "rmdir": false,
    "format": false, "reg": false
  },
  "chat.tools.urls.autoApprove": {
    "https://github.com/*": true,
    "https://raw.githubusercontent.com/*": true,
    "https://api.github.com/*": true
  },

  // Optional, experimentell: echte Grenze für Terminalbefehle
  "chat.agent.sandbox.enabledWindows": "on",
  "chat.agent.sandbox.network.allowNetwork": false
}
```

Mit Sandbox und `allowNetwork: false` scheitert auch `git push`. Push, Pull und Clone machst du dann selbst.

---

### Stufe 4 – Vollautomatik (nur Wegwerf-Projekt, VM, Container)

```jsonc
{
  "chat.tools.global.autoApprove": true,
  "chat.agent.maxRequests": 100
}
```

Oder nur für eine Sitzung: `/yolo` im Chat, beenden mit `/disableYolo`.

**Nie** auf dem echten Rechner mit wichtigen Daten. Hier gibt es keine Rückfragen und keinen Schutz.

---

## 3. Dinge gezielt sperren

| Ziel                                          | Einstellung                                                                                   |
| --------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Sensible Dateien nicht ohne Nachfrage ändern | `edits.autoApprove`: `"**/.env": false`, `"**/*.pfx": false`, `"**/.git/**": false`   |
| Löschbefehle immer fragen                    | `terminal.autoApprove`: `"rm": false`, `"Remove-Item": false`, `"del": false`         |
| Downloads immer fragen                        | `"curl": false`, `"Invoke-WebRequest": false`, `"npm install": false`, `"pip": false` |
| Internet im Terminal komplett aus             | Sandbox an + `chat.agent.sandbox.network.allowNetwork: false`                               |
| Nur bestimmte Webseiten frei                  | `chat.tools.urls.autoApprove` mit `true`-Einträgen, Rest fragt                           |
| Keine globale Vollautomatik                   | `chat.tools.global.autoApprove: false`                                                      |
| Ordner außerhalb nur lesen                   | `chat.additionalReadAccessFolders`                                                          |
| Sandbox: Pfade lesen verbieten                | `chat.agent.sandbox.fileSystem.windows` → `denyRead`                                     |
| Git-Zugangsdaten nicht in der Sandbox         | `chat.agent.sandbox.credentials.authenticategit: false`                                     |

---

## 4. Pro Projekt anders

Datei `.vscode/settings.json` im Projektordner überschreibt die Benutzereinstellungen für dieses Projekt.
Beispiel: In einem Testprojekt Stufe 3, global Stufe 2.

Achtung: Diese Datei kann der Agent selbst ändern. Deshalb `"**/.vscode/**": false` in `edits.autoApprove`.

---

## 5. Testen

| Test im Chat                                             | Erwartung (Stufe 3) |
| -------------------------------------------------------- | ------------------- |
| „Führe `git status` aus“                            | läuft ohne Frage   |
| „Führe `curl https://example.com` aus“              | fragt               |
| „Lade https://example.com“                             | fragt               |
| „Lade https://github.com“                              | ohne Frage          |
| „Schreibe `test` nach `C:\test.txt`“ (mit Sandbox) | scheitert           |
| „Ändere die `.env`“                                 | fragt               |

---

## 6. Sicherheitsnetz

- Git: Vor jeder Agent-Sitzung committen. Fehler mit `git diff` und `git restore` zurückholen.
- NAS: Snapshots der Projekt-Freigabe.
- Regeln sind Schutz vor Versehen, nicht vor Absicht.
