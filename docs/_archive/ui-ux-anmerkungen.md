# UI/UX Anmerkungen - Sammlung

## Status: SAMMLUNG ABGESCHLOSSEN - PRIORISIERTE ABARBEITUNG

Datum: 2024-01-15
Von: User

---

## Bereits umgesetzte Punkte ✓

1. ✓ Klare Bezeichner statt verwirrende Abkürzungen
2. ✓ Keine verwirrenden Fortschrittsbalken mehr
3. ✓ Klare Unterscheidung zwischen Trade und Strategie
4. ✓ Trennung zwischen globalen Controls und Strategy-spezifischen Elementen
5. ✓ Emergency Stop rechts neben System-Controls positioniert
6. ✓ Kein globaler Mode-Switch (Overdesign) - Per-Strategy Mode Control
7. ✓ Settings-Zugang über ⚙️ Button

---

## PRIORISIERUNG & ABHÄNGIGKEITEN

**Logische Reihenfolge basierend auf Abhängigkeiten:**

**PRIO 1: Fundamentale Architektur-Entscheidungen**
- ✅ Anmerkung 2: Kanban-Board Konzept (GRUNDKONZEPT GEKLÄRT - Detailfragen offen)
- 🔄 Anmerkung 3: Statistik-Trennung (Datenarchitektur-Grundlage)

**PRIO 2: Detaillierte Feature-Spezifikation** 
- ⏳ Anmerkung 4: Portfolio-Metriken (baut auf Anmerkung 3 auf)
- ⏳ Anmerkung 5: Strategy-Metriken + UI-Aufteilung (baut auf Anmerkung 3 auf)

**PRIO 3: Enhancement-Features**
- ⏳ Anmerkung 1: TWS Status Interaktivität (Nice-to-have)
- ⏳ Anmerkung 6: Trade-Daten für AI-Analyse (Future Enhancement)

---

## Anmerkungen in Bearbeitungsreihenfolge

### 🔄 Anmerkung 2: Kanban-Board Konzept - GRUNDKONZEPT DEFINIERT
**STATUS: KONZEPT GEKLÄRT - Detailfragen noch offen für Diskussion**

**Kern-Erkenntnis:** Unterscheidung zwischen STRATEGIE und STRATEGIE-INSTANZ
- **STRATEGIE** = Regelwerk (SPX Iron Condor, SPY Strangle)
- **INSTANZ** = Einzelne Ausführung (SPX#001, SPY#007)

**Kanban-Spalten Flow:**
READY → OPTIONS SEARCH → ORDER PLACING → EXIT SETUP → ACTIVE → CLOSED → ARCHIVE

**Status-Ampel pro Instanz:**
- ✅ Good/Ready/Done - Gesund, kann weitergehen
- 🔄 Doing - Work in Progress  
- ⏸️ Wait - Wartet auf externe Bedingung
- ❌ Error - Technisches Problem, Intervention nötig
- ⚠️ Warning - Trade erfordert Aufmerksamkeit

**Control-Hierarchie:**
- Global: [🔴 STOP ALL] - Alle Live-Instanzen
- Strategy: [LIVE]/[EXPERIMENT] Mode per Strategie
- Instance: [🔴 STOP] per aktive Instanz

**Besonderheit:** Error-Karten "verstopfen" Spalten absichtlich → zwingt zur Behandlung!
**Discarded Status:** Für unfixbare Errors → Nutzer kann sie direkt auf seine Entscheidung hin archivieren.

**OFFENE FRAGEN für später:**
- Wo stehen Strategy-Statistiken? (Vor/nach Swimlane oder separater Bereich?)
- Was zeigen Karten in Kompakt- vs Detailansicht?
- Archive-Spalte: Wieviel anzeigen? Separater Trade-Log?
- Wann wandern Instanzen ins Archiv?

### ⏳ Anmerkung 3: Statistiken-Trennung nach Modi
- ALLE Statistiken sollten getrennt gehalten werden nach Modi:
  - LIVE-Statistiken (echte Trades)
  - EXPERIMENT-Statistiken (simulierte Trades)
  - OVERALL-Statistiken (beide kombiniert)
- Pro Strategie:
  - Kompletter Statistik-Block für Live/Experiment/Overall
- Global Portfolio-Level (alle Strategien kombiniert):
  - Alle Portfolio-Statistiken für Live/Experiment/Overall
- UI-Darstellung: Tabs oder Filter für Live/Experiment/Overall?

### ⏳ Anmerkung 4: Portfolio-Statistiken - Spezifische Metriken
- Global Portfolio-Level soll zeigen (für Live/Experiment/Overall):
  - P/L
  - CAGR (Compound Annual Growth Rate)
  - Max Drawdown
  - MAR Ratio (CAGR / Max Drawdown)
- Farbliche Kennzeichnung: Rot (schlecht) / Grün (gut) je nach Performance

### ⏳ Anmerkung 5: Pro-Strategie Statistiken - Detaillierte Metriken
- Pro Strategie für Live/Experiment/Overall:

**Mit farblicher Kennzeichnung (rot/grün):**
- P/L
- CAGR
- Max Drawdown  
- MAR Ratio
- Avg Per Trade ($ x / Lot)
- Avg Winner ($ x / Lot)
- Avg Loser (-$ x / Lot)
- Max Winner
- Max Loser

**Ohne Farbkennzeichnung:**
- Win Percentage
- Total Premium | Capture Rate
- Avg. Minutes (wenn > 8h dann 1,x Trading Days) in Trade
- Trades | Winners

**Noch zu definieren:**
- Was zeigt die Standard-Strategie-Kachel?
- Was gehört in die erweiterte Detailsicht?

### ⏳ Anmerkung 1: TWS Connection Status - Zustände und Interaktivität
- Soll die TWS-Verbindungsanzeige verschiedene Zustände haben? (z.B. Connected, Disconnected, Connecting, Error)
- Soll die Anzeige interaktiv sein? 
  - Click für manuelles Reconnect?
  - Quick-Link zu Verbindungseinstellungen (Port, IP)?
- Visuelle Darstellung der verschiedenen Zustände?

### ⏳ Anmerkung 6: Trade-Daten für AI-Analyse
- Pro Trade sammeln für spätere AI-gestützte Verbesserungsvorschläge
- Balance zwischen: a) Genug Daten für sinnvolle AI-Analyse, b) Speicherplatz nicht überlasten
- Welche Datenpunkte sind essentiell für AI-Pattern-Erkennung?
- Komprimierung/Archivierung alter Daten?
- Datenformat für AI-Tools optimiert?

---

## EMPFOHLENER PROMPT für nächste Session:

```
Lass uns mit Anmerkung 2 Detailfragen weitermachen. 
Konkret: Wo sollten Strategy-Statistiken im Kanban-Board stehen? 
Nutze @.bmad-core/data/elicitation-methods.md um das systematisch 
zu durchleuchten.
```

---

## Arbeitsstand

- ✅ Sammlung abgeschlossen (6 Anmerkungen)
- ✅ Priorisierung durchgeführt (nach Abhängigkeiten)
- ✅ Anmerkung 2 Grundkonzept geklärt
- 🔄 Anmerkung 2 Detailfragen offen
- ⏳ Anmerkungen 3-6 warten auf Bearbeitung
- ✅ Dokumentation aktualisiert (PRD + UI/UX Spec)
- ⏳ Review nach Abschluss aller Anmerkungen