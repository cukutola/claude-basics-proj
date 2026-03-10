# CLAUDE.md – Referenzdokument

> **Stand:** März 2026 · **Erstellt mit:** claude.ai (Sonnet 4.6)

---

## 1. Grundlagen

### Was ist Claude?
Claude ist ein Large Language Model (LLM) von Anthropic, das auf dem Prinzip des *Constitutional AI* basiert – ein eingebautes ethisches Regelwerk, das Hilfsbereitschaft und Sicherheit von Grund auf verankert.

**Verfügbare Modelle (Stand März 2026):**
| Modell | Stärke | Einsatz |
|--------|--------|---------|
| Claude Opus 4.6 | Tiefes Denken, komplexe Analysen | Schwierige Aufgaben, Recherche |
| Claude Sonnet 4.6 | Ausgewogene Leistung/Geschwindigkeit | Alltägliche Aufgaben, Standard |
| Claude Haiku 4.5 | Schnell & kosteneffizient | Einfache Aufgaben, Automatisierung |

### Wie funktioniert Claude?
- **Kontextfenster:** Claude verarbeitet den gesamten Gesprächsverlauf jedes Mal neu – es gibt kein intrinsisches „Gedächtnis" zwischen Sitzungen (außer durch explizite Speicher-Features)
- **Token-basiert:** Input und Output werden in Tokens gemessen; das Kontextfenster ist auf eine bestimmte Länge begrenzt (je nach Modell)
- **Kein Internetzugang (Standard):** Ohne aktiviertes Web-Search-Tool basiert Claude auf Trainingsdaten (Wissens-Cutoff: August 2025)

### Vorteile gegenüber anderen KI-Assistenten
- Starke Reasoning-Fähigkeiten bei komplexen Aufgaben
- Exzellent für langen, strukturierten Text und Code
- Projektbasiertes Gedächtnis (keine globale Datenvermischung)
- Klar auf Datenschutz und Sicherheit ausgerichtet

---

## 2. Persistenz: Chats, Sitzungen, Projekte

### 2.1 Einzelne Chats (kein Projekt)
- Chats werden **in der linken Seitenleiste gespeichert** und bleiben dauerhaft abrufbar
- Innerhalb eines Chats ist der gesamte Verlauf Kontext – aber **zwischen Chats gibt es keinen automatischen Transfer**
- Chats können **in Projekte verschoben** werden (nachträglich, per Dropdown-Menü im Chat)

### 2.2 Memory-Feature (Chat-übergreifend)
- Claude kann **aus vergangenen Gesprächen lernen** und relevante Informationen in neuen Chats referenzieren
- Memory war zunächst nur für bezahlte Pläne (ab August 2025), ist inzwischen auch für Free-Nutzer ausgerollt
- Nutzer behalten **volle Kontrolle**: Memory pausieren, löschen oder exportieren jederzeit möglich
- **Memory-Import/-Export:** Erinnerungen können zwischen KI-Diensten übertragen werden (experimentell)

### 2.3 Projekte – Die zentrale Persistenz-Lösung
Projekte sind **selbstständige Arbeitsbereiche** mit eigenem Chat-Verlauf und eigener Wissensbasis.

**Kernmerkmale:**
- **Wissensbasis:** Dokumente, PDFs, Code, Texte hochladen → stehen in *allen* Projekt-Chats als Kontext zur Verfügung
- **Projektanweisungen:** Einmalig definieren (Ton, Format, Rolle) → gilt automatisch für alle Chats im Projekt
- **Separates Memory:** Jedes Projekt hat eine eigene Memory-Zusammenfassung, getrennt von anderen Projekten und allgemeinen Chats
- **RAG (Retrieval Augmented Generation):** Bei bezahlten Plänen skaliert die Wissensbasis dynamisch durch RAG

**Wichtig:** Kontext wird *nicht* automatisch zwischen einzelnen Chats innerhalb eines Projekts geteilt – nur die Wissensbasis und die Projektanweisungen sind übergreifend.

### 2.4 Übersicht: Persistenz-Ebenen

| Ebene | Lebensdauer | Scope |
|-------|-------------|-------|
| Einzelner Chat | Dauerhaft gespeichert | Nur innerhalb dieses Chats |
| Projekt-Wissensbasis | Dauerhaft (manuell verwaltet) | Alle Chats im Projekt |
| Projekt-Anweisungen | Dauerhaft (manuell verwaltet) | Alle Chats im Projekt |
| Memory (automatisch) | Bis zur Löschung | Projekt-isoliert oder allgemein |
| Kontext-Fenster | Nur aktive Sitzung | Nur aktueller Chat |

---

## 3. Einsatzmöglichkeiten

### 3.1 Einfache Chats (claude.ai)
**Geeignet für:**
- Schnelle Fragen, Textentwürfe, Code-Snippets
- Einmalige Aufgaben ohne Wiederverwendungsbedarf
- Spontane Recherche (mit Web-Search-Tool)

**Verfügbare Tools im Chat:**
- Web-Suche (aktivierbar)
- Code-Ausführung
- Datei-Upload (PDFs, Bilder, Dokumente)
- Deep Research
- Artifacts (interaktive Ausgaben: React, HTML, SVG, Markdown, etc.)

### 3.2 Projekte – Strukturierte Langzeitarbeit
**Ideal für:**
- Forschungsprojekte mit vielen Quellen
- Langfristige Dokumentenerstellung (Handbücher, Specs, Reports)
- Konsistente Coding-Kontexte (z. B. Datenbankschemas, Konventionen hochladen)
- Kundenprojekte mit spezifischem Wissen

**Beispiel-Setups:**
```
Projekt: "SQL-Entwicklung"
├── Wissensbasis: DB-Schema, Coding-Guidelines, Namenskonventionen
├── Anweisung: "Antworte auf Deutsch, nutze MS SQL Syntax, erkläre Entscheidungen"
└── Chats: Einzelne Aufgaben, Queries, Reviews
```

### 3.3 Team & Enterprise (claude.ai/work)
- **Projekt-Sharing:** Mit bestimmten Mitgliedern oder der ganzen Organisation teilen
- **Berechtigungen:** "Kann nutzen" vs. "Kann bearbeiten"
- **Shared Activity Feed:** Team-Mitglieder können beste Konversationen teilen
- **Admin-Kontrolle:** Memory-Features organisationsweit steuerbar

### 3.4 Claude Code (Terminal)
- Kommandozeilen-Tool für agentisches Coding
- Integration in Neovim via `coder/claudecode.nvim` Plugin
- Eigenständige Sitzungen, kein direkter Chat-Verlauf auf claude.ai

### 3.5 Sonstige Zugänge
| Zugang | Beschreibung |
|--------|--------------|
| API | Für Entwickler, pay-per-token |
| Claude in Chrome | Browser-Agent (Beta) |
| Claude in Excel/PowerPoint | Office-Integration (Beta) |
| Cowork | Desktop-Tool für Datei-/Aufgaben-Automatisierung |
| Mobile App | iOS & Android, inkl. Voice-Mode |

---

## 4. Hinweise, Tipps & Best Practices

### 4.1 Prompting – Effektive Kommunikation

**Positiv formulieren:**
```
✗ "Mach es nicht zu lang"
✓ "Fasse in maximal 3 Sätzen zusammen"
```

**Kontext an den Anfang:**
```
✓ "Du bist ein erfahrener MS SQL Entwickler. 
   Optimiere folgende Query für Performance: [Query]"
```

**Format explizit angeben:**
```
✓ "Antworte als nummerierte Liste"
✓ "Gib mir nur den Code, keine Erklärungen"
✓ "Schreibe auf Deutsch, formell"
```

**Schritt-für-Schritt anleiten:**
```
✓ "Denke zuerst laut nach, dann gib die Antwort"
✓ "Gehe jeden Punkt einzeln durch"
```

### 4.2 Projekte optimal einrichten

1. **Wissensbasis sorgfältig befüllen** – Nur relevante Dokumente hochladen; zu viel = schlechteres Retrieval
2. **Klare Projektanweisungen** – Ton, Sprache, Format, Rolle einmalig definieren spart Zeit
3. **Separate Projekte für separate Kontexte** – Vermeidet Kontextvermischung (besonders wichtig für verschiedene Kunden/Themen)
4. **Chats bei Bedarf archivieren** – Erlaubt Aufräumen ohne Datenverlust

### 4.3 Kontext-Management

- **Lange Chats werden träge:** Bei sehr langen Gesprächen lieber neuen Chat starten + relevanten Kontext kurz zusammenfassen
- **Explizit referenzieren:** "Wie besprochen in meiner letzten Nachricht..." hilft bei langen Verläufen
- **Wissensbasis statt Wiederholung:** Häufig genutzten Kontext (Schemas, Guidelines) ins Projekt hochladen statt jedes Mal einzufügen

### 4.4 Wichtige Hinweise

> ⚠️ **API-Key & Pro-Abo:**  
> Wenn `ANTHROPIC_API_KEY` als Umgebungsvariable gesetzt ist (z. B. für Claude Code), werden API-Aufrufe direkt abgerechnet – das Pro-Abo auf claude.ai wird dabei *umgangen*. Kosten entstehen separat!

> 💡 **Memory-Kontrolle:**  
> Memory kann jederzeit unter Einstellungen → Memory verwaltet werden. Einzelne Erinnerungen lassen sich löschen oder exportieren.

> 🔒 **Datenschutz & Incognito:**  
> Im Incognito-Modus ist das Memory-System deaktiviert – keine Daten werden gespeichert oder für Training genutzt.

### 4.5 Workflow-Empfehlungen

**Für Entwickler:**
- Projekt mit Codebase-Dokumentation und Konventionen anlegen
- Claude Code (Terminal) für interaktives Coding nutzen
- API-Key-Kosten im Blick behalten (nicht mit Pro-Abo verwechseln)

**Für Wissensarbeiter:**
- Projekt pro Themenbereich/Kunde
- Wichtige Referenzdokumente in die Wissensbasis
- Memory-Feature für persönliche Präferenzen nutzen

**Für Teams:**
- Shared Projects für gemeinsame Wissensbasis
- Klare Berechtigungen (use vs. edit) vergeben
- Standardisierte Projektanweisungen für konsistente Outputs

---

*Erstellt auf Basis offizieller Anthropic-Dokumentation (support.claude.com) und aktueller Quellen, Stand März 2026.*
