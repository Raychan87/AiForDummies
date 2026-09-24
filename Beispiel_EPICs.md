Der Kern ist, dass Referenz und Templates als Dateien im Repo liegen. Die KI muss davon ableiten, statt zu erfinden, und Hooks prüfen das Ergebnis. Ein Vorbild wirkt bei der KI stärker als eine lange Regelbeschreibung, deshalb sind Templates hier das wichtigste Werkzeug.

## Welches Mittel für welche Aufgabe

| Aufgabe | Mittel |
|---|---|
| Grundregel "erst Vorbild suchen, nichts erfinden" | `AGENTS.md` |
| Regeln nur für bestimmte Dateien (`*.db`, `*.template`, `*.substitutions`, `st.cmd`) | `.github/instructions/*.instructions.md` mit `applyTo` |
| Referenz und Templates als Nachschlagewerk, nur bei Bedarf geladen | Skill (`references/`, `assets/`) |
| Wiederkehrende Aufgabe, z. B. "neuer Record aus Template X" | Prompt-Datei mit Eingaben |
| Feste Rollen: planen, umsetzen, gegen die Referenz prüfen | Custom Agents mit Handoffs |
| Erzwingen und automatisch prüfen | Hooks |
| Referenzmaterial liegt außerhalb des Repos (Wiki, Doku) | MCP, nur dann |
| Build-Ausgaben nicht lesen lassen | Ignore-Regeln |

## Mögliche Struktur

```
epics-projekt/
├── AGENTS.md
├── reference/                    ← Codereferenz, nur lesen
│   ├── beispiel-ioc/
│   └── konventionen.md
├── templates/                    ← Vorlagen für Records, Substitutions, st.cmd
├── .agents/skills/record-aus-template/
│   ├── SKILL.md
│   └── assets/                   (optional)
└── .github/
    ├── instructions/
    │   ├── db-dateien.instructions.md
    │   └── startskripte.instructions.md
    ├── prompts/neuer-record.prompt.md
    ├── agents/{planer,umsetzer,pruefer}.agent.md
    └── hooks/                    ← Schutz und Prüfung
```

`reference/` und `templates/` liegen im Hauptordner, damit Hooks, Skill und Menschen dieselbe Quelle nutzen. Der Skill verweist nur per Pfad darauf.

## Die wichtigsten Dateien

**`AGENTS.md`, Abschnitt "Referenz zuerst"**

```markdown
## Arbeitsweise: Referenz zuerst
- Vor jedem Record, jeder Substitutions-Datei und jedem Startskript: passendes Vorbild in
  `templates/` und `reference/` suchen und davon ableiten.
- Keine Record-Typen, Felder, Feldwerte oder Makros erfinden. Nur verwenden, was im
  Vorbild oder in der `.dbd`-Definition des Record-Typs vorkommt.
- Gibt es kein passendes Vorbild: nachfragen statt raten.
- Am Ende den Pfad des verwendeten Vorbilds nennen.

## Nie ändern
- `reference/` ist schreibgeschützt.
- Build-Ausgaben (`O.*`, `bin/`, `lib/`).
```

**`SKILL.md`**

```markdown
---
name: record-aus-template
description: Legt neue EPICS-Records, Substitutions-Dateien oder Startskript-Einträge nach den Vorlagen und der Codereferenz des Projekts an. Verwenden, wenn ein neuer Record, ein neues Gerät oder eine neue PV gebraucht wird.
---
1. Frage nach Zweck, Record-Typ und Signalnamen, falls nicht genannt.
2. Suche in `templates/` die passendste Vorlage, kopiere und passe sie an.
3. Prüfe jedes Feld gegen die `.dbd`-Definition des Record-Typs.
4. Halte die Namenskonvention aus `reference/konventionen.md` ein.
5. Nenne die verwendete Vorlage.
```

## Rolle der übrigen Bausteine

- **Instructions pro Dateityp:** Regeln nur für `*.db`, `*.template` und `*.substitutions` (z. B. Namensschema, Pflichtfelder) kommen in eine Datei mit `applyTo: "**/*.db,**/*.template,**/*.substitutions"`. So bleibt die `AGENTS.md` kurz.
- **Prompt-Datei `/neuer-record`:** fragt mit `${input:...}` nach Signalname und Record-Typ und ruft den Ablauf des Skills auf.
- **Agenten:** Ein Planer (nur lesen) wählt das Vorbild und legt den Plan vor. Ein Umsetzer schreibt. Ein Prüfer (nur lesen) vergleicht das Ergebnis mit dem Vorbild und meldet Abweichungen, per Handoff-Kette Planer → Umsetzer → Prüfer.
- **Hooks:**
  - Ein `PreToolUse`-Hook lehnt jede Änderung unter `reference/` hart ab (`deny`), wie im Beispiel mit dem Config-Ordner.
  - Ein `Stop`-Hook prüft vor dem Beenden geänderte Dateien, z. B. Template mit `msi` expandieren und die `.db` mit `softIoc` laden lassen. Er blockiert bei Fehlern, wie im Build-Beispiel. Prüfe die Befehle in deiner EPICS-Umgebung.
  - Ein Hook kann auch die Namenskonvention mit einem regulären Ausdruck prüfen. Eine Regel, die sich maschinell prüfen lässt, sollte kein Prompt sein.

## Der wichtigste Trick

Die KI erfindet bei EPICS am ehesten Feldnamen und Feldwerte, die es für den Record-Typ gar nicht gibt. Die verlässliche Quelle dafür sind die `.dbd`-Dateien deiner EPICS-Base-Installation. Verweise im Skill und in der `AGENTS.md` darauf und lass die KI jedes Feld dagegen prüfen.

## Reihenfolge für den Einstieg

1. `reference/` und `templates/` ins Repo, dazu die `AGENTS.md` mit "Referenz zuerst".
2. Instructions pro Dateityp.
3. Skill `record-aus-template`.
4. Hooks: Schutz von `reference/`, dann die Prüfung.
5. Prompt-Datei und Agenten, wenn die ersten Schritte sitzen.

Das bringt die größte Wirkung zuerst und lässt sich jederzeit ausbauen.
