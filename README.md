# Lehrer-Agent

KI-Agent mit spezialisierten Skills für Lehrkräfte an allgemeinbildenden Schulen. Unterricht planen, Tests erstellen, Material sammeln, Verwaltungsvorschriften nachschlagen. Alle Ausgaben (Hefteinträge, Tests, Arbeitsblätter) werden als **.docx** (Word) erzeugt.

- **Schulbücher**: Greift auf Offline-Bücher von Westermann (BiBox), Klett und Cornelsen zu — Buchtexte direkt als Volltext, kein OCR
- **Rahmenlehrpläne**: Aller 16 Bundesländer — bereits enthalten
- **Verwaltungsvorschriften**: Schulgesetze, Verordnungen und Erlasse aller 16 Bundesländer — bereits enthalten

Der Agent ist als [Claude Code Plugin](https://docs.anthropic.com/en/docs/claude-code/plugins) paketiert. Die Skills bestehen aus Markdown-Dateien und funktionieren mit jeder KI, die Dateien als Kontext lesen kann.

**Empfohlene KI**: Claude 4.6 (Opus oder Sonnet)

## Voraussetzungen

- **[`uv`](https://docs.astral.sh/uv/)** — wird für Word-Generierung (.docx) und Buch-Extraktion benötigt. Wird vom Agent automatisch installiert falls nicht vorhanden.
- **Schulbücher als Offline-Daten** — der Agent arbeitet direkt mit den Buchtexten. Ohne Bücher fehlt die fachliche Grundlage für Unterrichtsvorbereitung, Tests und Materialsuche. Mindestens eine der folgenden Apps installieren und Bücher offline herunterladen:
  - BiBox 2.0 (`https://bibox2.westermann.de`)
  - Klett Lernen (`https://www.klett.de/inhalt/klett-lernen/158307`)
  - Cornelsen Offline Lernen: macOS (`https://ebook.cornelsen.de/uma20/public/v2/uma/offline/mac`) / Windows (`https://ebook.cornelsen.de/uma20/public/v2/uma/offline/win`)
- **Node.js** (optional) — nur für Claude Code CLI, wenn Browser-Zugriff auf Verlags-Plattformen benötigt wird (Playwright MCP). In Cowork nicht nötig — dort ist der Browser eingebaut.

## Nutzung

### Claude Cowork / OpenWork (empfohlen)

**Methode 1 — Marketplace:**

1. Claude Desktop-App öffnen → oben auf **Cowork** wechseln
2. Plugin-Sidebar → **"+"** → **Add marketplace from GitHub**
3. Eingeben: `lehrer-tools/marketplace` → **Sync**
4. **lehrer-agent** auswählen → **Install**

**Methode 2 — ZIP Upload:**

1. [ZIP herunterladen](https://github.com/lehrer-tools/lehrer-agent/archive/refs/heads/main.zip)
2. Claude Desktop-App öffnen → oben auf **Cowork** wechseln
3. Plugin-Sidebar → **"+"** → **Upload** → ZIP-Datei auswählen

**Methode 3 — Ordner öffnen:**

1. [ZIP herunterladen](https://github.com/lehrer-tools/lehrer-agent/archive/refs/heads/main.zip) und entpacken
2. Claude Desktop-App öffnen → oben auf **Cowork** wechseln
3. **Work in a folder** → den entpackten `lehrer-agent` Ordner wählen

Skills per Chat aufrufen:

```text
/lehrer-agent:unterricht Mathe Einsetzungsverfahren
/lehrer-agent:leistungskontrolle
/lehrer-agent:material Physik Optik Klasse 8
/lehrer-agent:vv Wie viele Klassenarbeiten pro Halbjahr?
```

### Claude Code (CLI / IDE)

```bash
# Plugin aus Marketplace installieren
claude plugin marketplace add lehrer-tools/marketplace
claude plugin install lehrer-agent

# Oder: Repo klonen und direkt nutzen
git clone https://github.com/lehrer-tools/lehrer-agent.git
cd lehrer-agent
claude
```

Alternativ: [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=anthropics.claude-code) oder [JetBrains Extension](https://plugins.jetbrains.com/plugin/claude-code) installieren und `lehrer-agent/` als Workspace öffnen.

### Ohne Claude

Funktioniert mit jeder KI, die Dateien als Kontext lesen kann:

1. [ZIP herunterladen](https://github.com/lehrer-tools/lehrer-agent/archive/refs/heads/main.zip) und entpacken
2. Den gewünschten Skill-Ordner (z.B. `skills/unterricht/`) in dein Projektverzeichnis oder nach `~/.claude/skills/` kopieren
3. `config/` für das Lehrer-Profil kopieren
4. Den Inhalt von `CLAUDE.md` in dein eigenes `CLAUDE.md` einfügen
5. Die Skills funktionieren dann als Kontext — die KI liest die SKILL.md und arbeitet danach

## Konfiguration

### Ersteinrichtung

Beim ersten Start fragt der Agent nach:

| Einstellung | Beispiel | Beschreibung |
|---|---|---|
| `bundesland` | Brandenburg | Bundesland der Schule |
| `schultyp` | Oberschule | Schultyp (Förderschule, Oberschule, Gesamtschule, Gymnasium) |
| `schulstufe` | Sekundarstufe I | Schulstufe (Primarstufe, Sekundarstufe I, Sekundarstufe II) |

### Lehrer-Profil (`config/lehrer.md`)

Der Agent lernt automatisch aus den Interaktionen und speichert Präferenzen in `config/lehrer.md`:

- Fächer und Kurse (A/B), Klassenstufen
- Unterrichtsstil (Merke-Text-Länge, Beispielquellen, Hausaufgaben)
- Leistungskontrolle (Testdauer, Aufgabenformate)
- Verwendete Lehrbücher
- Dokumentvorlagen (Kopfzeilen, Formatierung)

Die Datei wird bei jeder Nutzung gelesen und bei neuen Erkenntnissen aktualisiert.

### Dokumentvorlagen (`config/vorlagen/`)

Benutzerdefinierte Vorlagen für Hefteinträge, Tests und Arbeitsblätter.

## Skills

### Unterricht (`/lehrer-agent:unterricht`)

Bereitet Unterrichtsstunden vor: Analysiert Buchmaterial aus `books/`, schlägt Varianten für Merke-Texte, Beispiele und Aufgaben vor, generiert Hefteinträge als .docx.

- Lädt den schulinternen Lehrplan (oder fragt danach)
- Berücksichtigt bundeslandspezifische Rahmenlehrpläne (16 Bundesländer)
- Fachdidaktische Referenzen für Mathematik, Deutsch, Englisch, Physik, Chemie
- Förderschwerpunkt-Anpassungen (Hören, ESE, Lernen)

```
/lehrer-agent:unterricht Mathe Einsetzungsverfahren
/lehrer-agent:unterricht Englisch Simple Past
```

### Leistungskontrolle (`/lehrer-agent:leistungskontrolle`)

Erstellt Tests, Klassenarbeiten und TÜ als .docx mit A-Kurs/B-Kurs-Differenzierung, echten Word-Formeln (OMML), Rechenkästchen und Koordinatensystemen.

- Erweiterte Abfrage: Thema, Buchseiten, Zeitraum, geübte Aufgabentypen, Klassenschwächen
- Individuelle Variante (IND) für Förderbedarf
- Anforderungsbereiche I/II/III nach Lehrplan

### Material (`/lehrer-agent:material`)

Sammelt Unterrichtsmaterial aus verschiedenen Quellen:

- Lokale Bücher (`books/`)
- BiBox Desktop + Online
- Verlags-Plattformen (Klett, Cornelsen — konfigurierbar in `portale.md`)
- Freie Quellen (Serlo, GeoGebra, PhET, Planet Schule, ZUM-Wiki)

### VV (`/lehrer-agent:vv`)

Beantwortet Fragen zu Verwaltungsvorschriften und Schulrecht aller 16 Bundesländer. Lokale Volltexte für alle 16 Bundesländer.

**Wichtig**: Trifft niemals Annahmen. Wenn die Antwort lokal nicht gefunden wird, wird online gesucht. Jede Aussage mit Paragraph und Quelle belegt.

## Tools

Verlags-Extraktionstools stehen über `bin/` im PATH zur Verfügung. Sie extrahieren Offline-Bücher aus Verlags-Apps in PDF + Markdown.

### bibox

Extrahiert Schulbücher aus der **BiBox 2.0** Desktop-App (Westermann-Gruppe). Entschlüsselt AES-256-CTR-verschlüsselte Offline-Bücher, erzeugt durchsuchbare PDFs mit Text-Overlay und Markdown.

**Voraussetzung**: BiBox 2.0 Electron-App installiert, Bücher offline synchronisiert.

```bash
bibox                        # Alle Bücher extrahieren
bibox --book <id>            # Einzelnes Buch
bibox --no-materials         # Ohne Zusatzmaterialien
bibox --force                # Bestehende Dateien überschreiben
```

### klett

Extrahiert Schulbücher aus der **Klett Lernen** Desktop-App (Klett-Gruppe). Bücher sind nicht verschlüsselt. Extrahiert ePaper-PDFs, Volltext aus Suchindex und Zusatzmaterial (Kopiervorlagen, Tafelbilder).

**Voraussetzung**: Klett Lernen App installiert, Bücher offline heruntergeladen.

```bash
klett                        # Alle Bücher extrahieren
klett --book <DUA-ID>        # Einzelnes Buch
klett --markdown             # Volltext als .md statt .txt
klett --no-materials         # Ohne Zusatzmaterialien
klett --force                # Bestehende Dateien überschreiben
```

### cornelsen

Extrahiert Schulbücher aus der **Cornelsen Offline Lernen** Desktop-App (Cornelsen-Gruppe). Entschlüsselt AES-128-CBC-verschlüsselte PDFs, extrahiert Volltext per PyMuPDF und konvertiert Zusatzmaterial (Tipps, PDFs) nach Markdown.

**Voraussetzung**: Cornelsen Offline Lernen App installiert, Bücher offline heruntergeladen.

```bash
cornelsen                    # Alle Bücher extrahieren
cornelsen --book <id>        # Einzelnes Buch
cornelsen --markdown         # Volltext als .md statt .txt
cornelsen --no-materials     # Ohne Zusatzmaterialien
cornelsen --force            # Bestehende Dateien überschreiben
```

### Workflow

Lehrer nennt Fach/Buch → Agent prüft ob bereits in `books/` vorhanden → falls nicht: extrahiert per `bibox`, `klett` oder `cornelsen`.

## Projektstruktur

```
lehrer-agent/
  .claude-plugin/plugin.json    Plugin-Manifest + userConfig (Claude Code Kompatibilität)
  CLAUDE.md                     Agent-Persona, Routing, Konventionen
  .mcp.json                     MCP-Server (Playwright, Perplexity)
  bin/
    bibox                       BiBox-Extraktionstool (Westermann)
    klett                       Klett-Extraktionstool (Klett)
    cornelsen                   Cornelsen-Extraktionstool (Cornelsen)
  config/
    lehrer.md                   Selbstlernendes Lehrer-Profil
    vorlagen/                   Benutzerdefinierte Dokumentvorlagen
  books/                        Extrahierte Schulbücher (gitignored)
  skills/
    unterricht/                 Unterrichtsvorbereitung
      SKILL.md
      lehrplan.md               Lehrplan-Quellenübersicht
      didaktik/                 Fachdidaktische Referenzen
        lehrplan/               Rahmenlehrpläne (16 Bundesländer)
          schulintern/          Schulinterne Lehrpläne
        foerderschwerpunkt/     Hören, ESE, Lernen
      scripts/                  Word-Generierung (template.py, omml_utils.py)
    leistungskontrolle/         Testerstellung
      SKILL.md
      reference/                Aufgabentypen, Lehrplan
      scripts/                  Word-Generierung
    material/                   Materialsammlung
      SKILL.md
      portale.md                Konfigurierbare Verlags-/Web-Portale
    vv/                         Verwaltungsvorschriften
      SKILL.md
      quellen/
        brandenburg/            7 VV-Texte (Sek I-V, Leistungsbewertung, ...)
        berlin/                 SchulG, Sek I-VO
        bayern/                 BayEUG, BaySchO
        nrw/                    SchulG, APO-S I, APO-GOSt
        sachsen/                SchulG, SOOSA, SOGS, SOGY
        niedersachsen/          NSchG, Erlasse, Zeugnisse
        bremen/                 SchulG, Zeugnisverordnung
        rlp/                    SchulG, SchulO
        hessen/                 VOGSV, OAVO
        bw/                     SchG, NVO
        thueringen/             SchulG, SchulO
        sachsen-anhalt/         SchulG LSA, Leistungsbewertung Sek
        hamburg/                HmbSG
        mv/                     SchulG M-V, LeistBewVO
        sh/                     SchulG SH, Schulrecht-Grundlagen
        saarland/               SchoG, ASchO
        urls_pro_bundesland.md  Rechtsportale aller 16 Bundesländer
```

## Bücher hinzufügen

1. **BiBox**: `bibox --book <id>`
2. **Klett**: `klett --book <DUA-ID>`
3. **Cornelsen**: `cornelsen --book <id>`
4. **PDF manuell**: PDF in `books/<Buchname>/` ablegen, per `pdftotext` in Markdown umwandeln
5. **Playwright**: Agent kann Login-pflichtige Verlags-Plattformen per Browser ansteuern

## Standalone-Repos

Die Verlags-Tools gibt es auch als eigenständige Projekte (ohne Agent/Plugin):

- [bibox-to-pdf-md](https://github.com/lehrer-tools/bibox-to-pdf-md) — BiBox 2.0 (Westermann)
- [klett-to-pdf-md](https://github.com/lehrer-tools/klett-to-pdf-md) — Klett Lernen
- [cornelsen-to-pdf-md](https://github.com/lehrer-tools/cornelsen-to-pdf-md) — Cornelsen Offline Lernen

## Lizenz

MIT
