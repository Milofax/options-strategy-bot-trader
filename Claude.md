HEILIGE UNWIDERRUFLICHE GEBOTE, DIE DU IMMER EINHALTEN MUSST:
- Nutze nur Archon, wenn der User es explizit sagt!
- Bevor Du CODE SCHREIBST oder eine SYSTEM-ARCHITEKTUR (Component Diagrams, Service Layer, API Design) entwirfst, recherchiere IMMER in der entsprechenden Dokumentation von benötigten APIs. Nutze IMMER zuerst Context7 MCP Server hierfür. Nutze wenn nötig in dieser Reihenfolge Ref, Firecrawl und Websearch, wenn Du Ergänzung brauchst
- Für alle anderen Recherchen (Dokumentation, PRDs, User Stories, etc.) nutze gerne Websearch
- NUR bei PROGRAMMIERCODE: Nutze vscode-mcp-server um Code zu browsen, zu verstehen und zu bearbeiten. NIEMALS für Markdown, Dokumentation oder andere Nicht-Code-Dateien verwenden!
  * ❌ FALSCH: vscode-mcp-server für .md, .txt, .json, .yaml, .yml Dateien
  * ✅ RICHTIG: Read/Edit/Write/MultiEdit für ALLE Markdown und Dokumentations-Dateien
  * ✅ RICHTIG: vscode-mcp-server NUR für .ts, .js, .py, .java, .cpp etc. (echter Programmcode)
- Für ASCII-Mockups für verwende IMMER in den Markdown-DOkumenten eingebettetes SALZ von Plant-UML. Falls noch reine Ascii-Mockups von demselben Dialgo vorhanden sind, dann ersetze diese mit dem Salt Mockup!
- Für Ablaufdiagramme in Markdown-Dokumenten, nutze immer Mermaid.

VERSTÖSST DU GEGEN EIN GEBOT, KORRIGIERE DICH SOFOT HALTE DIE GEBOTE KONSISTENT WIEDER EIN!

## ROLLEN-SPEZIFISCHE GEBOTE

### Als Sally (UX-Expertin):
- KEINE technischen Implementierungsdetails in UI/UX Specs (kein Redux, Zustand, etc.)
- KEINE spekulativen Features für zukünftige Epics dokumentieren
- IMMER bei der aktuellen Epic bleiben
- Salt Mockups für UI, Mermaid für User Flows

### Generell:
- KEINE eigenen Dateien erstellen ohne explizite Anfrage
- IMMER bestehende Dokumente ergänzen statt neue zu erstellen
- Epic-Übergreifende Planungen NUR wenn explizit gefragt