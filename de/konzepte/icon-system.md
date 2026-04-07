# Icon-System

[← Zurück zum Index](../INDEX.md) | [Icon-Baum](icon-baum.md) | [Themes](themes.md) | [UI-Standards](ui-standards.md)

---

## Warum ein eigenes Icon-System?

Icons sind Sprache. Ein Icon das für "Einstellungen" steht, sollte überall dasselbe bedeuten —
egal ob es in einem Menü, in einem Formularfeld oder als Programm-Icon erscheint.

FreeSynergy standardisiert Icons auf drei Ebenen:

1. **Welches Icon wofür** — einheitliche Bedeutung über alle Programme
2. **Wie Icons kombiniert werden** — das Dual-Icon-Prinzip
3. **Wie sie dargestellt werden** — konfigurierbare Layouts

---

## Das Dual-Icon-Prinzip

Jede "Sache" in FreeSynergy kann **zwei Icons** haben:

| Rolle | Frage | Technischer Key |
|---|---|---|
| **Action-Icon** | "Was mache ich?" | `action` |
| **Identity-Icon** | "Wer bin ich?" | `identity` |

Zusammen ergeben beide Icons einen vollständigen Kontext — ohne Text.

### Beispiele

| Situation | Action-Icon | Identity-Icon | Bedeutung |
|---|---|---|---|
| Desktop → Einstellungen → Sprache | `language` | `settings` | Spracheinstellung ändern |
| Browser mit Firefox-Engine | `browser` | `firefox` | Ich bin ein Browser, meine Engine ist Firefox |
| Namensfeld einer Person | `name` | `person` | Ich bin der Name einer Person |
| Namensfeld eines Ortes | `name` | `place` | Ich bin der Name eines Ortes |
| Home-Button in der Navigation | `home` | `navigation` | Zurück zur Startseite |
| Home-Ordner im Dateisystem | `folder` | `home` | Mein Home-Verzeichnis |

### Das Action-Icon ist primär

Das Action-Icon steht immer im Vordergrund — es sagt was passiert oder was das Ding tut.
Das Identity-Icon liefert den Kontext — wo bin ich, zu wem gehöre ich.

Ein Icon bleibt immer dasselbe Icon, unabhängig davon ob es als Action oder als Identity
verwendet wird. Bedeutung entsteht durch Position, nicht durch verschiedene Icons.

### Identity ist optional

Nicht jedes Element braucht zwei Icons. Ein einfacher "Löschen"-Button braucht kein Identity-Icon.
Erst wenn der Kontext mehrdeutig ist oder der Nutzer schnell verstehen soll wo er sich befindet,
lohnt sich das zweite Icon.

---

## Icon-Sets

Ein Icon-Set ist eine austauschbare Sammlung von SVG-Dateien.
Alle Sets implementieren dieselbe Schnittstelle — der Nutzer kann Sets im Desktop wechseln.

Sets können verschiedene Stile haben:
- Flache SVGs (Standard)
- Animierte SVGs
- Icons mit 3D-Effekt
- Monochrom / Farbig

Sets werden im Repo `FreeSynergy/fs-icons` verwaltet.
Der Nutzer wählt sein aktives Set in den Desktop-Einstellungen unter **Icons**.

---

## Layout-System

Das Layout bestimmt wie Action-Icon und Identity-Icon zusammen dargestellt werden.

### Standard-Layouts

#### Default

Das Action-Icon ist groß, das Identity-Icon hat die Hälfte des Radius.
Der Mittelpunkt des Identity-Icons liegt am äußersten Rand des Action-Icons,
beide sind nach unten bündig ausgerichtet (bottom-aligned).
Das Identity-Icon ist dadurch leicht nach rechts versetzt, aber unten bündig.

```
┌─────────────┐
│             │
│  Action     │
│  (groß)   ┌─┤
└───────────┤ │Identity
            └─┘
```

#### Side-by-Side

Beide Icons gleich groß. Das Action-Icon ist vorne/links,
das Identity-Icon ist um 1/3 überlappend versetzt dahinter/rechts.
Das Action-Icon liegt in der vorderen Z-Ebene.

```
┌─────────┐
│         ┌─────────┐
│ Action  │Identity │
│         └─────────┘
└─────────┘
```

#### Overlay

Beide Icons gleich groß, direkt übereinander zentriert.
Das Action-Icon liegt oben (volle Deckkraft), das Identity-Icon leicht transparent dahinter.

### Custom-Layout

Der Nutzer kann eine eigene Schablone definieren. Dabei sind einstellbar:

| Parameter | Möglichkeiten |
|---|---|
| Position | X/Y-Offset relativ zum Zentrum |
| Größe | Eigener Radius je Icon |
| Z-Ebene | Vorne / Hinten |
| Ausrichtung | Top / Center / Bottom / Left / Right |

Custom-Layouts können gespeichert und benannt werden.
Sie sind in den Desktop-Einstellungen unter **Icons → Layout** konfigurierbar.

---

## Einzahl und Mehrzahl

Icons unterscheiden zwischen Einzahl und Mehrzahl wo es semantisch einen Unterschied macht.

### Wann Einzahl/Mehrzahl sinnvoll ist

| Einzahl | Mehrzahl | Unterschied |
|---|---|---|
| `person` | `people` | Ein Nutzer vs. Gruppe/Team |
| `file` | `files` | Eine Datei vs. Datei-Auswahl / Batch |
| `message` | `chat` | Eine Nachricht vs. Konversationsverlauf |
| `package` | `packages` | Ein Paket installieren vs. Paketverwaltung |
| `task` | `tasks` | Eine Aufgabe vs. Aufgabenliste |

### Wann kein Plural nötig ist

Konzepte die keinen sinnvollen Plural haben oder wo der Kontext ausreicht:
`home`, `settings`, `server`, `calendar`, `search`, `filter` — diese existieren nur in Einzahl.

**Regel:** Plural nur wenn das UI zwischen "einem Objekt" und "mehreren Objekten" unterscheiden muss
und der Nutzer diesen Unterschied sehen soll.

---

## Übersetzung (i18n)

Jedes Icon hat FTL-Schlüssel in `fs-i18n`. Damit hat jedes Icon einen Namen und eine Beschreibung
in jeder unterstützten Sprache — unabhängig davon wie es heißt oder aussieht.

### Key-Schema

```
icon-{name} = Anzeigename
icon-{name}-description = Beschreibung wann/wofür dieses Icon eingesetzt wird
```

### Beispiel (`fs-i18n/locales/de/icons.ftl`)

```ftl
icon-language = Sprache
icon-language-description = Anzeigesprache auswählen oder die Sprache einer Information

icon-settings = Einstellungen
icon-settings-description = Konfigurationsbereich eines Programms oder Objekts

icon-person = Person
icon-person-description = Eine einzelne Person, ein Nutzer-Profil

icon-people = Personen
icon-people-description = Eine Gruppe von Personen, ein Team

icon-home = Zuhause
icon-home-description = Das persönliche Home-Verzeichnis oder die Startseite
```

Die Übersetzung gilt sowohl für das Icon als auch für den Tooltip wenn ein Icon in der UI erscheint.

---

## Wo Icons eingesetzt werden

### Programme

Jedes Programm hat ein Icon. Das ist heute Standard.
Das Programm-Icon erscheint im App-Launcher, in der Taskleiste und im Store.

### Menüpunkte

Jeder Menüpunkt in einem Programm hat ein Icon.
Das zwingt zur Konsistenz: Gleiche Aktion = gleiches Icon, überall.

Wenn zwei Programme denselben Menüpunkt haben (z.B. "Einstellungen"), verwenden sie dasselbe Icon.
Das schult die Mustererkennung des Nutzers — er findet Dinge schneller.

### Formularfelder

Felder profitieren vom Dual-Icon besonders:
Ein `name`-Feld kann für eine Person, einen Ort, ein Programm oder einen Server stehen.
Das Action-Icon (`name`) ist immer gleich, das Identity-Icon (`person`, `place`, `app`, `server`)
erklärt den Kontext — ohne Beschriftung.

### Objekte (FsObjects)

Jedes FsObject hat ein Icon. Da Objekte auch die Grundlage für Masken/Forms bilden,
gleichen sich Objekt-Icons und Formular-Icons an:

- Das Objekt "Person" und das Formular "Personendaten" haben dasselbe Identity-Icon
- Das Programm "Browser" und das Objekt "Browser-Session" haben dasselbe Icon
- Adapter-Icons (Firefox, Kanidm) erscheinen als Identity-Icon wenn ein Objekt diese Engine nutzt

---

## Corporate Design — Grundprinzipien

1. **Ein Icon, eine Bedeutung** — ein Icon steht immer für dasselbe Konzept
2. **Ontologie vor Verwendung** — Icons werden nach dem geordnet was sie sind, nicht wie sie benutzt werden
3. **Konsistenz erzwingt Verständnis** — Entwickler und Nutzer lernen das System einmalig
4. **Erklärbarkeit** — jedes Icon hat eine dokumentierte Begründung warum es so aussieht und wo es steht
5. **Sets tauschen, Bedeutung bleibt** — egal welches Set aktiv ist, `language` bleibt `language`

---

Weiter: [Icon-Baum](icon-baum.md) | [Themes](themes.md) | [UI-Standards](ui-standards.md) | [Icon Manager](../programme/icons/README.md)
