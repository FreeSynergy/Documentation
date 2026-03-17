# Pakete

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Rollen](rollen.md)

---

## Paket-Typen

| Typ | Inhalt | Beispiele |
|---|---|---|
| `program` | FreeSynergy-Kernprogramm | Node, Desktop, Conductor |
| `container` | Service-Modul (Quadlet + Config) | Kanidm, Forgejo, Outline |
| `group` | Meta-Paket, bündelt andere Pakete | server-minimal, desktop-full |
| `language` | Sprach-Snippets (.ftl) | Deutsch, Französisch |
| `theme` | Visuelles Theme | Midnight Blue, Nordic |
| `widget` | Desktop-Widget | Uhr, System-Info |
| `bot` | Bot-Definition | Broadcast, Gatekeeper |
| `bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `task` | Automatisierungs-Template | "Docs ins Wiki" |

**Libraries** (`fsn-*` Crates) sind KEINE eigenständigen Pakete. Sie sind Abhängigkeiten die mit den Programmen mitkommen. Sie leben in einem shared-Ordner, Git handled das.

## Paket-Lifecycle

```
Browse/Suche im Store
  → Installieren (Download + Signatur-Check + Abhängigkeiten + Scripts)
    → Konfigurieren
      → Nutzen
        → Update (neue Version, alte optional behalten)
          → Rollback (vorherige Version)
            → Deinstallieren ("Daten behalten?")
              → Vollständig löschen (alles weg)
```

## Pflicht-Metadaten

Jedes Paket MUSS haben:
- `id` (einzigartiger Name, KEIN Typ-Prefix)
- `name` (Anzeigename)
- `version` (SemVer, aus Git-Tag)
- `type` (program, container, group, language, theme, widget, bot, bridge, task)
- `description` (Kurzbeschreibung)
- `tags` (für Suche — muss aussagekräftig sein)
- `icon` (SVG oder Icon-Name; PFLICHT, wenn fehlt → generisches Icon)

**Jedes Paket ist ein Objekt.** Überall wo es angezeigt wird sieht man Icon, Name, Version, Tags.

## Tags

Tags sind das primäre Suchinstrument. **Schlechte Tags = Paket unsichtbar.**

Details und Filter-Syntax: [Store](../programme/store/README.md#tag-system)

## Versionierung

SemVer + Git-Tags. Parallele Versionen möglich (installiert mit Versionsnummer). Standard: latest + alte löschen. Release-Channels: stable, testing, nightly.

Details: [Store](../programme/store/README.md#versionierung)

## Signierung

ed25519 als Default, austauschbar (ed448, RSA). Signatur wird vor Installation geprüft.

Details: [Store](../programme/store/README.md#paket-signierung)

## Scripts

| Script | Wann |
|---|---|
| `pre_install` | Vor Installation (Voraussetzungen) |
| `post_install` | Nach Installation (Setup, Konfiguration) |
| `pre_remove` | Vor Deinstallation (Warnung, Aufräumen) |
| `post_remove` | Nach Deinstallation (Cleanup) |

## Abhängigkeiten

```toml
[dependencies]
fsn-node = ">= 0.5.0"

[optional-dependencies]
kanidm = ">= 1.4.0"
```

Der Store löst Abhängigkeiten automatisch auf.

## Variable-System (für Container-Pakete)

Variablen haben Basis-Typen und Rollen. Siehe [Conductor](../programme/conductor/README.md) für die Analyse und [Rollen](rollen.md) für die Hierarchie.

Secrets werden mit `age` verschlüsselt. Rollen-typisierte Variablen werden automatisch befüllt wenn ein passender Service installiert ist.

---

Weiter: [Store](../programme/store/README.md) | [Conductor](../programme/conductor/README.md) | [Rollen](rollen.md)
