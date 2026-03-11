# Sinnvoller Einsatz von Claude - Kontext bewahren zwischen chat<->claude-project<->claude.md<->...



## Prompt

übersichtliche erklärung der workflows und den sinnvollen einsatz:
* claude-project(https://claude.ai/project)` <-> chat(https://claude.ai/recents) <-> CLAUDEE.md <-> ...?`
* konkretes beispielszenario: myproject(csharp-app)<->multiple_chats(not-too-long).preserve_context(CLAUDE.md|claude-project|...)?
* claude, leite meine absicht ab und ergänze meinen prompt im sinne einer verbesserung!


## Antwort

Lass mich kurz die bisherigen Docs lesen, um nahtlos darauf aufzubauen:Gut – ich kenne den Stand. Jetzt erstelle ich das Workflow-Dokument:Zu deinem letzten Punkt – **abgeleitete Absicht und Prompt-Verbesserung:**

Dein eigentliches Ziel war: *„Ich entwickle eine C#-App, arbeite in mehreren kurzen Chats, und will verstehen wie ich Kontext sauber zwischen diesen Chats, Claude Code (Neovim), claude.ai-Projekten und CLAUDE.md verteile – ohne Redundanz, ohne Token-Verschwendung."*

Der Kern des Dokuments beantwortet genau das mit dem **Kontext-Verteilungsschema**:

```
Dauerhafte Konventionen    → Projektanweisungen (claude.ai)
Schema / Architektur-Doku  → Wissensbasis (claude.ai KB)
Build-Cmds / Folder-Logik  → CLAUDE.md (im Repo, für Claude Code)
Persönliche IDE-Präfernz.  → ~/.claude/CLAUDE.md (global)
Chat-zu-Chat-Übergang      → Transition-Zusammenfassung (manuell)
```

Das kritischste Pattern daraus: **ein Chat = eine abgeschlossene Aufgabe**, Übergang per generierter 3–5-Punkte-Zusammenfassung. Damit bleibt jeder Chat schlank, Claude hat immer frischen Kontext, und du verlierst keine Information zwischen den Sessions.

---

# CLAUDE-workflows.md – Workflows & Sinnvoller Einsatz

> **Stand:** März 2026 · Aufbauend auf CLAUDE.md + CLAUDE-limits.md

---

## 1. Das Werkzeug-Ökosystem im Überblick

Bevor Workflows erklärt werden: Was ist was, und wofür ist es gedacht?

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLAUDE ÖKOSYSTEM                         │
├──────────────────┬──────────────────┬───────────────────────────┤
│  claude.ai/      │  claude.ai/      │  Claude Code              │
│  recents         │  project/...     │  (Terminal / Neovim)      │
│  (freie Chats)   │  (Projekte)      │                           │
├──────────────────┼──────────────────┼───────────────────────────┤
│ Einzelne Aufgabe │ Themen-Workspace │ Agentisches Coding        │
│ Kein Kontext-    │ Wissensbasis     │ Liest Dateien direkt      │
│ overhead         │ Projektanweisun  │ CLAUDE.md als Gedächtnis  │
│                  │ -gen persistent  │                           │
├──────────────────┼──────────────────┼───────────────────────────┤
│ Kontext: Chat    │ Kontext:         │ Kontext:                  │
│ (nur dieser      │ KB + Anweisungen │ CLAUDE.md + aktiver Chat  │
│ Verlauf)         │ + Chat-Verlauf   │ + gelesene Dateien        │
└──────────────────┴──────────────────┴───────────────────────────┘

         ⚠️ Alle teilen dasselbe Token-Kontingent (Pro/Max)!
```

### Die vier Kontext-Ebenen

| Ebene | Werkzeug | Lebt in | Persistent? |
|-------|----------|---------|-------------|
| **Session** | Alle | Aktiver Chat-Verlauf | Nein – endet mit Chat |
| **Projekt** | claude.ai Projekte | Wissensbasis + Anweisungen | Ja – manuell verwaltet |
| **Lokal** | Claude Code | `CLAUDE.md` im Repo | Ja – committed |
| **Global** | Claude Code | `~/.claude/CLAUDE.md` | Ja – persönlich |

---

## 2. Workflow-Entscheidungsbaum

```
Habe ich eine Aufgabe für Claude...
│
├── ...die ich einmalig erledige und nie wieder brauche?
│   └── → Freier Chat (claude.ai/recents)
│       Kein Overhead, kein Setup, einfach starten.
│
├── ...die zu einem Thema/Projekt gehört, das wiederkehrt?
│   └── → Claude-Projekt (claude.ai/project/...)
│       Wissensbasis + Anweisungen einmalig konfigurieren.
│       Pro Chat: Aufgabe fokussiert halten, nicht ausufern lassen.
│
├── ...bei der ich Code schreibe und direkt im Repo arbeite?
│   └── → Claude Code (Terminal / Neovim via claudecode.nvim)
│       CLAUDE.md als Kontext-Anker im Repo.
│       Chat-Splits: ein Feature / ein Bugfix pro Session.
│
└── ...die eine Kombination braucht?
    └── → Hybrid-Workflow (siehe Abschnitt 4)
        Claude Code für Implementierung,
        claude.ai Projekt für Analyse / Review / Doku.
```

---

## 3. Die Werkzeuge im Detail: Wann was?

### 3.1 Freier Chat – `claude.ai/recents`

**Sinnvoll wenn:**
- Einmalige Fragen, Recherche, Textentwürfe
- Schnelle Code-Snippets ohne Projektkontext
- Experimente: "Kann Claude das überhaupt?"
- Vertrauliches (kein Projektkontext notwendig)

**Nicht sinnvoll wenn:**
- Du dieselben Grundlagen jedes Mal neu erklärst
- Die Aufgabe zu einem größeren Thema gehört
- Du zwischen Chats Kontinuität brauchst

**Token-Verhalten:** Nur Chat-Verlauf, kein Overhead durch KB oder Anweisungen.  
→ Günstigste Option für kurze, fokussierte Aufgaben.

---

### 3.2 Claude-Projekt – `claude.ai/project/...`

**Sinnvoll wenn:**
- Wiederkehrender Themenkontext (Projekt, Kunde, Fachgebiet)
- Referenzdokumente (Schema, Guidelines, Specs) wiederverwendet werden
- Einheitlicher Ton/Stil über viele Chats gewünscht
- Team-Zusammenarbeit (Team/Enterprise)

**Nicht sinnvoll wenn:**
- Einmalige Aufgabe ohne Wiederverwendung
- Wissensbasis ist leer (dann bringt Overhead nichts)
- Aufgabe ist komplett unabhängig vom Projektthema

**Interna:**
```
Projekt-Chat Ablauf:
  1. Du sendest Nachricht
  2. Claude lädt Projektanweisungen (immer)
  3. Claude prüft Wissensbasis via RAG (relevante Teile)
  4. Claude verarbeitet Chat-Verlauf (kumulierend!)
  5. Claude generiert Antwort
```

**Wissensbasis richtig befüllen:**
```
✓ DB-Schema (kompakt, z.B. als CREATE TABLE Snippets)
✓ Architektur-Überblick (1–2 Seiten max)
✓ Sprachliche Konventionen, Glossar
✓ Wichtige Interface-Definitionen / API-Contracts
✗ Komplette Codebase (zu groß, schlechtes RAG-Signal)
✗ Veraltete Dokumente (schlechte Antworten)
✗ Dinge, die sich wöchentlich ändern
```

**Projektanweisungen (≈ CLAUDE.md für den Web-Chat):**
```
Beispiel C#-Projekt:
"Du bist ein erfahrener C#/.NET Entwickler.
Antworte auf Deutsch.
Nutze immer aktuelle .NET-Patterns (Minimal API, Records, Nullable).
Erkläre deine Entscheidungen kurz aber präzise.
Bei SQL: T-SQL Syntax (MS SQL Server), sp_executesql für dynamische Queries."
```

---

### 3.3 CLAUDE.md – für Claude Code

**Sinnvoll wenn:**
- Du Claude Code (Terminal/Neovim) nutzt
- Du nicht jedes Mal denselben Projektkontext erklären willst
- Mehrere Personen am selben Repo arbeiten

**Funktionsprinzip:**
```
Jede Claude Code Session:
  1. ~/.claude/CLAUDE.md  laden (global)
  2. Repo-Root CLAUDE.md laden (projekt)
  3. Subdirectory CLAUDE.md laden (modul, falls vorhanden)
  4. Alle drei kombiniert = aktiver Kontext
  5. Dann erst deine Aufgabe bearbeiten
```

**Nicht sinnvoll für:**
- claude.ai Web-Chats (dort → Projektanweisungen)
- Einmalige Repos die du nie wieder öffnest

---

## 4. Konkretes Szenario: C#-App mit mehreren Chats

### Setup: `myproject` (C# Web-App, MS SQL Backend)

```
myproject/
├── CLAUDE.md                 ← Projektgedächtnis für Claude Code
├── CLAUDE.local.md           ← Persönliche Overrides (in .gitignore)
├── src/
│   ├── Api/
│   ├── Services/
│   └── Data/
└── docs/
    ├── schema.md             ← DB-Schema Referenz
    └── architecture.md      ← Architektur-Überblick
```

```
claude.ai → Projekt "myproject"
├── Projektanweisungen: C#-Konventionen, Sprache, Stil
├── Wissensbasis:
│   ├── schema.md  (hochgeladen)
│   └── architecture.md  (hochgeladen)
└── Chats:
    ├── [Chat] "API-Design: User Endpoints"         (abgeschlossen)
    ├── [Chat] "SQL-Query: Report-Logik"            (abgeschlossen)
    ├── [Chat] "Bugfix: Null-Reference in Service"  (aktiv)
    └── ...
```

---

### Workflow: Eine Aufgabe von Anfang bis Ende

**Phase 1: Analyse / Design (claude.ai Projekt-Chat)**

Für Planung und Architektur ist der Web-Chat ideal – du brauchst keine Datei-Zugriffe, nur Kontext:

```
Chat: "API-Design: User Endpoints"
─────────────────────────────────
Du:     "Ich brauche REST-Endpoints für User-Management.
         Anforderungen: [Liste]. Welche Endpoints schlägst du vor?"

Claude: [nutzt Projektanweisungen + schema.md aus KB]
        → Schlägt Endpoint-Struktur vor

Du:     "Gut. Wie würde das DTO-Design aussehen?"

Claude: → DTO-Klassen als C# Records

Du:     "Fass die Entscheidungen kurz zusammen."
        ← WICHTIG: Zusammenfassung für Transition!

[Chat abschließen – Aufgabe erledigt]
```

**Phase 2: Implementierung (Claude Code / Neovim)**

Für das eigentliche Coding, direkt im Repo:

```bash
# In Neovim: <leader>a öffnet claude Code Panel
# Oder im Terminal:
cd myproject
claude

# Erste Nachricht: Transition-Kontext aus Phase 1
"Wir haben folgende API-Endpoints designed: [Zusammenfassung].
 Implementiere jetzt den UserController mit diesen Endpoints.
 DTOs als Records, Minimal API Pattern."

# Claude liest CLAUDE.md automatisch, kennt also:
# - Projektstruktur
# - Coding-Konventionen
# - Build-Commands
```

**Phase 3: Review / Dokumentation (neuer Projekt-Chat)**

```
Chat: "Code Review: UserController"
────────────────────────────────────
Du:   [Code einfügen oder Datei hochladen]
      "Reviewe diesen Controller auf:
       - .NET Best Practices
       - Potenzielle Null-Reference Issues
       - SQL-Injection-Risiken bei dynamischen Queries"

[Separater Chat → sauberer Kontext, keine Implementierungsdetails die stören]
```

---

### Das Kern-Prinzip: Chat-Splitting

```
Ein Chat = Eine abgeschlossene Aufgabe

❌ Nicht so:
┌─────────────────────────────────────────────────────┐
│ Mega-Chat (Wochen, 80+ Nachrichten)                 │
│ • API Design                                         │
│ • SQL-Queries                                        │
│ • Bugfixes                                           │
│ • Doku                                               │
│ • Nochmal ein Bugfix                                 │
│ • "Kannst du nochmal das vom Anfang erklären?"       │
│   → Claude hat es "vergessen" (Kontext-Overflow)    │
└─────────────────────────────────────────────────────┘

✅ So:
┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐
│ Chat 1    │  │ Chat 2    │  │ Chat 3    │  │ Chat 4    │
│ API Design│→T│ Controller│→T│ SQL Query │→T│ Review    │
│ [Fertig]  │  │ [Fertig]  │  │ [Fertig]  │  │ [Fertig]  │
└───────────┘  └───────────┘  └───────────┘  └───────────┘
       T = Transition-Zusammenfassung (1 Absatz)
```

**Transition-Template:**

```
"Fasse die wichtigsten Entscheidungen und Ergebnisse 
dieses Chats in 3–5 Bullet-Points zusammen, 
so dass ich den Kontext in einem neuen Chat 
effizient weitergeben kann."
```

Diese Zusammenfassung kopieren → erster Satz im nächsten Chat.

---

### Kontext-Erhaltung: Welches Werkzeug für was?

```
KONTEXT DEN ICH BRAUCHE          WO ICH IHN ABLEGE
─────────────────────────────────────────────────────
Dauerhafte Projektkonventionen  → claude.ai Projektanweisungen
DB-Schema, Architektur-Doku     → claude.ai Wissensbasis (KB)
Build-Commands, Folder-Struktur → CLAUDE.md (im Repo)
Persönliche IDE-Präferenzen     → ~/.claude/CLAUDE.md (global)
Aufgaben-Übergang (Chat→Chat)   → Transition-Zusammenfassung (manuell)
Temporärer Task-Kontext         → Erster Satz im neuen Chat
```

---

## 5. Vollständige Werkzeug-Matrix

```
                    FREIER CHAT   PROJEKT-CHAT  CLAUDE CODE
                    ──────────    ────────────  ───────────
Setup-Aufwand       keine         mittel        hoch (einmalig)
Persistenz          –             KB+Anweisun.  CLAUDE.md
Datei-Zugriff       Upload        Upload/KB     Direkt (Repo)
Gut für Analyse     ✓✓            ✓✓✓           ✓
Gut für Coding      ✓             ✓✓            ✓✓✓
Gut für Planung     ✓✓✓           ✓✓✓           ✓
Team-fähig          –             ✓✓ (Team+)    ✓ (via Repo)
Token-Overhead      niedrig       mittel        hoch (Dateien)
Ideal für           Einmaliges    Wiederkehrend Implementierung
```

---

## 6. Anti-Patterns – Was vermeiden?

### Anti-Pattern 1: Der Kontext-Monster-Chat
Wochenlange Konversation zu allem. Wird langsamer, teurer, Claude "vergisst" frühe Entscheidungen durch Kontext-Overflow.

### Anti-Pattern 2: KB mit der kompletten Codebase
Alle `.cs`-Dateien hochladen. RAG findet nichts, weil das Signal-zu-Rauschen-Verhältnis schlecht ist. Nur Kerndokumente hochladen.

### Anti-Pattern 3: CLAUDE.md als Essay
500-Zeilen CLAUDE.md mit allem, was man je über das Projekt weiß. Claude ignoriert die Hälfte, Kontext-Budget verpufft.

### Anti-Pattern 4: Kein Chat-Split beim Themensprung
Von "API-Design" zu "SQL-Performance" im selben Chat ohne Split. Der frühe API-Kontext stört die SQL-Diskussion und kostet unnötig Tokens.

### Anti-Pattern 5: Wissensbasis nie aktualisieren
Veraltetes Schema in der KB führt zu falschen Antworten. KB-Dokumente regelmäßig aktualisieren wenn sich die Realität ändert.

---

## 7. Empfohlenes Setup für `myproject` (Starter)

### Schritt 1: claude.ai Projekt anlegen
```
Name: myproject – C# App
Projektanweisungen:
  "C#/.NET Entwicklung (Minimal API, Records, Nullable enabled).
   MS SQL Server, sp_executesql für dynamische Queries.
   Antworte auf Deutsch, erkläre Entscheidungen kurz.
   Nutze moderne .NET-Patterns, keine Legacy-Patterns."

Wissensbasis (initial):
  → docs/schema.md hochladen
  → docs/architecture.md hochladen
```

### Schritt 2: CLAUDE.md im Repo
```markdown
# CLAUDE.md – myproject

## Project
C# Minimal API + MS SQL Server Backend.

## Commands
dotnet build          # Build
dotnet test           # Tests
dotnet run --project src/Api  # Dev-Server

## Architecture
/src/Api       → Minimal API Endpoints, DTOs
/src/Services  → Business Logic
/src/Data      → Repository Pattern, SQL-Queries
/docs          → Schema (@docs/schema.md), Architektur

## Conventions
- Records für DTOs (immutable)
- sp_executesql für alle dynamischen SQL-Queries
- NEVER string concatenation in SQL
- Nullable enabled: immer null-checks

## Reference
- DB-Schema: @docs/schema.md (lesen bei neuen Queries)
- Architektur: @docs/architecture.md (lesen bei neuen Services)
```

### Schritt 3: Globale CLAUDE.md
```markdown
# ~/.claude/CLAUDE.md – Global

## Language
Deutsch bevorzugt. Fachbegriffe auf Englisch OK.

## Style
Kurze Erklärungen. Entscheidungen begründen.
Code-First: Code zeigen, dann erklären wenn nötig.

## SQL
MS SQL Server Syntax. sp_executesql. TRY/CATCH.
```

### Schritt 4: Workflow etablieren

```
Analyse/Design     → claude.ai Projekt-Chat
Implementierung    → Claude Code (Neovim: <leader>a)
Review/Doku        → claude.ai Projekt-Chat (neuer Chat)
Chat-Limit         → Max ~15–20 Nachrichten, dann Split
Transition         → Immer Zusammenfassung am Chat-Ende
KB-Refresh         → Bei Schema-Änderungen aktualisieren
CLAUDE.md-Review   → Monatlich, nach Code-Reviews
```

---

## 8. Kurzreferenz-Karte

```
┌─────────────────────────────────────────────────────────────┐
│  AUFGABE                    WO STARTEN                      │
├─────────────────────────────────────────────────────────────┤
│  Einmalige Frage            Freier Chat                     │
│  Planung / Architektur      Projekt-Chat + KB               │
│  Feature implementieren     Claude Code + CLAUDE.md         │
│  Code Review                Neuer Projekt-Chat              │
│  SQL Query schreiben        Projekt-Chat (Schema in KB)     │
│  Bugfix                     Claude Code (direkt im Code)    │
│  Dokumentation              Projekt-Chat                    │
├─────────────────────────────────────────────────────────────┤
│  Chat wird lang (>15 Msg)?  → Split + Transition            │
│  Thema wechselt?            → Neuer Chat                    │
│  Kontext "vergessen"?       → KB prüfen / CLAUDE.md prüfen  │
│  Limits erreicht?           → Modell wechseln (Sonnet!)     │
└─────────────────────────────────────────────────────────────┘
```

---

*Aufbauend auf: CLAUDE.md (Grundlagen) + CLAUDE-limits.md (Limits & Effizienz)* 
*Stand: März 2026*

---

# Konkreter Setup für den Arbeitsplatz

## Prompt

Empfohlenes Setup für `myproject` (Starter):
* aufgaben und einsatzzwecke:
   * entwicklung des c#-apps: features, fehlerbehebung, analyse, visualisierung, berichterstellung, etc
* prompts und anweisungen: 
   * alle prompts und anweisungen, wie z.b. zum erstellen der vorgeschlagenen dateien `schema.md`, `architecture.md`, etc, einschließen.
* umgebung und konkretes szenario:
   * setup-umgebung:
      * arbeitsplatz: kein claude-code!
      * github-repo: csharp-projektdateien.

## Antwort

Lass mich kurz die bisherigen Docs lesen für vollständige Kontinuität:Klar – kein Claude Code, nur claude.ai Web + GitHub-Repo. Das ändert den Stack fundamental. Jetzt schreibe ich das vollständige Setup-Dokument mit allen konkreten Prompts:Das Dokument ist vollständig auf dein Setup zugeschnitten. Die wichtigste Korrektur gegenüber dem vorherigen Workflow-Doc:

**`CLAUDE.md` entfällt komplett** – das ist ein Claude Code-Feature für Terminal-Sessionen. In deinem Setup übernimmt das `docs/`-Trio im Repo (schema, architecture, conventions) die Rolle des persistenten Kontexts, aber als Wissensbasis im claude.ai-Projekt statt als automatisch geladene Datei.

Die drei Kern-Punkte des Setups in der Kurzform:

```
Repo-Docs (docs/*.md)     → Wissensbasis im claude.ai Projekt
Projektanweisungen        → Permanenter Kontext (ersetzt CLAUDE.md)
Transition-Snapshots      → Brücke zwischen Chats (manuell generiert)
```

Und der entscheidende praktische Hinweis für den Code-Austausch ohne Claude Code: nie die ganze Datei einfügen – nur den relevanten Ausschnitt (Klasse, Methode, Query). Das hält jeden Chat schlank und vermeidet frühen Kontext-Overflow.

---

# CLAUDE-setup-myproject.md
## Vollständiges Setup: myproject (C#-App) auf claude.ai
### Ohne Claude Code · GitHub-Repo · Web-only Workflow

> **Stand:** März 2026  
> **Umgebung:** claude.ai Web (Pro/Max) · GitHub · kein Claude Code · kein Terminal-Zugang zu Claude  
> **Aufbauend auf:** CLAUDE.md · CLAUDE-limits.md · CLAUDE-workflows.md

---

## 0. Architektur des Setups (Übersicht)

```
╔══════════════════════════════════════════════════════════════════╗
║                    MYPROJECT – SETUP ÜBERSICHT                   ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║   GitHub-Repo (myproject)          claude.ai/project/myproject   ║
║   ─────────────────────────        ─────────────────────────     ║
║   /docs/                           Projektanweisungen            ║
║     schema.md          ──upload──▶ Wissensbasis (KB):            ║
║     architecture.md    ──upload──▶  • schema.md                  ║
║     conventions.md     ──upload──▶  • architecture.md            ║
║   /src/                             • conventions.md             ║
║     (C#-Quellcode)     ──paste───▶  • [bei Bedarf: Dateiauszug]  ║
║                                                                  ║
║   Kein CLAUDE.md nötig!            Projektanweisungen =          ║
║   (ist Claude Code-Feature)        Permanenter Kontext           ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

**Kernprinzip ohne Claude Code:**
- GitHub-Repo = Quellcode-Heimat (wie immer)
- `docs/`-Ordner im Repo = Referenzdokumente (schema, architecture, ...)
- claude.ai Projekt = KI-Arbeitsumgebung mit KB aus diesen Docs
- Kein `CLAUDE.md` (das ist ein Claude Code-Feature für Terminal-Sessionen)
- Code-Austausch: manuell per Copy/Paste in den Chat

---

## 1. GitHub-Repo: Struktur einrichten

### Empfohlene Repo-Struktur

```
myproject/
├── docs/                          ← Referenzdokumente für KB
│   ├── schema.md                  ← DB-Schema (Hauptreferenz)
│   ├── architecture.md            ← Architektur-Überblick
│   ├── conventions.md             ← Coding-Konventionen
│   └── context-snapshots/         ← Chat-Übergänge (optional)
│       ├── 2026-03-feature-x.md   ← Transition-Zusammenfassungen
│       └── ...
├── src/
│   ├── MyProject.Api/             ← ASP.NET Minimal API
│   ├── MyProject.Services/        ← Business Logic
│   ├── MyProject.Data/            ← Repository / SQL
│   └── MyProject.Tests/           ← xUnit Tests
└── README.md
```

> Die `docs/`-Dateien werden einmal erstellt (→ Abschnitt 2), ins Repo committed, und in die claude.ai-Wissensbasis hochgeladen. Bei Änderungen: Datei aktualisieren, neu hochladen.

---

## 2. Referenzdokumente erstellen – Alle Prompts

Die folgenden Prompts werden jeweils in einem **freien Chat** (nicht im Projekt) ausgeführt, da es sich um einmalige Erstellungsaufgaben handelt. Das Ergebnis wird als `.md`-Datei ins Repo gespeichert.

---

### 2.1 `docs/schema.md` erstellen

**Kontext:** Du hast bereits ein bestehendes Datenbankschema (oder weißt was du brauchst).

**Prompt A – Schema aus bestehendem SQL generieren:**
```
Ich habe folgendes MS SQL Server Datenbankschema (CREATE TABLE Statements).
Erstelle daraus eine kompakte schema.md Referenzdatei für mein Entwicklungsteam und für KI-Assistenten.

Format:
- Jede Tabelle: Name, Kurzbeschreibung (1 Satz), Spalten mit Typ und Zweck
- Beziehungen zwischen Tabellen (Foreign Keys, logische Verknüpfungen)
- Wichtige Constraints und Besonderheiten
- Maximale Länge: 2–3 Seiten (token-effizient, kein Overhead)

Mein Schema:
[HIER SQL EINFÜGEN]
```

**Prompt B – Schema von Grund auf entwickeln:**
```
Ich entwickle eine C#-App mit MS SQL Server Backend.
Zweck der App: [KURZE BESCHREIBUNG – z.B. "Projektverwaltung für kleine Teams"]

Hauptentitäten: [z.B. "Projekte, Aufgaben, Benutzer, Zeiterfassung"]

Erstelle:
1. Ein sinnvolles MS SQL Datenbankschema (CREATE TABLE Statements, T-SQL)
2. Direkt danach: eine schema.md Referenzdatei im folgenden Format:

# Schema: myproject

## Tabellen
### [Tabellenname]
**Zweck:** [1 Satz]
| Spalte | Typ | Beschreibung |
|--------|-----|--------------|
| ...    | ... | ...          |

**Beziehungen:** [Foreign Keys, logische Verknüpfungen]

## Wichtige Constraints
[Unique Constraints, Check Constraints, Besonderheiten]

## Indizes (empfohlen)
[Performance-relevante Indizes]

Halte die Datei unter 300 Zeilen – sie wird als KI-Kontext verwendet.
```

---

### 2.2 `docs/architecture.md` erstellen

**Prompt:**
```
Ich entwickle eine C#/.NET Anwendung. Erstelle eine architecture.md 
Referenzdatei für Entwickler und KI-Assistenten.

Projektdetails:
- App-Typ: [z.B. "ASP.NET Core Web API / Minimal API"]
- Frontend: [z.B. "Blazor / React / keins (nur API)"]
- Backend: C# / .NET 9
- Datenbank: MS SQL Server
- Hosting: [z.B. "Azure App Service / lokaler Server / noch offen"]
- Wichtige Bibliotheken: [z.B. "Dapper, FluentValidation, MediatR"]

Projektstruktur:
[ENTWEDER: eigene Struktur einfügen ODER: "noch nicht definiert, schlage etwas vor"]

Erstelle die architecture.md mit folgenden Abschnitten:

# Architecture: myproject

## Überblick
[1–2 Sätze: Was macht die App, welches Architekturmuster]

## Tech-Stack
[Tabelle: Komponente | Technologie | Version/Begründung]

## Projektstruktur
[Verzeichnisbaum mit Erklärung jedes Ordners]

## Schichten & Verantwortlichkeiten
[Welche Schicht macht was, wo liegt welche Logik]

## Datenfluss
[Kurzes Beispiel: Request → Controller → Service → Repository → DB]

## Wichtige Entscheidungen / Constraints
[Architektur-Entscheidungen die jeder kennen muss]

Halte es unter 200 Zeilen. Fokus auf das Wesentliche für KI-Kontext.
```

---

### 2.3 `docs/conventions.md` erstellen

**Prompt:**
```
Erstelle eine conventions.md für mein C#/.NET Projekt.
Diese Datei wird von KI-Assistenten (Claude) als Kontext genutzt,
daher: knapp, präzise, keine Selbstverständlichkeiten.

Meine Präferenzen / Constraints:
- .NET Version: [z.B. ".NET 9"]
- Nullable: enabled
- Coding-Style: [z.B. "Records für DTOs, keine public Fields, keine var bei nicht-offensichtlichen Typen"]
- SQL: MS SQL Server, sp_executesql für dynamische Queries, NIEMALS String-Konkatenation
- Fehlerbehandlung: [z.B. "TRY/CATCH in SQL, Result<T> Pattern in C#"]
- Tests: [z.B. "xUnit, AAA-Pattern (Arrange/Act/Assert)"]
- Git: [z.B. "Feature-Branches, Conventional Commits"]
- Sprache: [z.B. "Code auf Englisch, Kommentare auf Deutsch"]

Erstelle conventions.md im Format:

# Conventions: myproject

## C# / .NET
[Bullet-Points, nur nicht-offensichtliche Regeln]

## SQL (MS SQL Server)
[Bullet-Points]

## Fehlerbehandlung
[Pattern-Beschreibung]

## Tests
[Kurze Konventionen]

## Git
[Branch-Naming, Commit-Format]

## Was NIEMALS tun
[Kritische No-Gos als NEVER-Liste]

Maximal 150 Zeilen.
```

---

### 2.4 `docs/context-snapshots/` – Transition-Template

Dieses Dokument wird nicht einmalig erstellt, sondern am Ende jedes Chats generiert. Template für den Prompt:

```
Fasse diesen Chat als Transition-Snapshot zusammen.
Format:

# Context Snapshot: [DATUM] – [THEMA]

## Was wurde entschieden
- [Entscheidung 1]
- [Entscheidung 2]

## Was wurde implementiert / erstellt
- [Code/Datei/Feature]

## Offene Punkte / Nächste Schritte
- [Was kommt als nächstes]

## Wichtiger Kontext für den nächsten Chat
[1–2 Sätze die am Anfang des nächsten Chats als Einleitung dienen]

Halte es unter 20 Zeilen – es ist ein Übergabe-Dokument, kein Roman.
```

---

## 3. claude.ai Projekt einrichten

### Schritt 1: Projekt anlegen

```
URL: https://claude.ai/projects
→ "+ Neues Projekt"
→ Name: "myproject – C# App"
→ Beschreibung: (optional, Claude sieht das nicht)
```

### Schritt 2: Projektanweisungen setzen

**Vollständiger Text für das Anweisungsfeld:**

```
Du bist ein erfahrener C#/.NET und MS SQL Server Entwickler.

SPRACHE & STIL
- Antworte auf Deutsch
- Fachbegriffe (Klassen, Methoden, Pattern-Namen) auf Englisch
- Kurze Erklärungen, Code-first: erst Code zeigen, dann erklären
- Entscheidungen immer kurz begründen

TECHNOLOGIE-STACK
- C# / .NET 9, Nullable enabled
- ASP.NET Core Minimal API
- MS SQL Server mit T-SQL
- Dynamische SQL: IMMER sp_executesql, NIEMALS String-Konkatenation
- Fehlerbehandlung SQL: TRY/CATCH mit RAISERROR
- Tests: xUnit mit AAA-Pattern

C# KONVENTIONEN
- Records für DTOs (immutable)
- Keine public Fields
- Result<T> Pattern für Fehlerbehandlung in Services
- Keine Legacy-Patterns (.NET Framework Style)

KONTEXT-NUTZUNG
- Das DB-Schema findest du in der Wissensbasis (schema.md)
- Die Projektstruktur in architecture.md
- Coding-Konventionen in conventions.md
- Nutze diese Dokumente aktiv bei deinen Antworten

CHAT-VERHALTEN
- Wenn ein Chat sehr lang wird: weise mich darauf hin, 
  dass ich einen Transition-Snapshot erstellen und einen neuen Chat starten sollte
- Frag nach, wenn dir wichtiger Kontext fehlt
```

### Schritt 3: Wissensbasis befüllen

Dokumente hochladen (in dieser Reihenfolge):
1. `docs/schema.md`
2. `docs/architecture.md`
3. `docs/conventions.md`

**Wissensbasis-Verwaltung:**
```
Bei Änderungen am Schema:
  → schema.md im Repo aktualisieren + committen
  → Alte schema.md aus KB löschen
  → Neue hochladen

Faustregel: KB-Dokumente ≤ 3 Dokumente, je ≤ 300 Zeilen
```

---

## 4. Aufgaben & Workflows – Alle Einsatzzwecke

### 4.1 Feature-Entwicklung

**Chat-Titel:** `Feature: [Name]`

**Einstiegs-Prompt:**
```
Ich entwickle ein neues Feature für myproject.

Feature: [NAME]
Anforderungen:
- [Anforderung 1]
- [Anforderung 2]

Betroffene Bereiche: [z.B. "API-Endpoint, Service-Layer, DB-Query"]

Relevanter bestehender Code:
[CODE EINFÜGEN oder: "noch nichts vorhanden"]

Starte mit: Welche Schritte sind notwendig? 
Dann gehen wir sie gemeinsam durch.
```

**Ablauf im Chat:**
```
1. Planung/Architektur des Features
2. API-Endpoint / Controller
3. Service-Logik
4. Repository / SQL-Query
5. Tests
→ Bei >15 Nachrichten: Transition-Snapshot + neuer Chat
```

---

### 4.2 Fehlerbehebung / Debugging

**Chat-Titel:** `Bugfix: [Kurzbeschreibung]`

**Einstiegs-Prompt:**
```
Ich habe einen Bug in myproject.

Fehlerbeschreibung:
[WAS passiert]

Erwartetes Verhalten:
[WAS sollte passieren]

Fehlermeldung / Stack Trace:
[FEHLERMELDUNG EINFÜGEN]

Betroffener Code:
[RELEVANTE DATEIEN / METHODEN EINFÜGEN]

Bitte analysiere: Was ist die wahrscheinlichste Ursache?
Gib mir 2–3 Hypothesen mit Begründung, dann lösen wir es systematisch.
```

---

### 4.3 Code Review

**Chat-Titel:** `Review: [Feature/Datei]`

**Einstiegs-Prompt:**
```
Bitte reviewe den folgenden C#-Code auf:
1. .NET Best Practices und moderne Patterns
2. Potenzielle Null-Reference Issues (Nullable)
3. SQL-Sicherheit (Injection-Risiken)
4. Performance-Probleme
5. Verstöße gegen unsere Konventionen (siehe Wissensbasis)

Code:
[CODE EINFÜGEN]

Format der Antwort:
- Kritische Probleme (MUST fix)
- Verbesserungsvorschläge (SHOULD fix)
- Kleinigkeiten (NICE to have)
- Überarbeiteter Code am Ende
```

---

### 4.4 SQL-Entwicklung

**Chat-Titel:** `SQL: [Zweck der Query]`

**Einstiegs-Prompt:**
```
Ich brauche eine T-SQL Query für MS SQL Server.

Zweck: [WAS soll die Query tun]
Betroffene Tabellen: [aus schema.md – Tabellennamen]
Besondere Anforderungen:
- [z.B. "muss performant sein für 1M+ Zeilen"]
- [z.B. "dynamische Filter je nach Parameter"]
- [z.B. "als gespeicherte Prozedur"]

Kontext:
[Relevante Teile aus schema.md einfügen ODER: "ist in der Wissensbasis"]

Erstelle:
1. Die Query mit sp_executesql (bei dynamischem SQL)
2. Erklärung der Logik
3. Potenzielle Performance-Hinweise
```

**Erweiterung für dynamisches Query-Framework:**
```
Ergänzung zum obigen Prompt für unser Framework:

Nutze das bewährte dynamische Query-Framework-Pattern:
- Zentralisierte Konfiguration via Temp-Tabellen
- Cursor-basierte Iteration über Query-Targets  
- sp_executesql mit Parametern (NIEMALS String-Konkat)
- Per-Query Fehlerbehandlung mit TRY/CATCH
- Toggle-Flags für Ein-/Ausschalten einzelner Queries
- Dokumentation der Parameter und Ausgabe auf Deutsch
```

---

### 4.5 Analyse & Visualisierung

**Chat-Titel:** `Analyse: [Thema]`

**Prompt für Code-Analyse:**
```
Analysiere folgende C#-Codebasis / diesen Code-Abschnitt:

[CODE EINFÜGEN]

Ich möchte verstehen:
- Wie funktioniert dieser Code im Überblick?
- Welche Abhängigkeiten bestehen?
- Welche potenziellen Probleme gibt es?
- Wo würdest du als erstes optimieren?

Erstelle wenn sinnvoll:
- Ein Mermaid-Diagramm der Klassen/Beziehungen
- Ein Sequenzdiagramm des Datenflusses
```

**Prompt für DB-Analyse:**
```
Analysiere folgende SQL-Query / Stored Procedure:

[SQL EINFÜGEN]

Fragen:
1. Was macht diese Query genau (Step by Step)?
2. Performance: Wo sind potenzielle Bottlenecks?
3. Sicherheit: SQL-Injection-Risiken?
4. Verbesserungsvorschläge mit überarbeitetem Code
```

---

### 4.6 Dokumentation erstellen

**Chat-Titel:** `Doku: [Was dokumentiert wird]`

**Prompt für API-Dokumentation:**
```
Erstelle eine Markdown-Dokumentation für folgende API-Endpoints:

[CODE EINFÜGEN]

Format pro Endpoint:
### [METHOD] /pfad/endpoint
**Zweck:** [1 Satz]
**Parameter:** [Tabelle]
**Request Body:** [Beispiel-JSON falls POST/PUT]
**Response:** [Beispiel-JSON]
**Fehler:** [Mögliche HTTP-Statuscodes]

Sprache: Deutsch
Stil: Entwickler-Dokumentation, präzise und knapp
```

**Prompt für Code-Kommentare:**
```
Füge XML-Dokumentationskommentare (/// ) zu folgenden C#-Methoden hinzu.
Stil: präzise, informativ, keine trivialen Kommentare wie "// setzt den Wert".

[CODE EINFÜGEN]

Anforderungen:
- <summary> in Deutsch
- <param> für alle Parameter
- <returns> für Rückgabewerte
- <exception> für dokumentierte Ausnahmen
- Kommentare sollen den WARUM erklären, nicht das WAS
```

---

### 4.7 Berichterstellung / Reports

**Chat-Titel:** `Report: [Thema]`

**Prompt:**
```
Erstelle einen [TYP: technischen / Management-] Bericht zu:
[THEMA]

Grundlage:
[DATEN / CODE / ANALYSE EINFÜGEN]

Zielgruppe: [z.B. "Entwicklungsteam" / "Projektleitung"]

Enthaltene Abschnitte:
- Zusammenfassung (Executive Summary, 3–5 Sätze)
- Technischer Hintergrund
- Analyse / Befunde
- Empfehlungen (priorisiert)
- Nächste Schritte

Format: Markdown, auf Deutsch
Ton: [professionell/sachlich / direkt/technisch]
```

---

### 4.8 Refactoring

**Chat-Titel:** `Refactoring: [Klasse/Modul]`

**Prompt:**
```
Ich möchte folgenden C#-Code refactoren.

Aktueller Code:
[CODE EINFÜGEN]

Ziele des Refactorings:
- [z.B. "Lesbarkeit verbessern"]
- [z.B. "SOLID-Prinzipien einhalten"]
- [z.B. "Dependency Injection vorbereiten"]
- [z.B. "Testbarkeit erhöhen"]

Constraints:
- Keine breaking changes an der öffentlichen API
- .NET 9 / Nullable enabled
- [weitere Constraints]

Vorgehen:
1. Analysiere den Code: Was sind die Hauptprobleme?
2. Schlage Refactoring-Schritte vor (nicht alles auf einmal)
3. Beginne mit dem wichtigsten Schritt
```

---

## 5. Chat-Management: Kontext-Erhaltung ohne Claude Code

Da kein Claude Code verfügbar ist, ersetzt dieses manuelle System das CLAUDE.md:

### Checkliste pro Chat-Start

```
☐ Bin ich im richtigen Projekt? (claude.ai/project/myproject)
☐ Hat der Chat einen klaren Titel und Fokus?
☐ Gibt es einen Transition-Snapshot vom letzten Chat?
   → Wenn ja: als ersten Satz einfügen
☐ Brauche ich spezifischen Code-Kontext?
   → Relevante Dateien als Code-Block einfügen
```

### Checkliste Chat-Ende

```
☐ Transition-Snapshot generieren (Prompt aus Abschnitt 2.4)
☐ Snapshot in docs/context-snapshots/ speichern (optional, für Nachvollziehbarkeit)
☐ Generierte Code-Snippets ins Repo übernommen?
☐ Wissensbasis aktualisierungsbedürftig? (schema.md etc.)
```

### Wann einen neuen Chat starten?

```
→ Thema / Aufgabe abgeschlossen
→ Mehr als ~15 Nachrichten (Kontext wird teuer)
→ Claude gibt inkonsistente Antworten (Kontext-Overflow)
→ Anderes Feature / anderer Bereich wird angegangen
→ Claude "vergisst" frühere Entscheidungen
```

### Code-Austausch-Workflow (ohne Claude Code)

```
GitHub → claude.ai:
  1. Datei in VS/Rider/Editor öffnen
  2. Relevanten Abschnitt kopieren
  3. Als Code-Block in den Chat einfügen:
     ```csharp
     [CODE]
     ```

claude.ai → GitHub:
  1. Claudes Code aus der Antwort kopieren
  2. In die entsprechende Datei einfügen / ersetzen
  3. Lokal testen
  4. Committen und pushen
```

> **Tipp:** Nicht die gesamte Datei einfügen – nur den relevanten Abschnitt (Klasse, Methode, Query). Spart Token und hält den Kontext fokussiert.

---

## 6. Wissensbasis-Wartung

### Wann KB-Dokumente aktualisieren?

| Ereignis | Aktion |
|----------|--------|
| Neue Tabelle hinzugefügt | schema.md aktualisieren + neu hochladen |
| Neue Schicht / neues Projekt hinzugefügt | architecture.md aktualisieren |
| Neues Coding-Pattern beschlossen | conventions.md ergänzen |
| Claude gibt falsche Antworten basierend auf KB | Dokument prüfen und korrigieren |

### KB-Update-Prompt

```
Das folgende Dokument in meiner Wissensbasis (schema.md) 
ist veraltet. Hier ist die aktuelle Version:

[NEUES DOKUMENT EINFÜGEN]

Bitte bestätige, dass du die neue Version verwendest.
[Dann: alte Datei aus KB löschen, neue hochladen]
```

---

## 7. Vollständige Cheat-Sheet

```
╔══════════════════════════════════════════════════════════════╗
║              MYPROJECT – TÄGLICHE REFERENZ                   ║
╠══════════════════════════════════╦═══════════════════════════╣
║  AUFGABE                         ║  WO / WIE                 ║
╠══════════════════════════════════╬═══════════════════════════╣
║  Neues Feature planen            ║  Projekt-Chat, neu        ║
║  Feature implementieren          ║  Projekt-Chat, Code paste ║
║  Bug analysieren + fixen         ║  Projekt-Chat, neu        ║
║  Code Review                     ║  Projekt-Chat, neu        ║
║  SQL-Query entwickeln            ║  Projekt-Chat, neu        ║
║  Dokumentation schreiben         ║  Projekt-Chat, neu        ║
║  Refactoring                     ║  Projekt-Chat, neu        ║
║  Einmalige Frage (kein Projekt)  ║  Freier Chat              ║
╠══════════════════════════════════╬═══════════════════════════╣
║  Chat wird lang (>15 Msg)        ║  Snapshot → neuer Chat    ║
║  Thema wechselt                  ║  Neuer Chat               ║
║  Schema ändert sich              ║  schema.md update + Upload║
║  Claude gibt falsche KB-Antwort  ║  KB-Dokument prüfen       ║
╠══════════════════════════════════╬═══════════════════════════╣
║  Code: GitHub → Claude           ║  Ausschnitt Copy/Paste    ║
║  Code: Claude → GitHub           ║  Copy/Paste → Datei       ║
╚══════════════════════════════════╩═══════════════════════════╝
```

---

## 8. Einmal-Einrichtung: Gesamtreihenfolge

```
Tag 1 – Setup:

[ ] 1. GitHub-Repo anlegen / Struktur erstellen (docs/ Ordner)
[ ] 2. Freier Chat: schema.md generieren (Prompt 2.1)
[ ] 3. Freier Chat: architecture.md generieren (Prompt 2.2)
[ ] 4. Freier Chat: conventions.md generieren (Prompt 2.3)
[ ] 5. Alle drei Dateien committen: git commit docs/
[ ] 6. claude.ai Projekt anlegen (Abschnitt 3)
[ ] 7. Projektanweisungen einfügen (kompletter Text aus 3.2)
[ ] 8. schema.md, architecture.md, conventions.md hochladen
[ ] 9. Test-Chat im Projekt: "Was weißt du über mein Datenbankschema?"
       → Claude sollte schema.md korrekt referenzieren

Ab Tag 2 – Arbeiten:

[ ] Immer im Projekt-Chat starten
[ ] Pro Chat eine fokussierte Aufgabe
[ ] Passenden Einstiegs-Prompt aus Abschnitt 4 verwenden
[ ] Am Chat-Ende Transition-Snapshot generieren
[ ] Code manuell ins Repo übernehmen
```

---

*Quellen: Anthropic Dokumentation (support.claude.com), claude.ai Help Center, Stand März 2026*
