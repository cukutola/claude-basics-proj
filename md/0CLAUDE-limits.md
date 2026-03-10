# CLAUDE-limits.md – Limits, Effizienz & CLAUDE.md

> **Stand:** März 2026 · **Quellen:** Offizielle Anthropic-Docs, support.claude.com, community-verifizierte Messungen

---

## 1. Grundlage der Limit-Arithmetik: Tokens, nicht Nachrichten

### Was ist ein Token?
Das gesamte Limit-System basiert auf **Tokens**, nicht auf der Anzahl sichtbarer Nachrichten:

```
~4 Zeichen   ≈  1 Token
~750 Wörter  ≈  1.000 Tokens
1 Seite Text ≈  500–800 Tokens
```

**Was zählt alles als Token-Verbrauch:**

| Quelle | Tokens (Beispiel) |
|--------|-------------------|
| Deine Frage (kurz) | ~200 |
| Claudes Antwort (lang) | 2.000–8.000 |
| Gesamter Chat-Verlauf (wird jedes Mal neu mitgesendet) | kumulierend! |
| Projekt-Wissensbasis (RAG-Abruf) | je nach Inhalt |
| Hochgeladene Datei (mittelgroß) | 10.000–50.000 |
| Systemanweisungen / Projektanweisungen | 500–2.000 |
| 5 mittlere Code-Dateien in Claude Code | ~30.000+ |

> ⚠️ **Kritischer Effekt:** Nachricht Nr. 50 in einem langen Chat kostet massiv mehr als Nachricht Nr. 1 – weil der gesamte bisherige Verlauf jedes Mal mitübertragen wird.

### Das Kontextfenster
Das Kontextfenster ist die maximale Menge an Text, die Claude in einer Anfrage gleichzeitig verarbeiten kann:

- **Standard:** 200.000 Tokens (~150.000 Wörter)
- **Beta (Opus/Sonnet 4.x):** 1.000.000 Tokens – aber mit erhöhten API-Kosten ab >200K Input-Tokens

---

## 2. Wie werden Limits berechnet? Das Dual-Layer-System

Seit **August 2025** gibt es zwei überlagernde Limit-Schichten:

### Schicht 1: Rollendes 5-Stunden-Fenster
Das am häufigsten misverstandene Konzept – es ist **kein festes Reset**, sondern ein gleitendes Fenster:

```
10:00 Uhr – 10 Nachrichten gesendet  → Zähler: 10
11:00 Uhr – 15 Nachrichten gesendet  → Zähler: 25
12:00 Uhr – 10 Nachrichten gesendet  → Zähler: 35
13:00 Uhr –  5 Nachrichten gesendet  → Zähler: 40
15:00 Uhr – Die 10:00-Nachrichten verfallen → Zähler: 30
```

**Das Fenster läuft kontinuierlich.** Älteste Nachrichten „verfallen" nach 5h und geben Kapazität frei – es gibt keinen harten Reset-Zeitpunkt.

### Schicht 2: Wöchentliches Kontingent (ab Aug. 2025)
Zusätzlich zur 5h-Schranke gibt es ein Wochenlimit, das alle 7 Tage zurückgesetzt wird. Betroffen sind laut Anthropic weniger als 5% der Nutzer – aber intensive Entwickler (Claude Code + agentic tasks) gehören dazu.

---

## 3. Limit-Typen und Pläne im Überblick

### 3.1 Claude.ai Pläne (Web-Interface + App)

| Plan | Preis/Monat | Nachrichten / 5h (Sonnet) | Nachrichten / 5h (Opus) | Besonderheiten |
|------|-------------|--------------------------|------------------------|----------------|
| **Free** | $0 | ~40 kurze/Tag (limitiert) | – | Kein Claude Code, kein RAG |
| **Pro** | ~$20 | ~45 (token-abhängig) | ~45 | Voller Zugang, kein Extra-Kauf |
| **Max 5×** | $100 | ~225 | ~225 | 5× Pro, Extra-Usage kaufbar |
| **Max 20×** | $200 | ~900 | ~900 | Für Heavy-User/Entwickler |

**Wichtig:** "Nachrichten" sind **Token-gewichtet**. Eine lange Opus-Konversation mit Datei-Upload entspricht 5–10 kurzen Sonnet-Nachrichten im Limit-Verbrauch.

**Approximative Token-Limits pro 5h-Fenster:**
- Pro: ~44.000 Tokens
- Max 5×: ~88.000 Tokens
- Max 20×: ~220.000 Tokens

### 3.2 Claude.ai vs. Claude Code: Gemeinsamer Pool!

```
⚠️ KRITISCH: Claude.ai und Claude Code teilen dasselbe Kontingent.
30 Nachrichten auf claude.ai = 30 weniger für Claude Code.
```

Schwere Claude Code Agentic-Tasks verbrauchen das Limit **5–10× schneller** als einfache Chats.

### 3.3 API (separat, pay-as-you-go)

Die API hat eigene Limits – unabhängig vom claude.ai-Abo:

| Metrik | Einheit |
|--------|---------|
| RPM (Requests per Minute) | Je nach Tier |
| ITPM (Input Tokens per Minute) | Je nach Tier & Modell |
| OTPM (Output Tokens per Minute) | Je nach Tier |

**Wichtig:** Die API nutzt das **Token-Bucket-Algorithmus** – Kapazität wird kontinuierlich aufgefüllt, kein festes Reset-Intervall.

> ⚠️ **API-Key-Falle (bekannt aus Setup):** Wenn `ANTHROPIC_API_KEY` als Umgebungsvariable gesetzt ist, werden Anfragen direkt über die API abgerechnet (pay-per-token) – das Pro-Abo auf claude.ai wird vollständig umgangen!

---

## 4. Kosteneffiziente Nutzung: Technischer Hintergrund

### 4.1 Was Tokens „frisst" – die versteckten Kostentreiber

**1. Kumulierender Chat-Verlauf**
Jede neue Nachricht in einem bestehenden Chat überträgt den gesamten bisherigen Verlauf. Ein Chat mit 40 langen Nachrichten kostet für Nachricht 41 vielleicht 10× mehr als Nachricht 1.

**2. Projekt-Wissensbasis**
Alles, was in der Wissensbasis eines Projekts liegt, wird per RAG bei Bedarf abgerufen. Zu volle Wissensbasis = schlechteres Retrieval + mehr Token-Overhead.

**3. Datei-Uploads**
Jede hochgeladene Datei verbleibt im Kontext des Chats. Eine 50-seitige PDF = ~25.000 Tokens, die bei jeder Folgenachricht mitlaufen.

**4. Systemanweisungen & CLAUDE.md**
Werden bei jeder Sitzung in den Kontext geladen – je länger, desto teurer.

**5. Output-Tokens sind teurer**
Bei der API kosten Output-Tokens ca. 8× mehr als Input-Tokens (Generierung ist rechenintensiver als Lesen).

### 4.2 Prompt-Caching – der Game-Changer für Effizienz

Bei langen, wiederverwendeten Inhalten (Systemanweisungen, Dokumente) kann Caching bis zu **90% der Input-Kosten** sparen:

```
Normal:      $3,00 / MTok Input
Cache-Write: $3,75 / MTok (1,25×)  ← einmalig beim Schreiben
Cache-Read:  $0,30 / MTok (0,10×)  ← bei Wiederverwendung
```

> Relevant für API-Nutzer und Claude Code – weniger für claude.ai direkt.

---

## 5. Kosteneffiziente Nutzungs-Konzepte

### 5.1 Chats strategisch splitten

**Das Grundprinzip:** Ein Chat = Eine Aufgabe / Ein Thema

```
✗ INEFFIZIENT:
Langer Mega-Chat über Wochen → immer schwerer, immer teurer

✓ EFFIZIENT:
Chat 1: Datenbankschema designen     → Abschluss, neuer Chat
Chat 2: Query für Report schreiben   → Abschluss, neuer Chat
Chat 3: Performance-Analyse          → Abschluss, neuer Chat
```

**Wann einen neuen Chat starten:**
- Thema/Aufgabe wechselt
- Claude fängt an, frühere Entscheidungen zu "vergessen" oder zu widersprechen
- Der Chat läuft seit mehreren Stunden mit vielen Nachrichten
- Vor einer komplett neuen Problemstellung

**Transition-Technik:** Am Ende eines Chats kurz zusammenfassen lassen und die Zusammenfassung in den nächsten Chat als erste Nachricht einfügen.

### 5.2 Modell-Wahl bewusst einsetzen

```
Sonnet 4.6  →  80-90% aller Aufgaben, deutlich weniger Token-Verbrauch
Opus 4.6    →  Nur für echte Komplexität: Architekturentscheidungen, 
               schwieriges Debugging, mehrstufige Analysen
Haiku 4.5   →  Schnelle Lookups, simple Transformationen, API-Automatisierung
```

> Pro-Tipp: In Claude Code `/model` nutzen, um je nach Aufgabe zu wechseln.

### 5.3 Projekte als Effizienz-Werkzeug

**Was Projekte einsparen:**
- Kein wiederholtes Einfügen von Kontext (Schema, Guidelines, etc.)
- Einmalige Projektanweisungen gelten für alle Chats
- RAG holt nur das Relevante – nicht alles gleichzeitig

**Was Projekte NICHT tun:**
- Chats innerhalb des Projekts teilen keinen Kontext untereinander (nur Wissensbasis + Anweisungen)
- Kein Ersatz für Chat-Splitting

**Optimale Projekt-Wissensbasis:**
```
✓ Kernschema (DB, API-Struktur)
✓ Projekt-spezifische Konventionen
✓ Wichtige Referenzdokumente (knapp)
✗ Komplette Codebase (zu groß, zu teuer)
✗ Veraltete oder selten gebrauchte Dokumente
```

### 5.4 Antwort-Länge kontrollieren

```
✓ "Antworte in maximal 3 Absätzen"
✓ "Nur den Code, keine Erklärung"
✓ "Bullet-Point-Format, max 10 Punkte"
✓ "Zeige nur die geänderten Zeilen, nicht die komplette Datei"
```

### 5.5 Dateien effizient handhaben

```
✓ Relevanten Ausschnitt kopieren statt ganze Datei hochladen
✓ Bei großen Dateien: nur den betroffenen Bereich einfügen
✓ Datei nicht mehrfach hochladen – Claude erinnert sie im Chat-Verlauf
✗ Nie 5 große PDFs gleichzeitig hochladen wenn nur eine gebraucht wird
```

### 5.6 Zeitplanung für das 5h-Fenster

Da das Fenster gleitend ist, kann man gezielt planen:
- Intensive Arbeitsphasen so legen, dass man das Fenster voll ausnutzt
- Nach einer Pause (>5h) ohne Claude verfallen alte Nachrichten → frisches Kontingent
- Bei Claude Code: `--max-turns N` begrenzt agentic loops und verhindert unkontrollierten Verbrauch

---

## 6. CLAUDE.md – Alles über die Konfigurations-Datei

### 6.1 Was ist CLAUDE.md?

`CLAUDE.md` ist eine Markdown-Datei, die **Claude Code** automatisch zu Beginn jeder Sitzung in den Kontext lädt. Sie funktioniert als persistentes Projektgedächtnis – ein konfiguriertes Onboarding-Dokument für den KI-Agenten.

**Analogie:** CLAUDE.md ist der „Briefing-Zettel", den ein neuer Mitarbeiter bei jedem Arbeitstag automatisch liest – damit du nicht jedes Mal von vorne erklären musst.

> **Wichtige Unterscheidung:**
> - `CLAUDE.md` = für **Claude Code** (Terminal/IDE)
> - **Projektanweisungen** auf claude.ai = für den **Web-Chat**
> Beide sind ähnlich in der Idee, aber technisch getrennte Systeme.

### 6.2 Wo liegt CLAUDE.md? – Die Hierarchie

Claude Code unterstützt **mehrere CLAUDE.md-Dateien** in einer Hierarchie – alle werden kombiniert geladen:

```
~/.claude/CLAUDE.md          ← Globale Präferenzen (gilt für alle Projekte)
~/projects/my-app/CLAUDE.md  ← Projekt-Root (Hauptdatei, ins Repo committen)
~/projects/my-app/src/CLAUDE.md         ← Modulspezifisch (Frontend)
~/projects/my-app/src/backend/CLAUDE.md ← Noch spezifischer (Backend)
CLAUDE.local.md              ← Persönliche Overrides (in .gitignore!)
```

**Ladeverhalten:** Ancestor-Dateien werden zuerst geladen, dann absteigende Verzeichnisse. Bei Konflikten gewinnt die spezifischere Datei.

**Globale CLAUDE.md** (`~/.claude/CLAUDE.md`) – ideal für:
```markdown
# Global Preferences
- Sprache: Deutsch bevorzugt
- Editor: Neovim-kompatible Ausgabe
- Keine unnötigen Erklärungen bei einfachen Aufgaben
- SQL: MS SQL Server Syntax bevorzugen
```

### 6.3 Inhalt: Was gehört rein? Was nicht?

**Checkliste: Was rein gehört**

```markdown
## ✓ Build & Test Commands
npm run dev / npm test / dotnet build

## ✓ Architektur-Überblick (knapp)
Welche Hauptverzeichnisse, welche Rolle

## ✓ Projekt-spezifischer Jargon & Terminologie
Domain-Begriffe, die nicht selbsterklärend sind

## ✓ Kritische Constraints (MUST/NEVER)
"NEVER commit .env files"
"Database migrations ONLY via Flyway"

## ✓ MCP-Server-Hinweise
Welche MCP-Tools verfügbar sind und wann nutzen

## ✓ Verweise auf tiefere Dokumente (Progressive Disclosure)
"For auth flow, see @docs/auth.md"
```

**Checkliste: Was NICHT rein gehört**

```markdown
## ✗ Code-Style-Details
→ Nutze Linter/Formatter (ESLint, Prettier, SQL Formatter)
→ LLMs sind langsam und teuer für Aufgaben, die Tools instant erledigen

## ✗ Code-Snippets / Beispiele
→ Veralten schnell, fressen Token-Budget
→ Stattdessen: Datei-Referenz "@src/utils/example.ts"

## ✗ Offensichtliches
→ "The /src folder contains source code" – unnötig

## ✗ Aufgaben-spezifischer Kontext
→ Gehört in den Chat, nicht in CLAUDE.md
→ CLAUDE.md ist für universell gültige Dinge

## ✗ Secrets, API-Keys, Credentials
→ Nie in CLAUDE.md, besonders nicht wenn ins Repo committet
```

### 6.4 Länge und Struktur

**Zielgröße:** Unter 200 Zeilen – idealer Richtwerk: 60–100 Zeilen

```
< 100 Zeilen  → Optimal
100–200 Zeilen → Akzeptabel
200–300 Zeilen → Grenzwertig
> 300 Zeilen  → Claude beginnt, Teile zu ignorieren
```

**Empfohlene Struktur:**

```markdown
# CLAUDE.md

## Project Context (WHY & WHAT)
Kurze Beschreibung: Was ist das Projekt, was ist das Ziel.

## Commands (HOW – only essentials)
```bash
npm run dev      # Dev-Server starten
npm test         # Tests ausführen
npm run migrate  # DB-Migration
```

## Architecture (WHAT – directories & roles)
/src/routes    → API Endpoints
/src/models    → DB-Modelle (Prisma)
/src/services  → Business Logic

## Conventions (only non-obvious ones)
- API-Antworten immer: { success, data, error }
- Neue Migrations NEVER manuell editieren
- IMMER tests vor commit

## Terminology (domain-specific)
- "Wrap": Jahresrückblick-Feature für User
- "Slot": Buchungszeitraum in der Kalenderlogik

## Reference Documents (Progressive Disclosure)
- Auth-Flow: @docs/authentication.md (lesen bei Auth-Änderungen)
- DB-Schema: @docs/schema.md (lesen bei Migrationen)
```

### 6.5 Progressive Disclosure – das Schlüsselkonzept

Statt alles direkt einzubetten, zeige Claude *wo* es Informationen findet:

```markdown
## ✗ Falsch (alles eingebettet – teuer und veraltet schnell):
### Auth Flow
1. User sendet credentials
2. Server prüft gegen DB
3. JWT wird generiert mit folgendem Schema: {...}
...50 Zeilen Auth-Dokumentation...

## ✓ Richtig (Pointer statt Kopie):
For authentication changes, read @docs/auth.md first.
It describes JWT schema, token refresh logic, and OAuth flows.
```

**Warum Pointer, keine Kopien:**
- Dokument bleibt die Single Source of Truth
- CLAUDE.md veraltet nicht
- Token-Budget wird geschont

### 6.6 Initialerstellung mit `/init`

In Claude Code:

```bash
# Im Projektverzeichnis:
/init
```

Claude analysiert die Codebase (package.json, Configs, Ordnerstruktur) und generiert eine Starter-CLAUDE.md. Diese dann:
1. Offensichtliches löschen
2. Fehlende Domain-Begriffe ergänzen
3. Kürzen, kürzen, kürzen

### 6.7 CLAUDE.md für Nicht-Code-Projekte (claude.ai Web)

Im Web-Interface gibt es kein CLAUDE.md im technischen Sinne – aber das Konzept lässt sich übertragen:

**Option A: Projektanweisungen auf claude.ai**
```
Projektanweisungen → direkte Entsprechung zu CLAUDE.md für Web-Chats
```

**Option B: CLAUDE.md als erste Nachricht im Chat**
Die Inhalte der CLAUDE.md als Kontext zu Beginn einfügen – manuell, aber effektiv.

**Option C: In die Wissensbasis hochladen**
Eine CLAUDE.md-Datei oder ein äquivalentes Kontext-Dokument in die Projekt-Wissensbasis hochladen. Claude greift per RAG darauf zu.

### 6.8 CLAUDE.md warten und optimieren

**Wann aktualisieren:**
- Nach Code-Reviews, wenn Konventionen verletzt wurden (→ in CLAUDE.md dokumentieren)
- Wenn Claude wiederholt dieselbe Korrektur bekommt (→ einmal rein, nie wieder erklären)
- Wenn neue Tools/MCP-Server hinzukommen
- Wenn sich die Architektur ändert

**Optimierungs-Review (monatlich empfohlen):**
```
Frage an Claude: "Review this CLAUDE.md. 
Identify: redundant instructions, outdated content, 
missing context. Suggest a shorter, improved version."
```

**Wichtige Operatoren:**
```markdown
IMPORTANT: oder MUST   → Erhöht Aufmerksamkeit (sparsam einsetzen!)
NEVER                  → Absolute Verbote (immer Alternative angeben!)
"If you encounter X, read @path/to/docs.md" → Conditional loading
```

> ⚠️ Wenn alles IMPORTANT ist, ist nichts important. Maximal 2–3 kritische Highlights pro Datei.

### 6.9 Mein CLAUDE.md – Starter-Vorlage für MS SQL / Neovim

Basierend auf dem bekannten Setup (MS SQL Server, Neovim, `coder/claudecode.nvim`):

```markdown
# CLAUDE.md – SQL Development

## Context
MS SQL Server T-SQL development. German-language responses preferred.
Dynamic query framework with centralized config and multi-table iteration.

## Commands
sqlcmd -S <server> -d <db> -i query.sql   # Execute SQL
-- Validation: always test with ROLLBACK before COMMIT

## Conventions
- Language: T-SQL (MS SQL Server syntax only, no ANSI deviations)
- Error handling: TRY/CATCH with custom RAISERROR messages
- Dynamic SQL: sp_executesql with parameters (never string concat)
- NEVER direct string concatenation in dynamic SQL (SQL injection risk)

## Architecture
/queries      → Einzelne Query-Dateien
/framework    → Zentralisierte Konfiguration, Iterator-Logik
/docs         → Schema-Referenz

## Reference
- Framework-Logik: @framework/README.md
- DB-Schema: @docs/schema.md (lesen bei Joins oder neuen Tabellen)
```

---

## 7. Kurzreferenz: Limit-Checkliste

```
☐ Chat-Verlauf kumuliert → Neue Chats für neue Aufgaben starten
☐ Modell passend wählen → Sonnet für 80% der Aufgaben
☐ Claude.ai + Claude Code teilen dasselbe Kontingent
☐ API-Key ≠ Pro-Abo → Separate Abrechnung!
☐ CLAUDE.md unter 200 Zeilen halten
☐ Progressive Disclosure: Pointer statt Kopien
☐ Dateien nicht unnötig re-uploaden
☐ Antwortlänge explizit steuern
☐ /compact in Claude Code bei vollem Kontext (vor 50%-Auslastung)
☐ CLAUDE.md regelmäßig reviewen und kürzen
```

---

*Quellen: Anthropic Help Center (support.claude.com), Anthropic Claude Code Docs (code.claude.com/docs), community-verifizierte Messungen (Stand März 2026)*
