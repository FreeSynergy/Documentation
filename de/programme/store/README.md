# Store — Das Wissen & Der Paketmanager

[← Zurück zum Index](../../INDEX.md)

---

## Was der Store ist

Der Store ist der **universelle Paketmanager** von FreeSynergy. Vergleichbar mit dnf/apt, aber flexibler. Er verwaltet ALLES: Programme, Services, Themes, Sprachen, Widgets, Bots, Tasks.

Gleichzeitig ist er ein **Wissensspeicher** der allen Programmen hilft — der Container Manager fragt "Kennst du diesen Service?" und der Store liefert Metadaten, Rollen, Variablen-Typen.

## Eigenständigkeit

Der Store funktioniert **ohne jedes andere FreeSynergy-Programm**. Er ist ein Git-Repository. CLI, API und GUI bieten identische Funktionen.

## Der Bootstrap-Prozess

```
FreeSynergy.Init (minimales Binary)
  → Klont den Store via gitoxide
    → Store installiert ALLES:
       Node, Desktop, Container Manager, Services, Themes, Sprachen, ...
```

Siehe [Init](../init/README.md).

---

## Paket-Typen

Es gibt keine Kategorien (Server/App/Desktop) — nur **Typen**. Jedes Paket hat genau einen Typ. Tags + SysInfo regeln was auf dem aktuellen System installierbar ist.

| Typ | Inhalt | Beispiele |
|---|---|---|
| `app` | Native Rust-Binary (Cross-Platform) | Node, Desktop, Browser, Kanidm, Zentinel, Stalwart, Mistral |
| `container` | Container-App — runtime-agnostisch (Podman oder Docker) | Forgejo, Postgres, Outline, CryptPad |
| `bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `widget` | Desktop-Widget | Uhr, System-Info, Nachrichten |
| `language` | Shared Snippets (Mozilla Fluent) | Deutsch, Arabisch |
| `bot` | Bot-Definition | Broadcast, Gatekeeper |
| `task` | Automatisierungs-Template | "Docs ins Wiki" |
| `bundle` | Meta-Paket aus beliebigen Paketen | server-minimal, desktop-full |
| `bootstrap` | Sondertyp: Init-Binary zum Download (kein Install) | fs-init |
| `repo` | Store-Repository-Quelle — Installation registriert neue Paketquelle | freesynergy-community |
| `icon_set` | SVG-Icon-Sammlung — kann Default-Set überschreiben, shareable | freesynergy-default |

**`app` vs. `container`:**
- `app` = natives Rust-Binary, kein Container-Runtime nötig: Kanidm, Stalwart, Zentinel, Mistral
- `container` = läuft als Container-Image: Forgejo, Postgres, Outline — kein `requires:podman`-Tag nötig, der Node erkennt die verfügbare Runtime (Podman oder Docker) automatisch

**Plattform-Einschränkungen** über Tags + SysInfo:
- `platform:linux` — nur Linux
- `platform:linux+macos` — Linux und macOS
- `requires:systemd` — systemd muss laufen (automatisches Hinweisbanner wenn nicht vorhanden)

**Hinweis:** Libraries (`fs-*` Crates) sind KEINE eigenständigen Pakete. Sie sind Abhängigkeiten die mit den Anwendungen mitkommen.

Details zu Typen und Manifest-Felder: [Pakete](../../konzepte/pakete.md)

---

## Bundles (Meta-Pakete)

Bundles sind mehr als dnf groups — sie sind **rekursiv komponierbar** und können Pakete, andere Bundles und Capabilities referenzieren. Damit werden sie zu einer Art "Distribution" für FreeSynergy.

    [package]
    id   = "server"
    name = "Server"
    type = "bundle"
    tags = ["server", "node", "iam", "proxy"]

    # Konkrete Pakete (mit optionalem Version-Pin)
    [[bundle.packages]]
    id      = "fs-node"
    version = ">=0.5.0"

    [[bundle.packages]]
    id      = "zentinel"
    version = "1.2.3"
    pin     = true      # darf nicht gelöscht werden solange Bundle aktiv

    # Abstrakte Capabilities (welches Paket das erfüllt, ist egal)
    [[bundle.capabilities]]
    name = "iam"

    [[bundle.capabilities]]
    name = "database.postgres"

    # Andere Bundles (rekursiv!)
    [[bundle.bundles]]
    name = "management-tools"

    [[bundle.bundles]]
    name = "monitoring"

    # Optional
    [[bundle.optional]]
    id = "forgejo"

Ein Starter-Bundle für Anfänger referenziert einfach andere Bundles:

    [package]
    id   = "starter"
    name = "Starter"
    type = "bundle"

    [[bundle.bundles]]
    name = "server"

    [[bundle.bundles]]
    name = "desktop-full"

    [[bundle.bundles]]
    name = "essentials"

**Version-Pins:** `pin = true` blockiert das Löschen der gepinnten Version.
Erst wenn kein aktives Bundle mehr pinnt, kann sie entfernt werden.

Details: [Capabilities](../../konzepte/capabilities.md)

---

## Versionierung

### SemVer (MAJOR.MINOR.PATCH)

- **PATCH** (0.5.1 → 0.5.2): Bugfixes. API-kompatibel. Sicheres Update.
- **MINOR** (0.5.0 → 0.6.0): Neue Features. API-kompatibel. Sicheres Update.
- **MAJOR** (0.x → 1.0): Breaking Changes. Muss neu konfiguriert/installiert werden.

Jede Version = Git-Tag. Changelog aus Git-Commit-Messages. Rollback auf ältere Version jederzeit möglich.

### Parallele Versionen

Jedes Paket wird mit Versionsnummer installiert:

```
/opt/freesynergy/packages/kanidm-1.5.0/
/opt/freesynergy/packages/kanidm-1.4.0/   ← alte Version, noch da
```

**Standard:** `latest` installieren, vorherige Versionen löschen.
**Explizit:** Zwei Versionen parallel betreiben wenn gewünscht.

```bash
fsn store install kanidm                    # latest, alte löschen
fsn store install kanidm --version 1.4.0    # spezifische Version
fsn store install kanidm --keep-previous    # latest + alte behalten
```

### Release-Channels

| Channel | Zweck |
|---|---|
| `stable` | Getestet, empfohlen. Default. |
| `testing` | Neue Features, möglicherweise instabil |
| `nightly` | Aktuellster Stand, kann kaputt sein |

```bash
fsn store install kanidm --channel testing
fsn store config set default-channel stable
```

---

## Paket-Signierung

Jedes Paket wird signiert. **ed25519 als Default**, aber austauschbar.

```toml
[package.signature]
algorithm = "ed25519"          # Default, kann geändert werden
key_id = "freesynergy-main"
signature = "base64:..."
```

Der Store prüft die Signatur **vor** jeder Installation. Unsignierte Pakete werden nicht installiert (kann mit `--trust-unsigned` überschrieben werden, zeigt Warnung).

Unterstützte Algorithmen:
- ed25519 (Default)
- ed448
- RSA (Legacy-Kompatibilität)

---

## Paket-Metadaten (Pflicht)

```toml
[package]
id      = "kanidm"              # Eindeutiger Name (KEIN Typ-Prefix!)
name    = "Kanidm"              # Anzeigename
version = "1.4.2"               # SemVer, aus Git-Tag
type    = "app"                 # app, container, bundle, language, bot, task, bridge,
                                # widget, bootstrap, repo, icon_set
summary = "Identity and access management — OAuth2, OIDC, LDAP, SCIM, WebAuthn"
                                # max 255 Zeichen — Store-Karte, Suchergebnisse
icon    = "icon.svg"            # SVG-Datei (PFLICHT)
tags    = ["iam", "oidc", "scim", "mfa", "webauthn", "identity", "rust"]
author  = "Kanidm Project / FreeSynergy Fork"
license = "MPL-2.0"
```

Zusätzlich empfohlen:
```toml
description      = "Kanidm is a modern IAM server..."   # mittellang, für Detailansicht
description_file = "help/en/description.ftl"            # lang, internationalisierbar
```

### Drei Beschreibungsebenen

| Feld | Max | Wo |
|---|---|---|
| `summary` | 255 Zeichen | Store-Karte, Suchergebnisse, Sidebar |
| `description` | frei | Store-Detailansicht (inline im Catalog) |
| `description_file` | — | `.ftl`-Datei — Doku, Help-Seiten, internationalisierbar |

**Jedes Paket ist ein Objekt** mit Icon und Metadaten. Überall wo ein Paket angezeigt wird sieht man Icon, Name, Version, Tags.

Jedes Paket MUSS ein Icon haben. Kein Icon → generisches Icon für den Typ.

---

## Tag-System

Tags sind das **primäre Suchinstrument**. Schlechte Tags = Paket unsichtbar.

| Pakettyp | Tag-Regeln |
|---|---|
| `app` | Funktion, Plattform, `platform:linux` etc. |
| `container` | Rollen, Unterrollen, kompatible Standards |
| `language` | Sprach-Code, Region |
| `widget` | Funktion, Datenquelle |
| `bot` | Plattform, Funktion |
| `task` | Quell-Service, Ziel-Service, Funktion |
| `bundle` | Enthaltene Pakete, Zweck |
| `icon_set` | Stil, Farbe, Autor |
| `repo` | Betreiber, Vertrauensstufe |

### Filter-Kombinationen

```
tags contains "iam"                                    # Alle IAM-Services
type = "container" AND tags contains "rust"            # Rust-Container
type = "language" AND tags contains "de"               # Deutsche Pakete
tags contains "mail" OR tags contains "smtp"           # Alles rund um Mail
```

---

## Paket-Lifecycle

```
Browse/Suche im Store
  → Installieren (Download + Signatur-Check + Abhängigkeiten auflösen + Scripts)
    → Konfigurieren
      → Nutzen
        → Update (neue Version, alte optional behalten)
          → Rollback (auf vorherige Version wechseln)
            → Deinstallieren ("Daten behalten?")
              → Vollständig löschen (alles weg)
```

Jedes Paket kann Scripts haben:

| Script | Wann |
|---|---|
| `pre_install` | Vor der Installation (Voraussetzungen prüfen) |
| `post_install` | Nach der Installation (Konfiguration, Setup) |
| `pre_remove` | Vor der Deinstallation (Aufräumen, Warnung) |
| `post_remove` | Nach der Deinstallation (Cleanup) |

Die Scripts aus der alten `fsn-install.sh` werden aufgeteilt:
- SSH-Key-Setup → Script im `node`-Paket
- Podman/Quadlet-Setup → Script im `container-app`-Paket
- Firewall → Script im `zentinel`-Paket
- IAM-Bootstrap → Script im `kanidm`-Paket
- FCOS-spezifisches → Script im `fcos-support`-Paket

---

## Abhängigkeiten

Pakete können Abhängigkeiten deklarieren:

```toml
[dependencies]
fs-node = ">= 0.5.0"

[optional-dependencies]
kanidm = ">= 1.4.0"     # Empfohlen, aber nicht Pflicht
```

Der Store löst Abhängigkeiten automatisch auf (wie dnf). Bei Konflikten: Warnung + Benutzer entscheidet.

---

## Store als Wissensquelle

Programme fragen den Store nach Metadaten:

```
GET /api/store/know/service/kanidm
→ Rollen, Variablen-Typen, Ports, Healthcheck

GET /api/store/know/variable/REDIS_URL
→ Typ: url, Rolle: cache.redis

GET /api/store/know/compatible?role=iam
→ Welche installierten Services haben diese Rolle?
```

Der [Container Manager](../container/README.md) nutzt das bei der YAML-Analyse: "Kennst du diesen Service?" → Store ergänzt Daten, überschreibt NICHT.

---

## CLI-Interface

```bash
# Suche
fsn store search "mail"
fsn store search --type container --tag smtp
fsn store search --type bundle

# Info
fsn store info kanidm
fsn store info --version 1.4.0 kanidm

# Installieren
fsn store install kanidm
fsn store install kanidm --version 1.4.0
fsn store install kanidm --keep-previous
fsn store install kanidm --channel testing
fsn store install server-minimal              # Bundle

# Update
fsn store update kanidm
fsn store update --all

# Rollback
fsn store rollback kanidm                     # Vorherige Version

# Deinstallieren
fsn store remove kanidm
fsn store remove --keep-data kanidm
fsn store remove --purge kanidm               # Alles weg

# Installierte Pakete
fsn store list
fsn store list --type container
fsn store list --outdated

# Catalog
fsn store sync
fsn store sync --offline

# Cache
fsn store cache refresh    # Pakete neu vom Store laden (Metadaten aktualisieren)
fsn store cache clear      # Lokalen Cache vollständig leeren
```

## GUI — Store-Oberfläche im Desktop

### Ein Catalogue, Type-Sidebar, Filter-Leiste

Es gibt keine Server/App/Desktop-Tabs mehr. Der Store ist ein einziger Catalogue mit:

```
┌─────────────────────────────────────────────────────────────────┐
│  [Typ-Sidebar]   [Suchefeld]   [Repo ▼] [Status ▼] [Tags ▼]   │
├──────────┬──────────────────────────────────────────────────────┤
│  Alle    │                                                      │
│  App     │   Paketliste                                         │
│  Bridge  │   (Grid oder Liste, je nach Einstellung)            │
│  Widget  │                                                      │
│  Theme   │                                                      │
│  Sprache │                                                      │
│  Bot     │                                                      │
│  Task    │                                                      │
│  Bundle  │                                                      │
│  ──────  │                                                      │
│  Init    │                                                      │
│  ──────  │                                                      │
│  Settings│                                                      │
└──────────┴──────────────────────────────────────────────────────┘
```

**Init** ist ein Sonderfall in der Sidebar — kein installierbares Paket, nur ein Download-Eintrag (siehe [Init-Download-Eintrag](#init-download-eintrag)).

**Beim Wechsel** zwischen Typen gleitet der Inhalt animiert rein (Slide + Blur-Effekt).

### Filter-Leiste

Die Filter-Leiste sitzt oben, immer sichtbar:

| Filter | Optionen |
|---|---|
| **Suche** | Freitext — sucht in Name, Beschreibung, Tags |
| **Repo** | Alle Repositories / nur Repo X / nur Repo Y |
| **Status** | Alle / Installiert / Nicht installiert / Aktualisierbar |
| **Tags** | Multi-Select — klickbare Tag-Chips; Kombination möglich |
| **Platform** | Alle / Linux / macOS / Windows (blendet inkompatible aus) |

Filter kombinieren sich: `Typ=App` + `Status=Installiert` + `Tag=rust` zeigt nur installierte Rust-Apps.

### Repositories (Settings)

Der angepinnte **Settings**-Eintrag in der Sidebar führt zur Repository-Verwaltung:
- Alle konfigurierten Repositories werden aufgelistet
- Jedes Repository kann **aktiviert** oder **deaktiviert** werden
- Repositories können **hinzugefügt** oder **entfernt** werden
- **Ausnahme:** Das Haupt-Repository (`freesynergy-main`) kann nur deaktiviert, aber **nicht gelöscht** werden

### Icons

Jedes Paket **muss** ein Icon haben. Im Store-UI werden Icons überall angezeigt: in der Paketliste, in der Detailansicht, in der Installationsbestätigung. Fehlt ein Icon im Paket, wird das generische Icon für den jeweiligen Pakettyp verwendet.

### Schnell-Installation vs. Detailansicht

Von der Paketliste aus gibt es zwei Wege:

1. **Direkt installieren** — ein Klick auf den Install-Button startet die Installation sofort
2. **Detailansicht** — Klick auf das Paket öffnet die Detailseite; von dort aus kann ebenfalls installiert werden

Ein **Pfeil oben links** (← Zurück) führt von der Detailansicht zurück zur Liste.

### Installations-Feedback

Nach einer Installation erscheint ein einfaches **Popup**:
- Installation erfolgreich
- Installation fehlgeschlagen

Kein Konfigurations-Zwischenschritt, keine weiteren Dialoge. Fertig.

### Detailansicht eines Pakets

Die Detailansicht zeigt alle relevanten Informationen zu einem Paket — schön aufbereitet:

| Bereich | Inhalt |
|---|---|
| **Header** | Icon, Name, Version, Typ-Badge, Kurzbeschreibung |
| **Vollständigkeit** | Ist das Paket vollständig? Sind alle Pflichtfelder gesetzt? (Icon, Beschreibung, Tags, Signatur, i18n, …) |
| **Sprachen** | Welche Sprachdateien (.ftl) sind enthalten? Liste der unterstützten Sprachen |
| **Icon** | Vorschau des Paket-Icons |
| **Tags** | Alle Tags als klickbare Filter-Chips |
| **Metadaten** | Autor, Lizenz, Homepage, Quell-Repository, Release-Channel |
| **Abhängigkeiten** | Direkte Abhängigkeiten + optionale Abhängigkeiten |
| **Signatur** | Signatur-Status (signiert / unsigniert + Key-ID) |
| **Screenshots/Previews** | Vorschaubilder wenn im Manifest hinterlegt |

### Bundles in der Detailansicht

Für Pakete vom Typ `bundle` zeigt die Detailansicht zusätzlich:
- Liste aller **enthaltenen Pakete** (inkl. Icons und Kurzbeschreibung)
- Liste der **optionalen Pakete**
- Jedes Paket in der Liste ist **anklickbar** → öffnet die Detailansicht des jeweiligen Pakets

---

## Init-Download-Eintrag

Init (`type = "bootstrap"`) ist **kein installierbares Paket**. Es ist der Bootstrap-Einstiegspunkt — das Binary das man braucht bevor der Store existiert. Deshalb wird es im Store als Download-Seite angeboten, nicht als normales Paket.

In der Sidebar erscheint "Init" als eigener Eintrag unterhalb der Typen. Die Ansicht zeigt:
- Kurzbeschreibung was Init ist
- Download-Buttons je Plattform + Architektur
- Alle verfügbaren Versionen (falls ein Repository nicht alle anbietet, wird das angezeigt)

### Manifest `[bootstrap]`-Block

```toml
[package]
id = "fs-init"
name = "FreeSynergy Init"
version = "0.3.0"
type = "bootstrap"
description = "Bootstrap-Binary — installiert den Store auf einem neuen Gerät"
icon = "freesynergy-init"
tags = ["bootstrap", "init", "installer"]
author = "FreeSynergy"
license = "AGPL-3.0"
protected = true

[bootstrap]
# Welche Versionen dieses Repository anbietet
versions = ["0.3.0", "0.2.1", "0.2.0"]

# Download-URLs je Plattform (Template: {version} wird ersetzt)
[bootstrap.downloads]
linux-x86_64   = "https://github.com/FreeSynergy/fs-init/releases/download/v{version}/fs-init-x86_64-linux"
linux-aarch64  = "https://github.com/FreeSynergy/fs-init/releases/download/v{version}/fs-init-aarch64-linux"
macos-x86_64   = "https://github.com/FreeSynergy/fs-init/releases/download/v{version}/fs-init-x86_64-macos"
macos-aarch64  = "https://github.com/FreeSynergy/fs-init/releases/download/v{version}/fs-init-aarch64-macos"
windows-x86_64 = "https://github.com/FreeSynergy/fs-init/releases/download/v{version}/fs-init-x86_64-windows.exe"
```

Der Store liest `[bootstrap.versions]` und zeigt einen Download-Button je Plattform. Bietet ein Repository nur bestimmte Versionen an, werden nur diese gelistet.

---

## Geschützte Pakete

Manche Pakete können **nicht deinstalliert** werden — sie dürfen nur aktualisiert werden. Das Flag `protected = true` im Manifest verhindert das Löschen.

| Paket | Warum geschützt |
|---|---|
| `fs-store` | Store kann sich nicht selbst löschen |
| `fs-init` (Bootstrap-Eintrag) | Einstiegspunkt muss immer abrufbar sein |

Versucht der Nutzer ein geschütztes Paket zu deinstallieren, erscheint ein Hinweis: "Dieses Paket ist für den Betrieb von FreeSynergy erforderlich und kann nicht deinstalliert werden."

---

## Sprachdateien pro Paket

Jedes Paket bringt seine eigenen Sprachdateien mit — die `.ftl`-Dateien sind Teil des Pakets selbst:

```
packages/apps/kanidm/
└── help/
    ├── en/          ← Pflicht (Fallback)
    │   ├── description.ftl   ← lange Paket-Beschreibung
    │   ├── overview.ftl      ← Hilfe-Übersicht im Manager
    │   └── fields.ftl        ← Feldbeschreibungen für Konfiguration
    ├── de/
    │   ├── description.ftl
    │   ├── overview.ftl
    │   └── fields.ftl
    └── ar/
        └── ...
```

Wenn eine Sprache aktiviert wird, koordiniert das **Store-Objekt** die i18n-Sammlung: Es fragt das **Inventory** nach allen installierten Paketen und holt deren `.ftl`-Dateien — denn nur Store + Inventory wissen was installiert ist. Jedes Programm kennt keine anderen Programme; der Store ist der Koordinator.

**Sprach-Pakete** (`type = "language"`) sind ausschließlich für **Shared Snippets** — allgemeine Strings die von allen Programmen genutzt werden: Aktionen (`save`, `cancel`, `delete`), Status (`loading`, `error`, `success`), Bestätigungen, Zeitangaben.

```
packages/i18n/de/
└── shared.ftl   ← "save = Speichern", "cancel = Abbrechen", ...
```

---

## API-Interface

```
GET  /api/store/packages?q=&type=&tags=&installed=
GET  /api/store/packages/:id
GET  /api/store/packages/:id/versions
POST /api/store/install    { "id": "kanidm", "version": "1.5.0" }
POST /api/store/update     { "id": "kanidm" }
POST /api/store/rollback   { "id": "kanidm" }
DELETE /api/store/packages/:id?keep_data=true
GET  /api/store/know/service/:id
GET  /api/store/know/variable/:name
GET  /api/store/know/compatible?role=
POST /api/store/sync
```

---

## Binary-Verteilung

Kompilierte Rust-Binaries gehören **nicht** in Git-Repos. Git ist für Text — `target/`-Ordner können mehrere GB groß werden, GitHub hat ein 100 MB-Limit pro Datei.

**Drei Ebenen:**

**Ebene 1 — Git-Repos (nur Code + Text):**
Jedes Programm hat ein eigenes Repo mit Quellcode, Manifesten, Configs. Kein `target/`, keine `.exe`, keine `.so`.

**Ebene 2 — GitHub Releases (Binaries für Veröffentlichung):**
Nur für getaggte Releases. GitHub Actions kompiliert automatisch wenn ein Tag gepusht wird:
```
git tag v0.5.0 && git push --tags
→ GitHub Actions: kompiliert für linux-x86_64, linux-aarch64, macos-*, windows-x86_64
  → Lädt Binaries als Release-Assets hoch
```

**Ebene 3 — Lokale Entwicklung:**
```bash
cargo build --release
fsn store install --local ./target/release/fsn-node
# Store liest Metadaten aus lokalem Git, kopiert Binary, registriert im Inventory
```

### Manifest-Feld [distribution]

Jedes App-Manifest deklariert die Download-URLs je Plattform:

```toml
[package]
id = "node"
name = "FreeSynergy Node"
version = "0.5.0"
type = "app"
icon = "freesynergy-node"
tags = ["node", "server", "infrastructure"]

[interfaces]
cli  = true
api  = true
wgui = false
tui  = false

[distribution]
linux-x86_64   = "https://github.com/FreeSynergy/fs-node/releases/download/v{version}/fsn-node-x86_64-linux.tar.gz"
linux-aarch64  = "https://github.com/FreeSynergy/fs-node/releases/download/v{version}/fsn-node-aarch64-linux.tar.gz"
macos-x86_64   = "https://github.com/FreeSynergy/fs-node/releases/download/v{version}/fsn-node-x86_64-macos.tar.gz"
macos-aarch64  = "https://github.com/FreeSynergy/fs-node/releases/download/v{version}/fsn-node-aarch64-macos.tar.gz"
windows-x86_64 = "https://github.com/FreeSynergy/fs-node/releases/download/v{version}/fsn-node-x86_64-windows.zip"
source         = "https://github.com/FreeSynergy/fs-node"
```

### Installationsfluss

```
Veröffentlichtes Paket:
  fsn store install node
    → Store liest manifest.toml → erkennt Plattform
      → lädt Binary von GitHub Releases
        → registriert im Inventory

Lokal (Entwicklung):
  cargo build --release
  fsn store install --local ./target/release/fsn-node
    → Store liest Metadaten aus lokalem Git
      → kopiert Binary aus target/
        → registriert im Inventory

Source-Build (Fallback):
  fsn store install node --from-source
    → klont Repo → cargo build --release → wie oben
```

### catalog.toml — Drei Ebenen

Der Store ist in drei Ebenen aufgebaut. Jede Ebene hat eine `catalog.toml`:

```
Store/
├── catalog.toml                         ← Ebene 1: Namespace-Index + Store-Metadaten
├── icon.svg
├── init/
│   └── catalog.toml                     ← Bootstrap-Paket (type = "bootstrap")
└── packages/
    ├── apps/catalog.toml                ← Ebene 2: Namespace-Catalog
    │   ├── node/catalog.toml            ← Ebene 3: Paket-Catalog (fs-node)
    │   ├── kanidm/catalog.toml          ← Ebene 3: Paket-Catalog
    │   └── ...
    ├── containers/catalog.toml
    │   ├── forgejo/catalog.toml
    │   └── ...
    ├── widgets/catalog.toml
    ├── themes/catalog.toml
    ├── i18n/catalog.toml
    ├── icons/catalog.toml               ← Icon-Sets
    ├── repos/catalog.toml               ← Store-Repository-Quellen
    ├── bots/catalog.toml
    ├── tasks/catalog.toml
    ├── bridges/catalog.toml
    └── bundles/catalog.toml
```

**Ebene 1 — Root `catalog.toml`** beschreibt welche Namespaces es gibt. Der Store-Client liest das beim Start um die Sidebar aufzubauen:

```toml
[catalog]
id      = "freesynergy-store"
name    = "FreeSynergy Store"
icon    = "icon.svg"
summary = "The official FreeSynergy package registry"

[[namespaces]]
id      = "init"
catalog = "init/catalog.toml"

[[namespaces]]
id      = "apps"
catalog = "packages/apps/catalog.toml"

[[namespaces]]
id      = "containers"
catalog = "packages/containers/catalog.toml"
# … weitere Namespaces
```

**Ebene 2 — Namespace-Catalog** (`packages/apps/catalog.toml`) listet die Pakete im Namespace:

```toml
[catalog]
id      = "apps"
name    = "Applications"
icon    = "icon.svg"
summary = "Native Rust applications — the core of FreeSynergy"

[[packages]]
id      = "kanidm"
catalog = "kanidm/catalog.toml"
```

**Ebene 3 — Paket-Catalog** (`packages/apps/kanidm/catalog.toml`) ist die vollständige Paket-Beschreibung (was früher `manifest.toml` hieß):

```toml
[package]
id      = "kanidm"
name    = "Kanidm"
type    = "app"
version = "1.4.2"
summary = "Identity and access management — OAuth2, OIDC, LDAP, SCIM, WebAuthn"
icon    = "icon.svg"
tags    = ["iam", "oidc", "oauth2", "ldap", "scim", "webauthn"]
```

Binaries werden **nie** im Store-Repo gespeichert — nur Metadaten + Referenzen zu GitHub Releases.

---

## Geplante Features

Features die noch nicht implementiert sind, aber zum Store-Konzept gehören:

| Feature | Beschreibung |
|---|---|
| **Store-Mirrors** | Andere Betreiber hosten eigene Stores. Mehrere Store-Quellen konfigurierbar (wie dnf-Repos). |
| **Lokaler Cache** | Einmal heruntergeladen → offline verfügbar. Kein zweiter Download bei Neuinstallation. |
| **Download-Statistiken** | Zählt wie oft ein Paket installiert wurde. Sichtbar im Store-UI. |
| **Kompatibilitäts-Matrix** | Jedes Paket deklariert "Funktioniert mit Node >= 0.5.0". Warnung bei Inkompatibilität. |
| **Screenshots/Previews** | SVG-Previews im Manifest (besonders sinnvoll für Themes und Widgets). |
| **Changelog-Anzeige** | Automatisch aus Git-Tags generiert, direkt im Store sichtbar. |
| **Bewertungen/Reviews** | Community-Bewertungen (erst wenn Community existiert). |

---

## Verzeichnisstruktur

    Store/
    ├── README.md
    ├── catalog.toml               ← Ebene 1: Namespace-Index + Store-Metadaten
    ├── icon.svg
    ├── init/                      ← Bootstrap-Sonderpaket (type = "bootstrap")
    │   ├── catalog.toml
    │   ├── icon.svg
    │   └── help/en|de/description.ftl
    └── packages/
        ├── apps/                  ← Native Rust-Binaries
        │   ├── catalog.toml
        │   ├── node/              ← fs-node App-Paket
        │   ├── kanidm/            ← alle anderen Apps flach
        │   ├── stalwart/
        │   ├── tuwunel/
        │   ├── zentinel/
        │   ├── zentinel-control-plane/
        │   ├── mistral/
        │   └── browser/
        ├── containers/            ← Container-Apps (Podman/Docker-agnostisch)
        │   ├── catalog.toml
        │   ├── forgejo/
        │   ├── postgres/
        │   └── ...
        ├── widgets/               ← Desktop-Widgets
        ├── themes/                ← Visuelle Themes (Bundles aus Theme-Ressourcen)
        ├── i18n/                  ← Shared Snippets (save, cancel, error, …)
        ├── icons/                 ← Icon-Sets (type = "icon_set", überschreibbar)
        ├── repos/                 ← Store-Repository-Quellen (type = "repo")
        ├── bots/                  ← Bot-Definitionen
        ├── tasks/                 ← Automatisierungs-Templates
        ├── bridges/               ← Service-Bridges
        └── bundles/               ← Meta-Pakete

    Jedes Paket-Verzeichnis enthält:
        {name}/
        ├── catalog.toml           ← Vollständige Paket-Beschreibung (war: manifest.toml)
        ├── icon.svg
        ├── help/
        │   ├── en/
        │   │   ├── description.ftl  ← lange Beschreibung (für Doku)
        │   │   ├── overview.ftl
        │   │   └── fields.ftl
        │   └── de/
        │       └── ...
        └── templates/             ← Konfigurationsvorlagen (nur wenn nötig)

---

## Implementierung (Crates)

| Bereich | Crate | Details |
|---|---|---|
| Ressource-Typen + Meta | `fs-types` (`resources/meta.rs`) | `ResourceType` (16 Varianten), `ResourceMeta`, `ValidationStatus` |
| Ressource-Structs | `fs-types` (`resources/*.rs`) | `AppResource`, `ContainerResource`, `WidgetResource`, `BotResource`, `BridgeResource`, `BundleResource`, Theme-Ressourcen |
| Release-Channels | `fs-pkg` (`channel.rs`) | `ReleaseChannel`: Stable/Testing/Nightly |
| Versionierung + Rollback | `fs-pkg` (`versioning.rs`) | `VersionManager`, `VersionRecord`, `RollbackError` |
| Paket-Signierung | `fs-crypto` (`signing.rs`, feature: `signing`) | `FsSigningKey`, `FsVerifyingKey`, `PackageSignature` — ed25519-dalek v2, SHA-256 |
| Signatur-Verifikation | `fs-pkg` (`signing.rs`) | `SignatureVerifier`, `SignaturePolicy` (RequireSigned/TrustUnsigned) |
| SQLite-Tracking | `fs-db` (`installed_package.rs`) | `installed_packages` Tabelle — `InstalledPackageRepo` |
| Store-Client (generisch) | `fs-store` (`fs-libs`) | `StoreClient`, `Catalog<M>`, `Manifest`-Trait, `I18nBundle`, `DiskCache`, `StoreSearch<M>` |
| Install-Kinds (OOP) | `fs-pkg` (`install_kind.rs`) | `PackageInstallExt`-Trait auf `PackageKind` — jede Art kennt ihren Installationsfluss |
| Abhängigkeits-Auflösung | `fs-pkg` (`dependency_resolver.rs`) | `DepGraph`, `DependencyResolver` (Kahn's Algorithmus) |
| Install-Lifecycle | `fs-pkg` (`installer.rs`) | `PackageInstaller`, Hooks, `EventBus` |
| Store-UI | `fs-store-app` (`fs-apps`) | Dioxus-App: Browser, Wizard, InstalledList, Settings — Provider Pattern |
| Builder + Validierung | `fs-builder` (`fs-apps`) | Container Builder, YAML-Analyse via `ComposeAnalyzer`, `res.save_to()` |

## Repos

| Bereich | GitHub |
|---|---|
| Store-Daten (Kataloge + Manifeste) | https://github.com/FreeSynergy/fs-store |
| Store-Client + Lib | `fs-store` Crate in https://github.com/FreeSynergy/fs-libs |
| Store-UI | `fs-store-app` Crate in https://github.com/FreeSynergy/fs-apps |

---

Weiter: [Init](../init/README.md) | [Pakete](../../konzepte/pakete.md) | [Desktop](../desktop/README.md)
