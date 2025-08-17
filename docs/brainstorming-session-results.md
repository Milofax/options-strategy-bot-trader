# Options Trading Automation Brainstorming Session

## Executive Summary
- **Topic:** Automatisierung eines 0DTE Iron Condor Trading Systems mit TWS API
- **Datum:** 2025-08-13
- **Ziel:** Fokussierte Ideenfindung für konkrete Implementierung
- **Strategie:** Täglicher IC um 15:32 CET, Delta 15 Short, ±20 Wings, 25% PT oder 17:30 Close

## Session-Fortschritt

### Gewählte Techniken (Analyst-Empfehlung)
1. "How Might We" - Technische Herausforderungen lösen
2. SCAMPER Method - Strategie-Optimierung
3. Worst Possible Idea - Risiko-Identifikation
4. First Principles Thinking - Fundamentale Anforderungen

---

## Technique 1: "How Might We" Questions
*Start: Jetzt*

### Generierte Ideen:

**HMW 1: Timing & Zuverlässigkeit**
- Trade-Bedingungen direkt an TWS anhängen prüfen
- Robuster Hintergrund-Prozess/Server für Zuverlässigkeit
- Exakte Zeitsynchronisation (NTP-Server)
- **WICHTIG:** 2 Minuten nach US-Marktöffnung (9:32 ET), nicht fixe CET-Zeit!
- Winter-/Sommerzeit-Unterschiede US/EU beachten
- Börsenkalender integrieren (Feiertage, Wochenenden)
- Verbindungs-Monitoring implementieren
- Multi-Channel-Benachrichtigungen:
  - Push-Notifications auf iPhone (priorität!)
  - E-Mail als Backup
  - SMS als weitere Option
- Erfolgsbestätigung für Trade + Exit-Orders (OCA)

**HMW 2: Delta-Matching-Algorithmus**
- Priorisierung nach tatsächlichem Delta (nicht Strike-Abstand)
- Delta-Auswahl-Logik:
  - Ziel: Delta 15
  - Bei 14.8 vs 15.3 → 14.8 (näher an 15)
  - Bei 14.8 vs 15.2 → 15.2 (leicht höheres Delta bevorzugen bei ähnlicher Distanz)
  - Bei 14 vs 16 → 16 (höheres Delta)
  - Obergrenze: 16.9 ist noch akzeptabel
  - Bei Delta >16.9 verfügbar → kleineres Delta wählen (z.B. 13 statt 17)
  - Risiko-Präferenz: Weniger Profit aber weniger Risiko bei Grenzfällen
- Wings: Immer ±20 Punkte vom gewählten Short Strike

**HMW 3: OCA Exit-Management**
- **WICHTIG: SPX Trading!** (Cash-settled, europäische Options)
- OCA-Gruppe "IC" mit zwei Orders:
  1. MKT Order mit Zeit-Bedingung: 11:30 ET (17:30 CET)
  2. LMT Order für 25% Profit-Target
- TWS-Spezifik: Negative Beträge für Credit-Spreads
  - Beispiel: IC für -5.00 verkauft → LMT Buy-Back bei -3.75
- OCA-Einstellung: "Cancel remaining orders" (nicht reduce size)
- Beide Orders müssen identische OCA-Gruppe haben
- Automatische Berechnung: Profit-Target = Entry-Preis × 0.75

**HMW 4: Fehlerbehandlung & Sicherheit**
- Status-Dashboard mit Fortschrittsanzeige:
  1. ✅ Verbindung zur TWS herstellen
  2. ⏰ Auf Einstiegszeitpunkt warten (9:32 ET)
  3. 📊 SPX-Preis abrufen
  4. 📈 Einstiegsindikator prüfen (optional)
  5. 🔍 Nach Optionen suchen (Delta-Matching)
  6. 🛒 Order erstellen und übertragen
  7. ⏳ Warten auf Ausführung
  8. 📤 Ausstiegsorder anhängen (OCA)
- Fill-Strategie:
  - Immer Mid-Price als Start
  - Bei Non-Fill: Nach 10 Sek neuer Mid-Price
  - Nach 20 Sek erneut versuchen
  - Adaptive Fill-Strategie entwickeln (z.B. graduell Richtung Ask/Bid)
- OCA nur nach erfolgreichem IC-Fill aufsetzen
- Fehler-Benachrichtigungen bei:
  - Verbindungsabbruch vor 9:32
  - Trading Permission Fehler
  - Failed Orders
- Manual Kill-Switch implementieren
- Optional: Markt-Condition-Filter (z.B. VIX-Schwelle)

**HMW 5: Performance-Tracking & Optimierung**
- Dashboard-Metriken (wie im Beispiel):
  - P/L (Total & per Trade)
  - CAGR & Win Percentage (83.7%!)
  - Max Drawdown & MAR Ratio
  - Capture Rate & Capital-Entwicklung
  - Avg Winner/Loser per Trade
  - Avg Minutes in Trade (59.4)
- Equity Curve Visualisierung (Linear & Log)
- Annual Metrics Bar Chart (Return vs MDD)
- Detaillierter Trade Log mit:
  - Alle 4 Legs mit Strike & Premium
  - Opening/Closing Zeiten & Preise
  - Exit-Grund (Profit Target vs Specified Time)
  - P/L pro Trade farbcodiert
- Filter & Sortier-Funktionen
- CSV-Export für externe Analyse
- Zusätzliche Tracking-Ideen:
  - Tatsächlich gewähltes Delta vs Target
  - Fill-Quality (Mid vs tatsächlicher Fill)
  - Marktbedingungen (VIX, SPX-Bewegung)
  - Time-to-Fill Statistiken

**HMW 6: Strategie-Integrität & Hypothetische Analysen**
- **PRIORITÄT 1: Strikte Regel-Implementierung**
  - Exakte Ausführung der definierten Strategie
  - Keine eigenmächtigen Änderungen
  - "Set and Forget" - Handarbeit eliminieren
- **PRIORITÄT 2: Hypothetische Analysen (parallel, nicht live)**
  - "What-if" Szenarien mitlaufen lassen
  - Beispiel: Bei Touch eines Short-Legs:
    - Was wäre bei Roll auf Delta 30?
    - Sofortiger Close des berührten Spreads?
  - Backtesting von Variationen
  - KI-gestützte Pattern-Erkennung
- Klare Trennung:
  - LIVE-Trading: Nur definierte Regeln
  - ANALYSE: Hypothetische Szenarien als Report
- Optimierungsvorschläge nur als Empfehlung
- Manuelle Freigabe für jede Regeländerung

---

## Technique 2: "Worst Possible Idea" - Risiko-Identifikation
*Start: Jetzt*

### Generierte Ideen:

**WPI 1: Position-Verdopplung bei Flash Crash**
- **Horror-Szenario:** Bot hält sich nicht an Regeln
  - Vergrößert unkontrolliert Positionen
  - Vergisst Exit-Orders
  - Kapital wird "zerschossen"
- **Schutzmaßnahmen:**
  - **3. OCA-Order als Katastrophen-Stop**
    - Bei 300% Verlust (IC für -5.00 → Stop bei -15.00)
    - Absolute Verlustbegrenzung
  - Strikte Position-Size-Limits im Code
  - Read-Only Mode bei extremen Marktbedingungen
  - Audit-Log für JEDE Bot-Aktion
  - Hard-coded Regel: NIEMALS Position erhöhen
  - Tägliches Limit: NUR 1 Trade pro Tag
- **Marktrisiko-Management:**
  - Circuit Breaker Erkennung
  - VIX-Spike Detection
  - Ungewöhnliche Spread-Weiten als Warning

**WPI 2: Trading mit veralteten Preisen**
- **Risiko teilweise mitigiert durch LMT-Orders!**
  - Keine MKT-Orders = keine Katastrophen-Fills
  - Mid-Price bietet Schutz vor extremen Preisen
- **Timeout-Strategie:**
  - Alle 5 Sekunden neuer Mid-Price Versuch
  - Harte Deadline: 9:33 ET → Trade abbrechen
  - Sofortige Benachrichtigung bei Abbruch
- **Definition "Erfolgreicher Trade":**
  - IC in TWS aufgesetzt ✓
  - 3 Exit-Orders (OCA) in TWS gesetzt ✓
  - NUR dann gilt Trade als erfolgreich
  - Alles andere → DRINGENDE Benachrichtigung
- **Phasen-Monitoring hilft:**
  - Jede Phase muss bestätigt werden
  - TWS-Verbindung kontinuierlich prüfen
  - Order-Status verifizieren
  - Exit-Orders nur nach IC-Bestätigung

**WPI 3: Bot trifft autonome Trading-Entscheidungen**
- **LÖSUNG: Feste Regeln, keine Autonomie!**
  - Jeden Tag exakt gleiche Strategie
  - Nur Kill-Switch als manuelle Kontrolle
- **Trading-Modi System:**
  - "Live" Mode: Echte Trades mit echtem Geld
  - "Experiment" Mode: Tracking mit echten Marktdaten
  - Toggle zwischen Modi möglich
  - Bessere Validierung als Paper-Trading (Mid-Price!)
- **Zukunftsvision (nach Stabilisierung):**
  - KI-gestützte hypothetische Korrekturen
  - Laufzeit-Analysen (30-Delta-Roll etc.)
  - Aber immer nur als Vorschlag, nie autonom
- **PRIORITÄT JETZT:**
  - Stabiler Bot für "Urlaubs-Modus"
  - Kein Blödsinn, keine Überraschungen
  - 100% Regelkonformität
  - Set-and-Forget Zuverlässigkeit

**WPI 4: Automatischer Neustart bei Fehlern**
- **KEINE doppelten/halben Orders!**
  - IC MUSS als EINE Combo-Strategy an TWS
  - Atomare Operation: Alles oder nichts
- **Preis-Ermittlung:**
  - Mid-Price für gesamte Combo-Strategy
  - Daraus TP (75%) und SL (300%) berechnen
  - Nur möglich mit vollständigem IC
- **State-Management & Persistenz:**
  - Bot muss Trade-Status kennen:
    - Noch nicht gestartet?
    - IC bereits in TWS?
    - Exit-Orders gesetzt?
  - Vor 9:33 ET: Neuer Versuch möglich
  - Nach 9:33 ET: Kein neuer Versuch heute
- **Persistente Datenspeicherung:**
  - Alle Trades in Datenbank
  - Status-Recovery nach Neustart
  - Statistiken dauerhaft gesichert
  - Audit-Trail für jeden Schritt

**WPI 5: Bot läuft auf privatem Laptop**
- **Risiken Desktop/Laptop:**
  - Windows/Mac Updates zur Trading-Zeit
  - Schlafmodus (weniger Problem mit TWS)
  - WLAN-Unterbrechungen (kritisch!)
  - Stromausfall/Hardware-Fehler
  - Unbeabsichtigte Interaktionen (Katze!)
  - TWS Session-Timeout
- **LÖSUNG: Server-Deployment**
  - Mac Mini Server als Basis
  - Container-Lösung (Podman Pod)
  - Headless Operation (keine GUI nötig)
  - Nur IP:Port Verbindung zu TWS
- **Verbindungs-Philosophie:**
  - TWS nicht erreichbar → Benachrichtigung
  - Kein Trade ist OK! (Konsistenz > Perfektion)
  - "Manchmal ist kein Trade der beste Trade"
  - Langfristigkeit und Konsistenz im Fokus
- **Monitoring & Alerting:**
  - Heartbeat-Check zur TWS
  - Multi-Channel Benachrichtigung
  - Graceful Degradation

**WPI 6: Position-Sizing nach Gewinn/Verlust-Streaks**
- **Problem:** Emotionale Size-Anpassungen
  - "Hot Hand Fallacy" bei Gewinn-Streaks
  - Angst-basiertes Pausieren bei Verlusten
  - Zerstört mathematische Edge der Strategie
- **LÖSUNG: Konsistente Position-Sizing Regeln**
  - Option 1: Fixe Kontraktanzahl (von User gesetzt)
  - Option 2: Fester % vom Kapital
  - Option 3: Fester % der verfügbaren Margin
  - Immer: Position-Kosten < definierte %
- **Keine Streak-basierten Änderungen:**
  - Statistik braucht Konsistenz
  - Jeder Tag ist unabhängig
  - Langfristige Edge > kurzfristige Varianz
- **Position-Size nur manuell änderbar:**
  - User entscheidet über Anpassungen
  - Bot führt stur aus
  - Keine "intelligenten" Anpassungen

**WPI 7: Multi-Strategie Chaos**
- **Chaos-Potential bei Multi-Strategie:**
  - Margin-Management wird komplex
  - Strategien könnten sich hedgen
  - Fehlerquellen multiplizieren sich
  - Order-Zuordnung unklar
  - Capital-Allocation-Albtraum
- **LÖSUNG: Phasenweiser Ausbau**
  - JETZT: Eine Strategie perfektionieren (0DTE IC)
  - Stabile Basis schaffen
  - Saubere Architektur für Erweiterung
- **Zukunfts-Strategien (nach Stabilisierung):**
  - Flyogonal (Custom-Strategie)
  - 42 DTE Strangles (TastyTrade Style)
  - Delta = Expected Move Ansatz
- **Architektur-Prinzipien für später:**
  - Jede Strategie eigenes Modul
  - Klare Kapital-Trennung
  - Separate Tracking & Reporting
  - Keine Cross-Strategie-Abhängigkeiten
- **Motto: "Eine Sache richtig machen"**
- **Zukunft: Portfolio-Optimierung mit KI**
  - Korrelationsanalyse zwischen Strategien
  - Portfolio-Management Best Practices
  - Diversifikation nach:
    - Richtung (direktional vs neutral)
    - Volatilität (low vs high vola plays)
    - Zeit (0DTE vs 42DTE)
  - Risiko-Reduktion:
    - Weniger anfällig für Richtungsschwankungen
    - Hedge-Effekte zwischen Strategien nutzen
  - KI-gestützte Hinweise:
    - "Dein Portfolio ist zu direktional"
    - "Vola-Exposure zu konzentriert"
    - Optimale Kapital-Allokation vorschlagen

**WPI 8: Backtest-Optimierung Gone Wild**
- **Problem: Überoptimierung/Curve-Fitting**
  - Bot passt Parameter an Noise an
  - Zerstört bewährte Edge
  - Ständige Regeländerungen
  - Performance-Tracking wird unmöglich
  - "Optimiert" bis zum Totalausfall
- **LÖSUNG: KEINE BOT-OPTIMIERUNG!**
  - Bot führt nur aus, optimiert nie
  - Parameter-Änderungen nur manuell
  - Strategie bleibt konstant
  - Statistiken sammeln, nicht reagieren
- **Wenn Optimierung, dann:**
  - Nur durch User-Entscheidung
  - Nach ausreichend Datenpunkten (>100 Trades)
  - Mit klarer Hypothese
  - Separat testen (Experiment-Modus)
  - Niemals automatisch live

---

## Session-Zusammenfassung & Action Items

### Kernziel
Automatisierung der täglichen 0DTE Iron Condor Strategie auf SPX via TWS API, um manuelle Arbeit zu eliminieren und konsistente Ausführung zu gewährleisten.

### Kritische Anforderungen (Must-Haves)
1. **Timing:** Trade um 9:32 ET (2 Min nach Marktöffnung)
2. **Strategie:** IC mit Delta 15 Short, ±20 Wings
3. **Exit:** 25% Profit Target ODER 17:30 CET Close (OCA)
4. **Sicherheit:** 3. OCA Stop bei 300% Verlust
5. **Atomare Ausführung:** IC als Combo-Order
6. **State-Management:** Persistenz & Recovery
7. **Benachrichtigungen:** Push/Email/SMS bei Problemen

### Technologie-Stack (Empfehlung)
- **Runtime:** Python (beste TWS API Support via ib_insync)
- **Deployment:** Podman Container auf Mac Mini
- **Database:** PostgreSQL für Trade-History
- **Monitoring:** Grafana Dashboard
- **Notifications:** Pushover/Telegram API

### Implementierungs-Roadmap

#### Phase 1: MVP (Woche 1-2)
- TWS API Verbindung etablieren
- Delta-Matching Algorithmus
- IC Combo-Order Execution
- OCA Exit-Orders
- Basic Logging

#### Phase 2: Robustheit (Woche 3)
- State-Management & Persistenz
- Error Handling & Recovery
- Notification System
- Container-Deployment

#### Phase 3: Monitoring (Woche 4)
- Dashboard mit Live-Status
- Performance Tracking
- Trade Log & Export
- Experiment-Modus

### Nächste konkrete Schritte
1. TWS API Gateway auf Mac Mini einrichten
2. Python-Projekt mit ib_insync initialisieren
3. Delta-Matching Algorithmus implementieren
4. Test mit Paper-Trading Account

---

## Technische Architektur-Details

### Library-Entscheidung: ib_insync
**Warum ib_insync statt native ibapi:**
- Async/await Support für zeitkritische Execution
- Pythonic API - weniger Boilerplate
- Eingebaute Reconnection-Logic
- Event-driven für Status-Monitoring
- Einfache Combo-Order Implementation

### UI/UX Architektur

#### 1. Web-Dashboard (Primäre UI)
- **Tech:** FastAPI + HTMX + Chart.js
- **Features:**
  - Live-Status Updates (wie Mockup)
  - Mobile-friendly (iPhone-Zugriff)
  - P/L Charts direkt eingebaut
  - Kill-Switch Button
- **Vorteil:** Kein App-Install, überall erreichbar

**Warum HTMX?**
- Echtzeit-Updates OHNE JavaScript-Framework (React/Vue)
- Server-side Rendering = Einfacher Code
- Perfekt für Status-Updates alle 2 Sekunden
- Minimale Komplexität, maximale Funktion
- Beispiel: `<div hx-get="/status" hx-trigger="every 2s">` = Auto-Refresh

**Alternativen zu HTMX:**

**Option A: Pure WebSockets**
```python
# Pro: Echte Realtime, bidirektional
# Contra: Mehr Code, Connection-Management
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        await websocket.send_json({"status": bot.phase})
        await asyncio.sleep(1)
```

**Option B: React/Vue SPA**
```javascript
// Pro: Rich UI, moderne UX
// Contra: Overkill, Build-Process, npm Dependencies
useEffect(() => {
  const interval = setInterval(() => {
    fetch('/api/status').then(r => r.json())
  }, 2000)
}, [])
```

**Option C: Server-Sent Events (SSE)**
```python
# Pro: Unidirektional, einfach
# Contra: Nur Server→Client
@app.get("/events")
async def events():
    async def generate():
        while True:
            yield f"data: {json.dumps(status)}\n\n"
            await asyncio.sleep(2)
    return StreamingResponse(generate())
```

**Option D: Plain JavaScript + Fetch**
```javascript
// Pro: Keine Dependencies
// Contra: Mehr manuelle DOM-Manipulation
setInterval(() => {
  fetch('/api/status')
    .then(r => r.json())
    .then(data => {
      document.getElementById('status').innerHTML = data.phase
    })
}, 2000)
```

**Empfehlung bleibt HTMX weil:**
- Sie wollen "Set & Forget" - HTMX ist wartungsarm
- Keine Build-Pipeline nötig
- HTML bleibt lesbar
- Progressive Enhancement möglich

#### 2. Discord Bot (Notifications & Control)
- **Warum Discord statt Telegram:**
  - Vermutlich bereits in Nutzung
  - Bessere Embeds & Formatierung
  - Private Server möglich
  - Thread-Support für History
- **Features:**
  - Trade-Notifications
  - Status-Commands (!status, !pnl)
  - Remote Kill-Switch
  - Rich Embeds für Reports

#### 3. Datenbank: SQLite
- **Warum SQLite statt PostgreSQL (initial):**
  - Zero-Configuration
  - Single-File Database
  - Ausreichend für <10k Trades/Jahr
  - Einfaches Backup
- **Migration-Path:** SQLite → PostgreSQL wenn nötig

### Deployment-Architektur

```
Mac Mini Server
├── Podman Container
│   ├── Trading Bot (Python)
│   ├── Web Server (FastAPI)
│   └── Discord Bot
├── IB Gateway (Host)
│   └── Port 7496 (Live) / 7497 (Paper)
└── SQLite Database (Volume Mount)
```

### Projekt-Struktur

```
options-strategy-bot-trader/
├── src/
│   ├── main.py           # Hauptprogramm & Scheduler
│   ├── trading/
│   │   ├── connection.py # TWS/IB Gateway Connection
│   │   ├── strategy.py   # IC Strategy Logic
│   │   ├── orders.py     # Order Management & OCA
│   │   └── monitor.py    # Position Monitoring
│   ├── ui/
│   │   ├── webapp.py     # FastAPI Dashboard
│   │   ├── discord_bot.py # Discord Integration
│   │   └── templates/
│   │       └── dashboard.html
│   └── utils/
│       ├── logger.py     # Logging Configuration
│       ├── config.py     # Settings Management
│       └── database.py   # SQLite Handler
├── config/
│   ├── settings.yaml     # Trading Parameters
│   └── .env             # Secrets (Discord Token, etc.)
├── data/
│   └── trades.db        # SQLite Database
├── docker/
│   └── Dockerfile       # Podman Container
└── tests/
    └── test_strategy.py # Unit Tests

```

### Entwicklungs-Phasen (Refined)

#### Phase 1: Core Trading Logic (Woche 1)
- IB Gateway Setup & Connection
- Delta-Matching Algorithmus
- IC Combo-Order Creation
- Paper Trading Tests

#### Phase 2: Reliability & Monitoring (Woche 2)
- State Management & Recovery
- Discord Notifications
- Error Handling
- Container Deployment

#### Phase 3: UI & Analytics (Woche 3)
- Web Dashboard
- Live Status Display
- P/L Tracking
- Trade History

#### Phase 4: Production & Optimization (Woche 4)
- Live Trading Activation
- Performance Monitoring
- Experiment Mode
- Documentation

### Security & Risk Management

#### Connection Security
- IB Gateway nur auf localhost
- API Read-Only wo möglich
- Credentials in .env (nie in Git)

#### Trading Safeguards
- Max 1 Trade pro Tag (hardcoded)
- Position Size Limits
- 300% Stop-Loss OCA
- Kill-Switch über Discord/Web

#### Monitoring & Alerts
- Heartbeat every 30 seconds
- Discord Alert bei:
  - Connection Loss
  - Failed Orders
  - Unusual Market Conditions
- Auto-Recovery wo möglich

### Code-Beispiele

#### TWS Connection (ib_insync)
```python
from ib_insync import IB, Index, Contract, ComboLeg, LimitOrder
import asyncio

class TWSConnection:
    def __init__(self, host='127.0.0.1', port=7497):
        self.ib = IB()
        self.host = host
        self.port = port
        
    async def connect(self):
        await self.ib.connectAsync(self.host, self.port)
        print(f"Connected to TWS on {self.host}:{self.port}")
        
    async def get_spx_chain(self, dte=0):
        spx = Index('SPX', 'CBOE')
        chains = await self.ib.reqSecDefOptParamsAsync(
            spx.symbol, '', spx.secType, spx.conId
        )
        # Filter für 0DTE
        return chains
```

#### Discord Integration
```python
import discord
from discord.ext import commands

class TradingBot(commands.Bot):
    def __init__(self, trading_system):
        super().__init__(command_prefix='!')
        self.trading = trading_system
        
    async def notify_trade(self, trade_info):
        channel = self.get_channel(TRADING_CHANNEL_ID)
        embed = discord.Embed(
            title="✅ Trade Executed",
            color=discord.Color.green()
        )
        embed.add_field(name="Strategy", value="0DTE IC")
        embed.add_field(name="Credit", value=f"${trade_info['credit']}")
        embed.add_field(name="Strikes", value=trade_info['strikes'])
        await channel.send(embed=embed)
```

#### Web Dashboard Route
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/api/status")
async def get_status():
    return {
        "connected": ib.isConnected(),
        "phase": bot.current_phase,
        "next_trade": bot.next_trade_time,
        "positions": bot.get_positions(),
        "daily_pnl": bot.calculate_daily_pnl()
    }

@app.post("/api/kill-switch")
async def kill_switch():
    await bot.emergency_close_all()
    await discord_bot.notify("🛑 Kill Switch Activated!")
    return {"status": "All positions closed"}
```

---

## Advanced Elicitation: Technische Vertiefung

### TWS API Connection Details

#### Connection Architecture
```python
# Connection Manager mit Auto-Reconnect
class TWS ConnectionManager:
    def __init__(self):
        self.ib = IB()
        self.reconnect_attempts = 0
        self.max_reconnects = 5
        
    async def maintain_connection(self):
        """Heartbeat & Auto-Reconnect Logic"""
        while True:
            if not self.ib.isConnected():
                await self.reconnect()
            await asyncio.sleep(30)  # 30s heartbeat
            
    async def reconnect(self):
        """Exponential backoff reconnection"""
        wait_time = min(2 ** self.reconnect_attempts, 60)
        await asyncio.sleep(wait_time)
        try:
            await self.ib.connectAsync('127.0.0.1', 7496)
            self.reconnect_attempts = 0
        except:
            self.reconnect_attempts += 1
```

#### SPX Options Contract Specification
```python
# SPX Contract Details
def create_spx_option(strike, right, expiry='20250813'):
    """SPX ist SPXW für 0DTE"""
    return Option(
        symbol='SPX',
        lastTradeDateOrContractMonth=expiry,
        strike=strike,
        right=right,  # 'P' oder 'C'
        exchange='SMART',
        currency='USD',
        multiplier='100'
    )

# Iron Condor als Combo
def create_iron_condor_combo(put_long, put_short, call_short, call_long):
    """BAG Order für atomare Execution"""
    combo = Contract()
    combo.symbol = 'SPX'
    combo.secType = 'BAG'
    combo.currency = 'USD'
    combo.exchange = 'SMART'
    
    # Leg IDs müssen von TWS qualified werden
    combo.comboLegs = [
        ComboLeg(conId=put_long.conId, ratio=1, action='BUY'),
        ComboLeg(conId=put_short.conId, ratio=1, action='SELL'),
        ComboLeg(conId=call_short.conId, ratio=1, action='SELL'),
        ComboLeg(conId=call_long.conId, ratio=1, action='BUY')
    ]
    return combo
```

### Order Management Details

#### OCA Group Implementation
```python
def create_oca_exit_orders(combo_contract, entry_price, quantity):
    """Drei Exit-Orders als OCA Gruppe"""
    oca_group = f"IC_EXIT_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
    
    # 1. Profit Target (25%)
    profit_target = LimitOrder(
        action='BUY',
        totalQuantity=quantity,
        lmtPrice=round(entry_price * 0.75, 2),  # 25% profit
        ocaGroup=oca_group,
        ocaType=1  # Cancel others
    )
    
    # 2. Time-based Exit
    time_exit = MarketOrder(
        action='BUY',
        totalQuantity=quantity,
        goodAfterTime='20250813 17:30:00 US/Eastern',
        ocaGroup=oca_group,
        ocaType=1
    )
    
    # 3. Stop Loss (300%)
    stop_loss = StopOrder(
        action='BUY',
        totalQuantity=quantity,
        stopPrice=round(entry_price * 3.0, 2),
        ocaGroup=oca_group,
        ocaType=1
    )
    
    return [profit_target, time_exit, stop_loss]
```

### State Management & Persistence

#### Database Schema (SQLite)
```sql
-- Trades Table
CREATE TABLE trades (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    trade_date DATE NOT NULL,
    entry_time TIMESTAMP,
    entry_price DECIMAL(10,2),
    put_short_strike INTEGER,
    put_long_strike INTEGER,
    call_short_strike INTEGER,
    call_long_strike INTEGER,
    exit_time TIMESTAMP,
    exit_price DECIMAL(10,2),
    exit_reason VARCHAR(50),
    pnl DECIMAL(10,2),
    commission DECIMAL(10,2),
    status VARCHAR(20)
);

-- Trade State Table (für Recovery)
CREATE TABLE trade_state (
    id INTEGER PRIMARY KEY,
    current_date DATE,
    phase VARCHAR(50),
    ic_order_id VARCHAR(50),
    oca_orders_placed BOOLEAN,
    last_update TIMESTAMP
);
```

#### State Recovery nach Restart
```python
class StateManager:
    def __init__(self, db_path='data/trades.db'):
        self.conn = sqlite3.connect(db_path)
        
    def recover_state(self):
        """Prüft ob heute schon getradet wurde"""
        today = datetime.now().date()
        cursor = self.conn.execute(
            "SELECT * FROM trade_state WHERE current_date = ?", 
            (today,)
        )
        state = cursor.fetchone()
        
        if state:
            return {
                'already_traded': state['oca_orders_placed'],
                'phase': state['phase'],
                'can_retry': datetime.now().time() < time(9, 33)
            }
        return {'already_traded': False, 'phase': 'waiting'}
```

### Container Deployment Details

#### Dockerfile für Podman
```dockerfile
FROM python:3.11-slim

# System dependencies
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Application
WORKDIR /app
COPY src/ ./src/
COPY config/ ./config/

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8000/health || exit 1

# Run
CMD ["python", "-m", "src.main"]
```

#### Podman Compose
```yaml
version: '3'
services:
  trading-bot:
    build: .
    container_name: spx-ic-bot
    volumes:
      - ./data:/app/data  # Persistent storage
      - ./logs:/app/logs
    environment:
      - TZ=America/New_York
      - TWS_HOST=host.containers.internal  # Mac-spezifisch
      - TWS_PORT=7496
      - DISCORD_TOKEN=${DISCORD_TOKEN}
    ports:
      - "8000:8000"  # Web Dashboard
    restart: unless-stopped
    network_mode: bridge
```

### Discord Bot Integration Details

```python
class TradingDiscordBot(commands.Bot):
    def __init__(self, trading_system):
        intents = discord.Intents.default()
        intents.message_content = True
        super().__init__(command_prefix='!', intents=intents)
        self.trading = trading_system
        
    @commands.command()
    async def status(self, ctx):
        """!status - Zeigt aktuellen Bot-Status"""
        status = self.trading.get_current_status()
        embed = discord.Embed(
            title="🎯 SPX IC Bot Status",
            color=0x00ff00 if status['connected'] else 0xff0000
        )
        embed.add_field(name="TWS", value="✅ Connected" if status['connected'] else "❌ Disconnected")
        embed.add_field(name="Phase", value=status['phase'])
        embed.add_field(name="Next Trade", value=status['next_trade'])
        embed.add_field(name="Today's P/L", value=f"${status['daily_pnl']:.2f}")
        await ctx.send(embed=embed)
    
    @commands.command()
    async def kill(self, ctx):
        """!kill - Aktiviert Kill-Switch"""
        if ctx.author.id == OWNER_ID:  # Nur Sie können das
            await self.trading.emergency_stop()
            await ctx.send("🛑 **KILL SWITCH ACTIVATED** - All trading stopped!")
```

### Monitoring & Alerting

#### Prometheus Metrics (Optional für später)
```python
from prometheus_client import Counter, Gauge, Histogram

# Metrics
trades_total = Counter('trades_total', 'Total trades executed')
trades_won = Counter('trades_won', 'Winning trades')
current_pnl = Gauge('current_pnl', 'Current P/L')
execution_time = Histogram('execution_time_seconds', 'Trade execution time')
```

### Security Considerations

#### Credentials Management
```python
# .env file (NEVER commit!)
DISCORD_TOKEN=your_discord_bot_token
TWS_USERNAME=your_username  # Wenn needed
TWS_PASSWORD=encrypted_password  # Verschlüsselt!

# config/settings.yaml
trading:
  max_position_size: 1
  target_delta: 15
  wing_offset: 20
  profit_target_percent: 25
  
security:
  allowed_ips: ['127.0.0.1']
  require_2fa_for_changes: true
  encrypt_sensitive_data: true
```

---

## Kritische Punkte - Geklärt

### ✅ Market Data
- Options Real-Time vorhanden (das reicht!)
- SPX Index Level nicht nötig für Strategy
- Delta/Greeks kommen von Options-Chain

### ✅ Combo Pricing
- TWS Combo-Mid verwenden (SPX sehr liquide)
- Alle 5 Sekunden neuer Versuch mit neuem Mid
- Konfigurierbar in Settings

### ✅ Live vs Experiment Mode
- KEIN Paper Trading!
- "Live" = Echte Orders an TWS
- "Experiment" = Tracking ohne TWS-Order (hypothetisch)
- Beide nutzen echte Marktdaten

### ✅ Feiertage
- NUR US-Börsenkalender relevant
- pandas_market_calendars für NYSE/CBOE

### ✅ Portfolio Margin
- Vorhanden! (~$500-800 pro IC)
- Viel effizienter als Reg-T

### ✅ IB Gateway Empfehlung
```yaml
Empfehlung: IB Gateway statt TWS
- Stabiler für 24/7 Operation
- Weniger Ressourcen
- Headless (keine GUI)
- Auto-Restart möglich
- Gleiche API, Port 4001 (Live) / 4002 (Paper)
```

### ✅ Timing
- Fix bei 9:32 ET (keine Optimierung nötig!)
- 300 Trades backgetestet
- Strategie ist bewährt

### ✅ Emergency Handling  
- Bei Short Strike Touch: NUR Alert
- Keine automatische Action
- Zukünftig: Parallele Analyse

### ✅ Performance Tracking
- Basis-Metrics tracken (Slippage, Commission)
- Keine Überladung
- SQLite bleibt schlank

### ✅ Partial Fills
- IC als BAG/COMBO Strategy = TWS Problem
- Atomare Execution garantiert
- Wir müssen nur Status prüfen

---

## Document Output

Das vollständige Brainstorming-Dokument wurde erfolgreich in `/Users/mathiasboni/Coding/options-strategy-bot-trader/docs/brainstorming-session-results.md` gespeichert.

### Inhalt des Dokuments:
1. **Executive Summary** - Projektziele und Strategie-Overview
2. **Session-Fortschritt** - Verwendete Brainstorming-Techniken
3. **How Might We Questions** - 6 kritische Fragen mit Lösungen
4. **Worst Possible Ideas** - 8 Risiko-Szenarien mit Schutzmaßnahmen
5. **Session-Zusammenfassung** - Klare Requirements und Action Items
6. **Technische Architektur** - Detaillierte Implementierungs-Specs
7. **Advanced Elicitation** - Technische Vertiefung
8. **Kritische Punkte** - Alle offenen Fragen geklärt

### Nächste Schritte:
1. IB Gateway auf Mac Mini installieren
2. Python-Projekt mit ib_insync initialisieren
3. Implementierung starten

Das Dokument enthält alle notwendigen Details für die Implementierung Ihres 0DTE Iron Condor Trading Bots.
