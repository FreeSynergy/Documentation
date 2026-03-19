# Synthesizer — Beschreibung statt Klicken

[← Zurück zum Index](../INDEX.md) | [Rollen-System](rollen.md) | [Cursor Manager](../programme/icons/cursor-manager.md) | [Icon Manager](../programme/icons/README.md)

---

## Was ein Synthesizer ist

Ein Synthesizer ist ein Dienst mit der Rolle `synthesizer`. Er nimmt eine **Beschreibung** und gibt **strukturierte Ausgabe** zurück.

Kein "KI", kein "AI" — weil beides mehr Fragen aufwirft als es beantwortet. Ein Synthesizer beschreibt genau was er tut: er **synthetisiert** Ausgabe aus Eingabe. Rohmaterial rein, fertiges Set raus.

In FSN wird der Synthesizer überall dort eingesetzt, wo ein Nutzer etwas erstellen soll:
- **Cursor-Sets** (Mauszeiger)
- **Icon-Sets**
- **Themes** (Farb-Schemata, Schriften)
- **Container-App-Konfigurationen**
- **Package-Beschreibungen**
- Überall sonst, wo ein Formular ausgefüllt werden muss

---

## Rollen

```
synthesizer
├── synthesizer.text        — erzeugt Freitext (Beschreibungen, Dokumentation)
├── synthesizer.structured  — erzeugt strukturierte Daten (TOML/JSON) → Manager-Integration
├── synthesizer.image       — erzeugt Bilder / SVG-Grafiken
└── synthesizer.code        — erzeugt Code (Skripte, Konfigurationen)
```

Die **Manager-Integration** setzt `synthesizer.structured` voraus. Nur wer strukturierte Daten zurückgeben kann, kann ein Formular vorausfüllen.

Ein lokales Sprachmodell (z.B. Ollama mit einem kleinen Modell) kann `synthesizer.text` erfüllen. Ein größeres Modell, das TOML-Strukturen zuverlässig ausgibt, erfüllt zusätzlich `synthesizer.structured`.

---

## Wie die Integration in jedem Manager funktioniert

### Voraussetzung

Ein Dienst mit Rolle `synthesizer.structured` muss installiert und aktiv sein.

### Button-Erscheinung

Wenn die Voraussetzung erfüllt ist, erscheint in jedem Erstellungs-Formular (neues Set, neue Konfiguration, neues Paket) ein Button im Header:

```
┌──────────────────────────────────────────────────────┐
│  Neues Cursor-Set erstellen          [✦ Synthesizer]  │
├──────────────────────────────────────────────────────┤
│  Name: ___________                                    │
│  ...                                                  │
```

Der Button ist sichtbar aber subtil — kein aufdringlicher Hinweis. Er ist da wenn man ihn braucht.

### Eingabe-Dialog

Klick auf **[✦ Synthesizer]** öffnet ein Modal:

```
┌─────────────────────────────────────────────────────────┐
│  Mit Synthesizer erstellen                         [×]  │
├─────────────────────────────────────────────────────────┤
│  Beschreibe was du möchtest:                            │
│  ┌───────────────────────────────────────────────────┐  │
│  │ Schmale, minimalistische Linien. Weiß auf         │  │
│  │ transparentem Grund. Cyberpunk-Stil, keine        │  │
│  │ Serifen, präzise Spitzen.                         │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  Referenz-Set:  [Kein Referenz-Set ▼]                   │
│  Anzahl Vorschläge:  [5 ▼]  (einstellbar in Settings)   │
│                                                         │
│  [Abbrechen]                    [Vorschläge generieren] │
└─────────────────────────────────────────────────────────┘
```

- **Referenz-Set** (optional): Der Synthesizer bekommt ein bestehendes Set als Stil-Vorlage
- **Anzahl Vorschläge**: Konfigurierbar, Standard 5 — in den Manager-Einstellungen änderbar

### Vorschläge-Ansicht

Nach der Generierung erscheint eine Liste der Vorschläge:

```
┌─────────────────────────────────────────────────────────┐
│  5 Vorschläge generiert                                 │
├─────────────────────────────────────────────────────────┤
│  ☐  Vorschlag 1 — "Cyber Minimal"                       │
│     [▶ default] [▶ pointer] [▶ text] [▶ busy] ...       │
│                                                         │
│  ☐  Vorschlag 2 — "Neon Edge"                           │
│     [▶ default] [▶ pointer] [▶ text] [▶ busy] ...       │
│                                                         │
│  ☐  Vorschlag 3 — "Sharp Line"                          │
│     [▶ default] [▶ pointer] [▶ text] [▶ busy] ...       │
│  ...                                                    │
├─────────────────────────────────────────────────────────┤
│  [Alle auswählen]  [Auswahl leeren]                     │
│  [Abbrechen]              [Auswahl ins Formular →]      │
└─────────────────────────────────────────────────────────┘
```

- Jeder Vorschlag zeigt eine Vorschau der wichtigsten Elemente
- Per **Checkbox** ein oder mehrere auswählen
- Mehrere Vorschläge können übernommen werden: das Formular bekommt den ersten; die anderen werden als Varianten gespeichert (oder: der Nutzer entscheidet pro Slot welche Variante er nimmt)

### Formular-Übernahme

Nach "Auswahl ins Formular →":
- Das Modal schließt sich
- Das Formular ist vorausgefüllt
- Der Nutzer kann jeden Wert noch ändern, ersetzen, ergänzen
- Der Synthesizer-Vorschlag ist **kein Endergebnis** — er ist ein Ausgangspunkt

---

## Was pro Manager als Kontext übergeben wird

Jeder Manager sendet dem Synthesizer neben der Nutzer-Beschreibung zusätzlichen Kontext:

### Cursor Manager

```toml
[context]
type        = "cursor-set"
slots       = ["default", "pointer", "text", ...]   # alle 31 Slots mit Bedeutung
style_ref   = "midnight"                             # optional: Referenz-Set-ID
```

### Icon Manager

```toml
[context]
type        = "icon-set"
style       = "outline"      # outline | filled | duotone | ...
size        = 24             # Standard-Größe in px
```

### Theme Manager

```toml
[context]
type        = "theme"
base        = "dark"         # dark | light
primary     = "#00bcd4"      # optional: Hauptfarbe vorgeben
```

### Container App Manager (Konfigurationsvorschlag)

```toml
[context]
type        = "container-config"
image       = "nextcloud"
variables   = ["NEXTCLOUD_ADMIN_USER", "NEXTCLOUD_DB_HOST", ...]
```

---

## Einstellungen

In den Einstellungen jedes Managers gibt es einen **Synthesizer**-Abschnitt:

| Einstellung | Standard | Beschreibung |
|---|---|---|
| `preferred_synthesizer` | (erster verfügbarer) | Welcher Synthesizer-Dienst bevorzugt wird |
| `suggestion_count` | `5` | Wie viele Vorschläge generiert werden |
| `auto_fill_single` | `true` | Bei nur 1 Vorschlag: direkt ins Formular, kein Modal |
| `show_button` | `true` | Button anzeigen / verstecken |

---

## Warum dieser Name

"KI" und "AI" beschreiben eine Technologie-Kategorie — nicht eine Funktion. Ein Synthesizer beschreibt was er **tut**: er synthetisiert. Er nimmt eine Beschreibung und erzeugt strukturierte Ausgabe. Das ist die einzige Sache die er für FSN tut.

Außerdem ist "Synthesizer" ein bekannter Begriff — jeder der je einen Synthesizer gehört hat, weiß dass Rohmaterial (ein paar Noten, ein Patch) in etwas Fertigem endet. Das ist exakt die Metapher.

---

## Zusammenfassung

| Frage | Antwort |
|---|---|
| Wann erscheint der Button? | Wenn ein Dienst mit `synthesizer.structured` aktiv ist |
| Welche Manager profitieren? | Alle die Erstellungs-Formulare haben: Cursor, Icons, Theme, Container App |
| Ist der Vorschlag bindend? | Nein — immer nur ein Ausgangspunkt, alles editierbar |
| Kann man mehrere Vorschläge nehmen? | Ja — Checkboxen, dann Übernahme ins Formular |
| Wo konfigurieren? | Manager-Einstellungen → Synthesizer-Abschnitt |

---

Weiter: [Rollen-System](rollen.md) | [Cursor Manager](../programme/icons/cursor-manager.md) | [Icon Manager](../programme/icons/README.md)
