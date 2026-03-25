# Pakete

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Rollen](rollen.md)

---

## Paket-Typen

Es gibt keine Kategorien (Server/App/Desktop). Jedes Paket hat genau einen **Typ**. Rust-Enum: `ResourceType` in `fs-types/src/resources/meta.rs`.

| Typ | Rust-Variant | Inhalt | Beispiele |
|---|---|---|---|
| `app` | `App` | Native Rust-Binary (Cross-Platform) | Node, Desktop, Browser, Kanidm, Zentinel, Stalwart, Mistral |
| `container` | `Container` | Container-App — läuft mit Podman **oder** Docker (runtime-agnostisch) | Forgejo, Postgres, Outline, CryptPad |
| `bridge` | `Bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `widget` | `Widget` | Desktop-Widget | Uhr, System-Info |
| `language` | `Language` | Shared Snippets (Mozilla Fluent) | Deutsch, Japanisch |
| `bot` | `Bot` | Bot-Definition | Broadcast, Gatekeeper |
| `task` | `Task` | Automatisierungs-Template | "Docs ins Wiki" |
| `bundle` | `Bundle` | Meta-Ressource, fasst beliebige Pakete zusammen — **Root-Level** (kein Unterverzeichnis von `packages/`) | Zentinel (Proxy + Control Plane) |
| `theme` | `Theme` | Bundle-Unterart mit fester Design-Struktur — **Root-Level** (`themes/`) | Midnight Blue, Nordic Dark |
| `bootstrap` | `Bootstrap` | Sondertyp: Init-Binary zum Download (kein Install) | fs-init |
| `repo` | `Repo` | Store-Repository-Quelle — Installation registriert neue Paketquelle | freesynergy-community |
| `icon_set` | `IconSet` | SVG-Icon-Sammlung — kann Default-Set überschreiben, shareable | freesynergy-default |

**`container` vs. `app`:**
- `app` = natives Rust-Binary (kompiliert, kein Container-Runtime nötig): Kanidm, Stalwart, Zentinel, Mistral
- `container` = läuft als Container-Image: Forgejo, Postgres, Outline — kein `requires:podman`-Tag nötig, der Node erkennt die verfügbare Runtime automatisch

**`language`-Pakete** sind ausschließlich für **Shared Snippets** (allgemeine Strings wie `save`, `cancel`, `error`). Jedes Programm bringt seine eigenen `.ftl`-Dateien mit. Wenn eine Sprache aktiviert wird, koordiniert das Store-Objekt die i18n-Sammlung: Es fragt das Inventory nach allen installierten Paketen und holt deren programm-spezifische Übersetzungen — denn nur Store + Inventory wissen was installiert ist.

**Bundles** und **Themes** sind keine Pakete im normalen Sinn — sie **enthalten keine Binaries**, sondern referenzieren andere Pakete per ID. Deshalb leben sie im Store **nicht unter `packages/`**, sondern im Root-Verzeichnis:

```
bundles/    ← generische Bundles (type = "bundle")
themes/     ← Design-Bundles (type = "theme")
packages/   ← alle anderen Pakete
```

Das Store-Objekt löst Bundle-Komponenten bei Bedarf auf und zeigt dem Benutzer Links zu den Einzelpaketen. Die Help-Dateien eines Bundles können über FTL-Variablen (`{ $link-paketname }`) auf Einzelpakete verweisen.

`is_bundle()` → gibt `true` für `Bundle` **und** `Theme` (beide sind Root-Level-Bundles).

**Theme-Ressourcen** (einzeln ladbar) — ein Theme-Bundle referenziert diese:

### Theme-Ressourcen (einzeln ladbar)

Ein **Theme** (`type = "theme"`) ist ein Bundle-Unterart mit **fester Struktur** — maximal 8 Theme-Ressourcen:

| Typ | Rust-Variant | Inhalt |
|---|---|---|
| `color_scheme` | `ColorScheme` | CSS-Farbpalette (alle Pflicht-Variablen) |
| `style` | `Style` | Spacing, Radius, Shadows (standardisiertes Schema) |
| `font_set` | `FontSet` | Schriftart-Dateien + @font-face |
| `cursor_set` | `CursorSet` | Mauszeiger-Dateien |
| `icon_set` | `IconSet` | SVG Icon-Sammlung |
| `button_style` | `ButtonStyle` | Button-Aussehen |
| `window_chrome` | `WindowChrome` | Titelleisten-Stil |
| `animation_set` | `AnimationSet` | CSS-Transitions + Keyframes |

Ein vollständiges Theme = Bundle das je eine dieser Ressourcen referenziert.

**Libraries** (`fs-*` Crates) sind KEINE eigenständigen Ressourcen. Sie sind Abhängigkeiten die mit den Anwendungen mitkommen. Sie leben im Monorepo `fs-libs`, Git handled das.

## Manageable-Trait — Pakete als selbstbeschreibende Objekte {#manageable-trait}

Jedes installierte Paket implementiert den `Manageable`-Trait (in `fs-pkg/src/manageable.rs`). Das Paket ist selbst verantwortlich für:

| Methode | Was sie liefert |
|---|---|
| `meta()` | Name, Version, Beschreibung, Icon, Kategorie, Autor |
| `package_type()` | App / Container / Bot / Bridge / … |
| `is_installed()` | Ob das Paket installiert ist |
| `run_status()` | Running / Stopped / Starting / Stopping / Error / NotInstalled |
| `config_fields()` | Alle Konfigurationsfelder mit Typ, Wert, Hilfe-Text |
| `check_health()` | Health-Checks: Konfigurationsdatei vorhanden? Datenverzeichnis existiert? |
| `apply_config()` | Konfiguration übernehmen (Key-Value-Map) |
| `instances()` | Sub-Instanzen (für Bot/Container — z.B. mehrere SMTP-Instanzen) |
| `build_fields()` | Build-Zeit-Felder (für den Builder-Tab) |
| `can_start()` / `can_stop()` / `can_persist()` | Welche Aktionen gerade möglich sind |

### Package-Objekt

Das `Package`-Struct (in `fs-pkg/src/package.rs`) vereint:
- `ApiManifest` — Metadaten aus dem Store
- `InstalledRecord` — Installations-Zustand (Version, Pfad, Status)
- `config_overrides` — aktive Konfiguration im RAM

Es ist die **einzige Quelle der Wahrheit** für ein Paket. Wer wissen will ob kanidm läuft, fragt `package.run_status()` — nie die Datenbank direkt.

### ConfigField — Felder beschreiben sich selbst

Jedes Konfigurationsfeld gibt an:
- `key` / `label` — Schlüssel und Anzeigename
- `help` — **Pflicht-Hilfetext** (fehlt er, wird das im Manager mit Warnung markiert)
- `kind` — Text / Password / Number / Bool / Select / Port / Path / Textarea
- `value` — aktueller Wert
- `required` — Pflichtfeld?
- `needs_restart` — Neustart nötig nach Änderung?

Der Manager (`ManagerView`) und der Settings Manager (`PackageSettingsView`) lesen diese Felder und rendern daraus eine fertige Oberfläche — ohne paketspezifischen UI-Code.

### HealthCheck

```rust
// Beispiel: Package::check_health() prüft automatisch:
// 1. Konfigurationsdatei vorhanden
// 2. Datenverzeichnis existiert
// 3. Paket-spezifische Prüfungen (override möglich)
```

Alle Health-Checks laufen im Info-Tab des Managers auf — grün wenn ok, rot mit Meldung wenn nicht.

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

## Interface-Spezifikation (für App-Pakete)

Jedes `app`-Paket deklariert welche Interfaces es hat:

```toml
[interfaces]
cli  = true    # Immer: clap-basiertes CLI
api  = true    # REST-API via axum (wenn sinnvoll)
wgui = false   # Web-based GUI via Dioxus WebView (optional)
tui  = false   # Terminal-UI via ratatui (später, niedrige Priorität)
```

**WGUI** steht für *Web-based GUI* — die UI wird immer in einer Web-Engine gerendert (HTML/CSS in webkit2gtk / WebView2 / WKWebView). Auch auf dem Desktop ist es technisch ein WebView, kein natives GTK/Qt.

**TUI** kommt nicht automatisch aus WGUI — HTML/CSS lässt sich nicht in Terminal-Raster übersetzen. TUI wird nur für ausgewählte CLI-Funktionen umgesetzt (z.B. `fsn container-app status`, `fsn store search`) und nicht als vollständiger Desktop-Ersatz.

## Pflicht-Metadaten

Jedes Paket MUSS haben:
- `id` (einzigartiger Name, KEIN Typ-Prefix)
- `name` (Anzeigename)
- `version` (SemVer, aus Git-Tag)
- `type` (app, container, bridge, widget, language, bot, task, bundle, bootstrap, repo, icon_set)
- `summary` (max 255 Zeichen — Store-Karte, Suchergebnisse; fehlt oder > 255 → `Incomplete`)
- `description` (mittellang, inline im Catalog — Store-Detailansicht; fehlt → `Broken`)
- `description_file` (Pfad zur `.ftl`-Datei — `help/en/description.ftl`; fehlt → `Broken`)
- `tags` (für Suche — muss aussagekräftig sein; inkl. `platform:*`-Tags)
- `icon` (SVG-Datei; PFLICHT, fehlt → `Broken`)

### Drei Beschreibungsebenen (alle Pflicht)

| Feld | Max | Wo | Übersetzbar |
|---|---|---|---|
| `summary` | 255 Zeichen | Store-Karte, Suchergebnisse, Sidebar | Über i18n-System |
| `description` | frei | Store-Detailansicht (inline im Catalog) | Über i18n-System |
| `description_file` | — | Pfad zu `help/en/description.ftl` — Doku, Help-Seiten | Ja — `.ftl` je Sprache |

Wenn ein Paket kein `description` oder kein `description_file` hat, wird es mit `ValidationStatus::Broken` markiert und kann nicht installiert werden. Fehlende Übersetzungen fallen auf `en` zurück.

**Jedes Paket ist ein Objekt.** Überall wo es angezeigt wird sieht man Icon, Name, Version, Tags.

## Installations-Pfad

Der Installationspfad wird **nicht im Manifest angegeben**, sondern vom Store deterministisch abgeleitet:

```
{base_dir}/{type}/{id}/{id}-{version}/
```

Beispiele:

| Paket | Typ | Pfad |
|---|---|---|
| kanidm 1.5.0 | app | `{base_dir}/app/kanidm/kanidm-1.5.0/` |
| desktop 0.8.0 | app | `{base_dir}/app/desktop/desktop-0.8.0/` |
| midnight-blue 1.0.0 | bundle | `{base_dir}/bundle/midnight-blue/midnight-blue-1.0.0/` |

**`base_dir` ist ein einziger konfigurierbarer Pfad** (Settings → Install Path). Standard: `/opt/freesynergy/` auf Linux, `~/.local/share/freesynergy/` auf anderen Plattformen.

**Pfad ändern:** Wenn der Nutzer den Install Path ändert, verschiebt der Store alle installierten Pakete automatisch (`mv {old_base}/* {new_base}/`) und aktualisiert die Pfade im Inventory.

Das Inventory (SQLite) kennt den vollen Pfad jedes installierten Pakets. Programme die wissen müssen wo ein anderes Paket installiert ist, fragen das Inventory — nie den Pfad selbst ableiten.

**Kein Init-Script nötig.** Der Store liest das Manifest, leitet den Pfad ab, installiert, schreibt ins Inventory. Fertig. Kein vorkompilierter Pfad, kein Hardcode.

## Sprachpakete (`language`)

Sprachpakete haben einen erweiterten `[language]`-Block mit Pflichtfeldern für Filter:

```toml
[language]
locale       = "de-DE"
display_name = "Deutsch"
direction    = "ltr"          # oder "rtl" (ar, fa, ur, ps)
completeness = 100            # Prozent der übersetzten Keys
programs     = ["node", "desktop", "init"]
family       = "Indo-European" # Sprachfamilie (Pflicht)
continent    = "Europe"        # Kontinent (Pflicht)
```

`family` und `continent` sind **eigene Felder, keine Tags** — damit sie beim Erstellen eines neuen Pakets nicht vergessen werden können. Mögliche Werte:

- **family:** `Indo-European`, `Afro-Asiatic`, `Sino-Tibetan`, `Dravidian`, `Uralic`, `Turkic`, `Austronesian`, `Niger-Congo`, `Japonic`, `Koreanic`, `Kra-Dai`, `Austroasiatic`
- **continent:** `Europe`, `Asia`, `Africa`, `Americas`, `Oceania`

Jedes Sprachpaket enthält 12 TOML-Snippet-Dateien (actions, nouns, status, errors, validation, phrases, time, help, labels, confirmations, notifications, meta) und ein eigenes Sprechblasen-Icon (24×24 SVG) mit dem Sprachcode.

Details: [i18n-Technik](../technik/i18n.md)

## Tags

Tags sind das primäre Suchinstrument. **Schlechte Tags = Paket unsichtbar.**

Tags sind **typsicher** — Rust-Typ `FsTag` aus bekannten Bibliotheken. Kein freier String.
Jeder Tag-Schlüssel ist ein i18n-Schlüssel: `tag.<key>` → übersetzt in jeder Sprache.

```toml
tags = ["package.database", "platform.linux", "api.oidc"]
```

### Tag-Bibliotheken

Drei eingebaute Bibliotheken (Rust: `PackageTags`, `PlatformTags`, `ApiTags`):

| Bibliothek | Prefix | Beispiele |
|---|---|---|
| `PackageTags` | `package.*` | `package.database`, `package.ai`, `package.security`, `package.chat` |
| `PlatformTags` | `platform.*` / `requires.*` | `platform.linux`, `platform.macos`, `requires.systemd`, `requires.podman` |
| `ApiTags` | `api.*` | `api.rest`, `api.oidc`, `api.matrix`, `api.activitypub` |

### Platform- und Feature-Tags (standardisiert)

Der Store kombiniert `PlatformTags` mit [SysInfo](sysinfo.md): Pakete werden aus- oder abgeblendet wenn die Voraussetzung auf dem aktuellen System nicht erfüllt ist.

| Tag-Schlüssel | Bedeutung |
|---|---|
| `platform.linux` | Nur Linux unterstützt |
| `platform.macos` | Nur macOS unterstützt |
| `platform.windows` | Nur Windows unterstützt |
| `requires.systemd` | systemd muss laufen |
| `requires.pam` | PAM muss verfügbar sein |
| `requires.podman` | Podman muss installiert sein |
| `requires.git` | Git muss installiert sein |

### Neue Tags hinzufügen

Neue Tags in die Bibliothek eintragen (nicht einfach einen freien String verwenden):

1. Schlüssel zu `ALL_KEYS` in `fs-types/src/tags/<bibliothek>.rs`
2. Factory-Funktion hinzufügen (`pub fn database() -> FsTag`)
3. i18n-Eintrag `tag.<key>` in alle Sprachpakete schreiben
4. Der Test `tag_key_convention` in `fs-types` prüft automatisch die Naming Convention

Details: [Typen-System](../technik/typen.md#tag-system) | [Store](../programme/store/README.md#tag-system)

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
fs-node = ">= 0.5.0"

[optional-dependencies]
kanidm = ">= 1.4.0"
```

Der Store löst Abhängigkeiten automatisch auf.

## Container-Manifest (Erweitertes Beispiel)

Container-Pakete (`type = "container"`) können mehrere Services definieren, Rollen deklarieren, Variablen typisieren und Healthchecks konfigurieren. Ein vollständiges Beispiel:

```toml
[package]
id = "ollama"
name = "Ollama + Open WebUI"
version = "0.1.0"
type = "container"
description = "Local LLM with Web Interface"
icon = "ollama"
tags = ["llm", "ai", "chat", "ollama", "openwebui", "gpu"]
author = "FreeSynergy"
license = "MIT"

[roles]
provides = ["llm", "llm.chat", "llm.api"]
requires_optional = ["iam.oidc-provider", "database.postgres"]

[services.main]
name = "open-webui"
image = "ghcr.io/open-webui/open-webui:main"
port = 3000

[services.sub.ollama]
name = "ollama"
image = "docker.io/ollama/ollama:latest"
port = 11434
internal = true           # Nicht von außen erreichbar

[variables]
WEBUI_SECRET_KEY         = { type = "secret",            required = true }
OLLAMA_BASE_URL          = { type = "url",               auto = "http://ollama:11434", internal = true }
OLLAMA_HOST              = { type = "hostname",          default = "0.0.0.0" }
ENABLE_OPENAI_API        = { type = "string",            default = "true" }
ENABLE_SIGNUP            = { type = "string",            default = "true" }
DO_NOT_TRACK             = { type = "string",            default = "true" }
OPENID_PROVIDER_URL      = { type = "url",               role = "iam.oidc-discovery",  required = false }
OAUTH_CLIENT_ID          = { type = "string",            role = "iam.oidc",            required = false }
OAUTH_CLIENT_SECRET      = { type = "secret",            role = "iam.oidc",            required = false }
DATABASE_URL             = { type = "connection-string", role = "database",            required = false }

[healthcheck.ollama]
test     = "curl -f http://localhost:11434/api/tags"
interval = "30s"

[healthcheck.open-webui]
test     = "curl -f http://localhost:8080/health"
interval = "30s"

[network]
auto = "ollama-backend"   # Container Manager legt internes Netz automatisch an

[volumes]
ollama_data    = { target = "/root/.ollama",        s3_path = "media/ollama" }
openwebui_data = { target = "/app/backend/data",    s3_path = "media/open-webui" }
```

Dieses Manifest plus eine zugehörige `podman-compose.yml` ergibt ein vollständiges Container-App-Paket. Der [Container Manager](../programme/container/README.md) validiert und installiert es, der Store verwaltet Versionen und Abhängigkeiten.

---

## Variable-System (für Container-Pakete)

Variablen haben Basis-Typen und Rollen. Siehe [Container Manager](../programme/container/README.md) für die Analyse und [Rollen](rollen.md) für die Hierarchie.

Secrets werden mit `age` verschlüsselt. Rollen-typisierte Variablen werden automatisch befüllt wenn ein passender Service installiert ist.

---

Weiter: [Store](../programme/store/README.md) | [Container Manager](../programme/container/README.md) | [Rollen](rollen.md)
