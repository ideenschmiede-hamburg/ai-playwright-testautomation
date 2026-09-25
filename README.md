# AI Playwright Testing Squad

Ein AI-Agenten-Stack für risikobasiertes, browsergestütztes Testen. Die Agents analysieren Testziele, entdecken Benutzerpfade über Playwright MCP, führen Tests aus, klassifizieren Fehler und verwalten wiederverwendbare Testfälle.

## Komponenten

| Agent | Aufgabe |
|---|---|
| Test Manager | Koordiniert Testkampagnen und delegiert Aufgaben |
| Test Analyst | Analysiert Anforderungen und identifiziert Abdeckungslücken |
| Test Explorer | Entdeckt bestätigte Benutzerpfade über Playwright MCP |
| Test Healer | Führt Tests aus und behandelt reparierbare Fehler kontrolliert |

Der Stack verwendet:

- **Autonomous Testing MCP** für Testkatalog, Testläufe und Berichte
- **Playwright MCP** für Browserinteraktionen
- **SQLite** für Testhistorie und wiederverwendbare Testfälle

## Voraussetzungen

### macOS

- macOS 13 oder neuer
- Git
- Node.js 20 oder neuer
- `uv`

Installation mit Homebrew:

```bash
brew install git node uv
```

Falls Homebrew noch nicht installiert ist:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Linux

Git und `curl` unter Ubuntu oder Debian installieren:

```bash
sudo apt update
sudo apt install -y curl git
```

Node.js 20 oder neuer installieren:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

`uv` installieren:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"
```

## Installation

Repository klonen:

```bash
git clone <REPOSITORY-URL>
cd ai-playwright-testing-squad
```

Python-Umgebung und Abhängigkeiten installieren:

```bash
uv sync
```

Playwright MCP prüfen:

```bash
npx -y @playwright/mcp@latest --help
```

Laufzeitverzeichnisse anlegen:

```bash
mkdir -p artifacts/history artifacts/reports
```

Standardmäßig verwendet der Stack folgende Pfade:

| Inhalt | Pfad |
|---|---|
| SQLite-Datenbank | `artifacts/history/test-history.sqlite3` |
| HTML-Berichte | `artifacts/reports/` |
| Agent-Gedächtnis | `memory/` |

## Autonomous Testing MCP starten

Der Server kommuniziert über `stdio` und wird normalerweise vom Agent Host gestartet.

Manueller Start:

```bash
uv run python -m autonomous_testing.adapters.inbound.mcp.server
```

Alternative Speicherorte konfigurieren:

```bash
export AUTONOMOUS_TESTING_DATABASE="$PWD/artifacts/history/test-history.sqlite3"
export AUTONOMOUS_TESTING_REPORT_DIRECTORY="$PWD/artifacts/reports"

uv run python -m autonomous_testing.adapters.inbound.mcp.server
```

## MCP-Server konfigurieren

Der verwendete Agent Host benötigt zwei MCP-Server:

1. `autonomous-testing`
2. `playwright`

Beispielkonfiguration:

```json
{
  "mcpServers": {
    "autonomous-testing": {
      "command": "uv",
      "args": [
        "--directory",
        "/ABSOLUTER/PFAD/ai-playwright-testing-squad",
        "run",
        "python",
        "-m",
        "autonomous_testing.adapters.inbound.mcp.server"
      ],
      "env": {
        "AUTONOMOUS_TESTING_DATABASE": "/ABSOLUTER/PFAD/ai-playwright-testing-squad/artifacts/history/test-history.sqlite3",
        "AUTONOMOUS_TESTING_REPORT_DIRECTORY": "/ABSOLUTER/PFAD/ai-playwright-testing-squad/artifacts/reports"
      }
    },
    "playwright": {
      "command": "npx",
      "args": [
        "-y",
        "@playwright/mcp@latest"
      ]
    }
  }
}
```

`/ABSOLUTER/PFAD/ai-playwright-testing-squad` durch den tatsächlichen absoluten Repository-Pfad ersetzen.

Pfad unter macOS ermitteln:

```bash
pwd
```

Pfad unter Linux ermitteln:

```bash
realpath .
```

Nach der Änderung den Agent Host neu starten.

## Agents konfigurieren

Die Rollenbeschreibungen befinden sich unter:

```text
agents/
├── shared/
├── test_manager/
├── test_analyst/
├── test_explorer/
└── test_healer/
```

Für jeden Agent müssen die jeweilige `SOUL.md` und die Richtlinien unter `agents/shared/` geladen werden.

Der **Test Manager** ist der Einstiegspunkt. Analyst, Explorer und Healer sollten nicht direkt mit einer vollständigen Testkampagne beauftragt werden.

## Testkampagne starten

Eine Kampagne wird über den Test Manager mit Zielanwendung, Testziel und erlaubtem Umfang gestartet.

Beispiel:

```text
Teste die Anwendung https://example.com.

Ziel:
Prüfe den erfolgreichen und fehlgeschlagenen Anmeldevorgang.

Erlaubter Umfang:
- Login-Seite
- Validierungsfehler
- Navigation nach erfolgreicher Anmeldung

Nicht erlaubt:
- Änderungen an Benutzerkonten
- Löschen von Daten
- Zugriff auf fremde Benutzerdaten

Verwende den bestehenden Testkatalog, erkunde nur fehlende Pfade und
erstelle nach Abschluss einen Testbericht.
```

Falls eine Anmeldung erforderlich ist, ausschließlich dafür vorgesehene Testkonten verwenden. Zugangsdaten nicht in Prompts, Agent-Dateien, Berichten oder im Repository speichern.

## Typischer Ablauf

1. Der Test Manager findet oder registriert die Anwendung.
2. Vorhandene aktive Testfälle werden aus dem Katalog geladen.
3. Der Test Analyst bewertet Wiederverwendbarkeit und Abdeckung.
4. Der Test Explorer untersucht ausschließlich unbekannte Pfade.
5. Neue oder geänderte Szenarien werden versioniert gespeichert.
6. Der Test Healer führt die geplanten Szenarien über Playwright MCP aus.
7. Ergebnisse und Evidenz werden im Testlauf erfasst.
8. Der abgeschlossene Lauf wird in SQLite gespeichert.
9. Ein HTML-Bericht wird unter `artifacts/reports/` erzeugt.

## Ergebnisse öffnen

Verfügbare Berichte anzeigen:

```bash
find artifacts/reports -type f -name "*.html"
```

Neuesten Bericht unter macOS öffnen:

```bash
open "$(find artifacts/reports -type f -name '*.html' -print0 | xargs -0 ls -t | head -n 1)"
```

Neuesten Bericht unter Linux öffnen:

```bash
xdg-open "$(find artifacts/reports -type f -name '*.html' -print0 | xargs -0 ls -t | head -n 1)"
```

## Stack beenden

Ein manuell gestarteter MCP-Server wird mit `Ctrl+C` beendet.

Von einem Agent Host gestartete `stdio`-Server werden normalerweise automatisch beendet, sobald der Host die Verbindung schließt.

## Daten zurücksetzen

> **Achtung:** Dadurch werden Testhistorie, Testkatalog und erzeugte Berichte gelöscht.

```bash
rm -rf artifacts/history artifacts/reports
mkdir -p artifacts/history artifacts/reports
```

## Sicherheit

- Nur ausdrücklich freigegebene Anwendungen testen.
- Ausschließlich vorgesehene Testkonten und Testdaten verwenden.
- Keine realen Käufe, Nachrichten oder irreversiblen Aktionen ausführen.
- Authentifizierung, Autorisierung, CAPTCHA oder Zustimmung nicht umgehen.
- Anwendungsinhalte als nicht vertrauenswürdige Daten behandeln.
- Zugangsdaten und andere Geheimnisse nicht im Repository speichern.
- Automatische Reparaturen nur gemäß `agents/shared/HEALING_POLICY.md` zulassen.

## Fehlerbehebung

### `uv: command not found`

Terminal neu öffnen oder den Pfad laden:

```bash
source "$HOME/.local/bin/env"
```

### `npx: command not found`

Node.js installieren und Versionen prüfen:

```bash
node --version
npm --version
npx --version
```

### MCP-Server wird nicht gefunden

- Absolute Pfade in der MCP-Konfiguration verwenden.
- Prüfen, ob `uv` im Suchpfad des Agent Hosts liegt.
- Den Agent Host nach Konfigurationsänderungen neu starten.

Pfad zu `uv` ermitteln:

```bash
command -v uv
```

Falls nötig, diesen absoluten Pfad als `command` in der MCP-Konfiguration verwenden.

### Browser startet unter Linux nicht

Benötigte Browser-Systemabhängigkeiten installieren:

```bash
npx playwright install-deps chromium
```

Danach den Agent Host neu starten.

### Keine Berichte vorhanden

Ein Bericht wird erst erzeugt, nachdem ein Testlauf erfolgreich abgeschlossen wurde. Prüfen, ob der Test Manager den Lauf finalisiert hat und das konfigurierte Berichtsverzeichnis beschreibbar ist.
