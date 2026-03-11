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
