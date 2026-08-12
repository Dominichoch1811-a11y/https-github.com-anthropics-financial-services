# Anleitung (Deutsch)

Dieses Repo ist eine Kopie von [anthropics/financial-services](https://github.com/anthropics/financial-services)
(Apache-2.0) mit einer Fehlerkorrektur. Es enthält **keine lauffähige Software**,
sondern Plugins für Claude selbst: Markdown und JSON, kein Build-Schritt.

> **Wichtig:** Nichts hier ist Anlage-, Rechts-, Steuer- oder Buchhaltungsberatung.
> Die Agenten erstellen Analysten-Entwürfe zur Prüfung durch einen Menschen. Sie
> geben keine Anlageempfehlungen und führen keine Transaktionen aus.

## Was drin ist

| | Anzahl | Wo |
|---|---|---|
| **Agenten** (fertige End-to-End-Workflows) | 10 | `plugins/agent-plugins/` |
| **Vertikale Plugins** (Skills + Slash-Kommandos) | 7 + 2 Partner | `plugins/vertical-plugins/`, `plugins/partner-built/` |
| **MCP-Datenanbindungen** | 12 | `plugins/vertical-plugins/financial-analysis/.mcp.json` |
| **Managed-Agent-Vorlagen** (headless über `/v1/agents`) | 10 | `managed-agent-cookbooks/` |

Die 10 Agenten: Pitch Agent, Meeting Prep, Market Researcher, Earnings Reviewer,
Model Builder, Valuation Reviewer, GL Reconciler, Month-End Closer, Statement
Auditor, KYC Screener.

Die 7 Verticals: financial-analysis (Kern), investment-banking, equity-research,
private-equity, wealth-management, fund-admin, operations. Dazu die
Partner-Plugins lseg und sp-global.

## Installation vom Handy

In der Claude-App (Cowork): **Einstellungen → Plugins → Plugin hinzufügen**,
dann diese Repo-URL einfügen:

```
https://github.com/Dominichoch1811-a11y/https-github.com-anthropics-financial-services
```

Danach aus der Marketplace-Liste auswählen, was du brauchst. **Nimm
`financial-analysis` zuerst** — dort liegen die gemeinsamen Modellier-Skills und
alle Datenanbindungen, die anderen Plugins bauen darauf auf.

Für deinen Anwendungsfall (Einzelaktien, Swing-Trading) ist danach
**`equity-research`** das passendste: Earnings-Analyse, Morning Note,
Katalysator-Kalender, Thesis-Tracker, Initiating Coverage.

### Am Rechner (Claude Code)

```bash
claude plugin marketplace add Dominichoch1811-a11y/https-github.com-anthropics-financial-services
claude plugin install financial-analysis@claude-for-financial-services
claude plugin install equity-research@claude-for-financial-services
```

Nach der Installation stehen die Slash-Kommandos in der Session zur Verfügung
(`/comps`, `/dcf`, `/earnings`, `/morning-note`, `/catalysts`, …), und die Skills
greifen automatisch, wenn sie zum Thema passen.

## MCP-Datenanbindungen — was das wirklich kostet

Das ist der Punkt, an dem man ehrlich sein muss: **alle 12 Anbindungen sind
kostenpflichtige Profi-Datendienste.** Die URLs stehen in der Konfiguration,
aber ohne Abo oder API-Key des jeweiligen Anbieters bekommst du keine Daten.

| Anbieter | Wofür | Zugang |
|---|---|---|
| Daloopa | Fundamentaldaten aus Filings | Abo |
| Morningstar | Fonds-, Aktien-Research | Abo |
| S&P Global (Kensho) | Capital IQ | Abo |
| FactSet | Terminal-Daten | Abo |
| Moody's | Kreditdaten | Abo |
| MT Newswires | Nachrichten | Abo |
| Aiera | Earnings-Call-Transkripte | Abo |
| LSEG | Anleihen, Swaps, FX, Vola | Abo |
| PitchBook | Private Markets | Abo |
| Chronograph | PE-Portfolio-Monitoring | Abo |
| Egnyte | Dokumentenablage | Abo/eigener Account |
| Box | Dokumentenablage | Abo/eigener Account |

**Praktische Konsequenz:** Die Skills und Agenten funktionieren auch ohne diese
Anbindungen — sie arbeiten dann mit dem, was du selbst hineingibst (hochgeladene
Geschäftsberichte, Excel-Dateien, Zahlen aus dem Chat) und mit Websuche. Was
fehlt, ist der automatische Datenabruf. Für Privatanleger ist das der
Normalfall; die Anbindungen zielen auf Banken und Fonds mit bestehenden
Terminal-Verträgen.

Dein `hebel-bot` löst dasselbe Problem anders: yfinance für Kurse und
Google-News-RSS für Schlagzeilen — kostenlos, dafür weniger tief.

## Abweichung vom Original

`plugins/vertical-plugins/financial-analysis/.mcp.json` war im Original
**syntaktisch kaputt**: nach dem `egnyte`-Block fehlte ein Komma, und der
`box`-Block war nicht korrekt geschlossen. Ein JSON-Parser bricht darauf ab —
das Plugin hätte **keine einzige** Datenanbindung geladen, und Box fehlte
zusätzlich ganz. In dieser Kopie ist das korrigiert, alle 12 Server sind
gültig konfiguriert. Geprüft mit `python3 scripts/check.py` (83 Dateien, 0
Probleme).

## Verbindung zum hebel-bot

Der [hebel-bot](https://github.com/Dominichoch1811-a11y/hebel-bot) hat bereits
zwei Skills aus diesem Repo übernommen, übersetzt in deterministischen
Python-Code:

- **`morning-note`** → Aufbau des Tagesreports (Top Call zuerst, dann Signale,
  dann Termine; je Signal die Invalidierung)
- **`catalyst-calendar`** → `bot/catalysts.py` (Earnings, Verfallstage, Makro)

Das sind zwei verschiedene Ebenen, die sich ergänzen:

| | hebel-bot | dieses Repo |
|---|---|---|
| Läuft | als Python-Cronjob auf deinem Rechner | in Claude (App oder Claude Code) |
| Macht | scannt täglich, rechnet Risiko-Parameter | hilft beim Analysieren und Schreiben |
| Daten | yfinance, RSS (kostenlos) | MCP-Anbieter (kostenpflichtig) oder was du hochlädst |
| Ergebnis | Signal mit Hebel, Stop, Größe | Modell, Memo, Research-Note |

Sinnvoller Ablauf: der Bot liefert morgens die Signale aufs Handy, und wenn du
zu einem Titel tiefer einsteigen willst, nimmst du dafür `equity-research` in
der Claude-App.

## Eigenes anpassen

Alles ist Markdown und JSON. Vor dem Pushen prüfen:

```bash
python3 scripts/check.py     # lintet jedes Manifest, prüft Querverweise
```

Neue Skills gehören unter `plugins/vertical-plugins/<vertical>/skills/`, danach
`python3 scripts/sync-agent-skills.py` laufen lassen, damit Agenten, die den
Skill bündeln, die Kopie bekommen.

## Lizenz

[Apache License 2.0](./LICENSE) — wie das Original.
