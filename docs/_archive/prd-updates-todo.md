# PRD Update TODO List

##  Erledigte Updates (2025-01-21)

### Retry-Logik Vereinheitlichung
- [x] FR23 ersetzt mit neuer Retry-Logik
- [x] Strategy A (SPX) angepasst: RTH-relative Zeiten, executionWindow
- [x] Strategy B (SPY) angepasst: RTH-relative Zeiten, executionWindow  
- [x] ExecutionWindow bei allen Exit-Orders entfernt
- [x] EmergencyClose vereinfacht (nutzt error retry)

##  Abgeschlossene Refactoring-Themen (2025-01-21 Session 2)

### 1.  Market Hours Service Integration
**Erledigt:** Neuer Abschnitt in "Technical Assumptions" hinzugef�gt mit:
- [x] API Integration Details (`reqContractDetails` f�r tradingHours/liquidHours)
- [x] Time Calculation Service Spezifikation mit Python Interface
- [x] Cache-Strategie f�r Market Hours (SQLite Schema)
- [x] DST Handling mit pytz/zoneinfo
- [x] Error Handling und Fallback-Mechanismen
- [x] Integration mit Strategy Execution

### 2.  Epic/Story Struktur - ZWEITES REFACTORING ABGESCHLOSSEN

**Problem erkannt & gel�st:** Struktur passt jetzt zu Meta-Strategy Model & Trading State Model

**Finale Struktur implementiert:**
```
Foundation Phase:
- Epic 1: Core Infrastructure & Meta-Strategy Framework 
- Epic 2: Trading Lifecycle Engine 

Phase 1: Strategy Configurations  
- Epic 3: SPX Iron Condor Configuration 
- Epic 4: SPY Strangle Configuration 

Phase 2: User Experience
- Epic 5: Dashboard & Monitoring  (Goal angepasst, Stories noch alt)
- Epic 6: Experiment Mode & Analytics  (Goal angepasst, Stories noch alt)

Phase 3: Advanced Features
- Epic 7: Multi-Strategy Orchestration  (Goal angepasst, Stories noch alt)
```

##  Session 5 Updates (2025-01-23 - KOMPLETTES EPIC RESTRUCTURING)

### REVOLUTION�RER NEUER ANSATZ: Value-Driven Epic Structure
**Problem:** Alte Epics zu technisch (Foundation � Strategy � UI), kein sofortiger User Value
**L�sung:** User-driven Ansatz - JEDES Epic = sofort nutzbares, testbares Inkrement

###  NEUE EPIC-STRUKTUR FINALISIERT:

#### Epic 1: "One-Click Daily Trade Execution"
**Konzept:** Manueller Trigger mit automatischer Ausf�hrung
- Web-UI mit "Schedule Trade" Button (immer aktiv)
- Editierbare Parameter: Entry Time, Exit Time, Profit Target, Stop Loss
- TWS Connection Settings (IP/Port - User Verantwortung)
- Podman Deployment von Anfang an
- Playwright E2E Tests inklusive
- **SAUBERE ERWEITERBARE ARCHITEKTUR** (ib_async, Service Layer, DI)
- **HINWEIS:** Zeiten nur in ET (Lokalisierung kommt in Epic 4)

#### Epic 2: "Minimal Kanban with Full Automation"
**Konzept:** Echtzeit-Visualisierung + volle Automatisierung
- Single-Row Kanban mit 7 Spalten (NEXT � ARCHIVE)
- Daily automation (automatisch jeden Trading Day)
- Skip & Archive Buttons
- TWS Reconciliation (2-Sekunden Polling)
- **HINWEIS:** In-Memory State (Persistence kommt in Epic 3)

#### Epic 3: "Professional UI, Analytics & Persistence"
**Konzept:** Von funktional zu professionell
- Finales Aqua-Style Design
- Swimlane-Struktur (prep f�r Multi-Strategy)
- Performance Metrics Panel
- Trade Log Table (mit Delete-Button)
- **SQLite Persistence f�r alles**
- Settings Modal statt inline Controls

#### Epic 4: "Setup Wizard & Localization"
**Konzept:** User-friendly Onboarding + korrekte Zeitanzeige
- Web-basierter 4-Screen Setup Wizard
- Deutsch/English Auswahl
- Timezone-korrekte Anzeige ("9:32 ET (15:32 CET)")
- TWS Connection Testing
- First-Run Auto-Launch

#### Epic 5: "Minimal Discord Integration"
**Konzept:** Remote Monitoring & Emergency Control
- Kritische Alerts (FR5, FR8, FR24, FR25)
- Kill-Switch Command (!kill)
- MINIMAL SCOPE (keine Status Commands etc.)
- Erweiterungen im Future Roadmap dokumentiert

###  WICHTIGE �NDERUNGEN:
- [x] Paper Trading KOMPLETT ENTFERNT (nur noch LIVE vs EXPERIMENT)
- [x] Port-Konfiguration = User-Verantwortung
- [x] Alte Epic-Struktur (Foundation � Strategy � UI) ARCHIVIERT
- [x] Stories werden vom ARCHITECT definiert (nicht vom PM)

## =� SUPERPROMPT F�R N�CHSTE SESSION

### Context f�r Claude:
```
Du bist John, der Product Manager f�r das Options Trading Bot Projekt.

AKTUELLER STATUS:
- PRD ist finalisiert mit 5 Value-Driven Epics
- Epic 1-5 sind vollst�ndig definiert
- Jedes Epic liefert sofortigen, nutzbaren Wert
- Paper Trading wurde entfernt (nur LIVE vs EXPERIMENT)
- Stories werden vom Architect definiert, nicht vom PM

EPIC �BERSICHT:
1. One-Click Trade (manueller Trigger, automatische Ausf�hrung)
2. Kanban & Automation (Visualisierung + Daily Auto-Trade)  
3. Polish & Persistence (Professional UI + SQLite)
4. Setup Wizard & Localization (Onboarding + Timezone)
5. Discord Integration (Minimal - nur kritische Alerts)

N�CHSTE SCHRITTE:
1.Ergänze Epic 1 auch mit der möglichen Positionsgröße als Einstellungsmöglichkeit (Standard: 1)
2. Dynamische Positionsermittlung in PRD aktualsieren
3. Epic 6 für dynmische Positionsermittlung erstellen
4. Alte technische Epics aus PRD entfernen (sind nur noch Referenz)
```

## =� N�CHSTE SESSION AGENDA

### 2. Dynamische Positionsermittlung in PRD aktualisieren
Fokussiere Dich nur auf das Bestimmmen der Positionsgröße beim Aufsetzen einer Strategieinstanz! Ich brauche kein Extra Dashboard mit Monitoring. Das soll keine Konkurrenz zur TWS werden.
  - Fixed Contract mit Auswirkung auf Kapital, Margin und Kaufkraft
  - Dynamic Position Sizing mit folgenden Parametern, die ich in den Einstellungen setzen muss
    - % Net Liq per Trade (Standard: 3 %)
    - % overall usage of overall buying power usage (Standard: 50 %)
    - VIX-Wert > x (Standard 30), ab wann Positionen halbiert werden sollen (minimum 1)
    - VIX-Wert > y (Standard 50), ab dann Positionen nicht mehr gehandelt werden sollen
- VIX-Wert darf höchstens 5 Sekunden alt sein. Ob gecached oder nicht, soll der Architekt enteschiden.
- Arbeite diese Bedingungen in existierende oder neue Warnungen in Spalte 1 ein. Falls diese Vorbedingungen bei start in Spalte 2 nicht matchen, soll die instanz in Spalte 2 einen entsprechenden Fehler bekommen, der nach unserer Spezifikation mit dem Archiv-Buton manuell archiviert werden muss.
- Verändere auf keinen Fall das Grundlayout der Karten. 
- Trenne sauber dabei zwischen Fachlichkeit (prd) und Anzeige (UI-UX-specification.md). Überarbeite die existierenden Anforderungen zum Thema und lösche sie wenn nötig. Alles muss konsistent und präzise sein!

### 3. Epic 6 für dynamische positionsermittlung erstellen
- Nutzer kann dann zwiscehn Fixed und Dynamic Positioning für alle neuen Trades umschalten.
- Wir haben Zugriff auf Vix-Wert

### 4. PRD Cleanup
- [ ] Alte Epic-Struktur (Technical Foundation etc.) aus PRD entfernen
- [ ] Nur neue Value-Driven Epics behalten
- [ ] Technical Models als Appendix verschieben


### 3. Epic 6 Definition
**Kandidaten basierend auf offenen Requirements:**
- **SPY Strangle (FR16-19)** - Zweite Strategy
- **Position Sizing (FR14)** - Risk Management
- **CSV Export (FR15)** - Vervollst�ndigt Analytics
- **Homebrew Installation (FR21)** - Deployment vereinfachen

### 4. Future Roadmap erweitern
- [ ] Epic 7-10 Ideen sammeln
- [ ] Multi-Strategy Orchestration planen
- [ ] Advanced Greeks Integration timing

## <� KRITISCHE ENTSCHEIDUNGEN GETROFFEN

1. **Value-Driven Epics** statt technische Phasen
2. **Architect definiert Stories** (PM nur Epics)
3. **Paper Trading entfernt** (Port-Config reicht)
4. **Discord minimal** (Erweiterungen sp�ter)
5. **Persistence erst Epic 3** (nicht Epic 2)
6. **Wizard in Epic 4** (nicht Epic 1)

## =� WICHTIGE ERKENNTNISSE

- User (Mathias) will **sofort nutzbaren Wert** pro Epic
- **MVP-Gedanke** wichtiger als Perfektion
- **Einfachheit** vor Feature-Reichtum
- Jedes Epic muss **f�r sich allein Sinn machen**
- **Iterative Entwicklung** basierend auf echtem Feedback

---

**STATUS: PRD Ready f�r Implementation von Epic 1!**