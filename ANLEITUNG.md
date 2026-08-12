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
| **MCP-Datenanbindungen** | 0 aktiv (12 als Vorlage) | `plugins/vertical-plugins/financial-analysis/.mcp.json.beispiel` |
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

## MCP-Datenanbindungen sind bewusst abgeschaltet

Im Original bringt das Kern-Plugin zwölf Datenanbindungen mit. **Alle zwölf sind
kostenpflichtige Profi-Datendienste** (Daloopa, Morningstar, S&P Global,
FactSet, Moody's, MT Newswires, Aiera, LSEG, PitchBook, Chronograph, Egnyte,
Box). Ohne Abo des jeweiligen Anbieters liefern sie keine Daten.

In dieser Kopie sind sie deshalb **deaktiviert** — alle `.mcp.json` stehen auf
`{"mcpServers": {}}`. Beim Installieren fragt dich damit nichts nach Zugängen,
und es läuft nichts ins Leere.

**Das kostet dich keine Funktion, die du hättest nutzen können.** Die Skills und
Agenten arbeiten ohnehin mit dem, was du hineingibst: hochgeladene
Geschäftsberichte und Excel-Dateien, Zahlen aus dem Chat, Websuche. Was fehlt,
ist allein der automatische Abruf aus einem Terminal — und den hättest du ohne
Vertrag sowieso nicht gehabt. Die Anbindungen zielen auf Banken und Fonds.

Zwei Plugins sind ohne ihre Datenquelle allerdings **sinnlos**: `lseg` und
`spglobal` (Partner-Plugins) bestehen praktisch nur aus Workflows auf genau
diesen Feeds. Sie liegen weiter im Repo, aber installier sie nicht — sie
brächten dir nichts.

Dein `hebel-bot` löst dasselbe Problem kostenlos: yfinance für Kurse,
Google-News-RSS für Schlagzeilen. Weniger tief, aber ohne Vertrag.

### Später doch aktivieren

Die reparierte Vollkonfiguration liegt als Vorlage bereit:
`plugins/vertical-plugins/financial-analysis/.mcp.json.beispiel`. Falls du
irgendwann einen Zugang hast, kopierst du den gewünschten Eintrag daraus in
`.mcp.json` (den Schlüssel `_hinweis` weglassen) — einzelne Anbieter gehen
auch, es müssen nicht alle sein.

## Abweichungen vom Original

1. **MCP-Anbindungen deaktiviert** (siehe oben) — die reparierte Vollversion
   liegt als `.mcp.json.beispiel` daneben.
2. **Syntaxfehler korrigiert:** `financial-analysis/.mcp.json` war im Original
   ungültiges JSON — nach dem `egnyte`-Block fehlte ein Komma, und der
   `box`-Block war nicht geschlossen. Ein Parser bricht darauf ab, das Plugin
   hätte also *keine einzige* Anbindung geladen, und Box fehlte zusätzlich
   ganz. Die Korrektur steckt in der Beispiel-Vorlage, damit sie beim
   Aktivieren nicht wieder verloren geht.
3. **ANLEITUNG.md** (diese Datei) und ein Fork-Hinweis oben in der README.

Sonst ist nichts verändert: Agenten, Skills, Kommandos und
Managed-Agent-Vorlagen sind Wort für Wort das Original. Geprüft mit
`python3 scripts/check.py` (83 Dateien, 0 Probleme).

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
