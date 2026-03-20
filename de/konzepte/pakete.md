# Pakete

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Rollen](rollen.md)

---

## Store-Kategorien

Der Store trennt Pakete in drei Ober-Kategorien. Diese Trennung ist sichtbar im Store-UI (drei Tabs) und im Manifest (`category`-Feld):

| Kategorie | Läuft auf | Beispiele |
|---|---|---|
| **`server`** | Node (Linux, dauerhaft) | Kanidm, Forgejo, Matrix, Outline |
| **`app`** | Gerät des Menschen (jede Plattform) | FreeSynergy Desktop, Browser |
| **`desktop`** | Desktop-Oberfläche | Widgets, Themes, Sprachpakete |

**Warum drei Kategorien?**
- `server` braucht einen laufenden Node (Linux) — darf nicht auf Desktop-Geräten installiert werden
- `app` läuft auf jedem Betriebssystem — Cross-Platform
- `desktop` sind UI-Komponenten — Widgets haben nichts mit allgemeinen Apps zu tun

---

## Ressource-Typen

Innerhalb der Kategorien gibt es spezifische **Typen**. Rust-Enum: `ResourceType` in `fsn-types/src/resources/meta.rs`.

| Kategorie | Typ | Rust-Variant | Inhalt | Beispiele |
|---|---|---|---|---|
| `server` | `container_app` | `ContainerApp` | Container-Service (Podman + Config) | Kanidm, Forgejo, Outline |
| `server` | `bridge` | `Bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `app` | `app` | `App` | FreeSynergy-Binary (Cross-Platform) | Desktop, Init |
| `desktop` | `widget` | `Widget` | Desktop-Widget | Uhr, System-Info |
| `desktop` | `language` | `Language` | Sprach-Snippets (Mozilla Fluent) | Deutsch, Japanisch |
| `desktop` | `bot` | `Bot` | Bot-Definition | Broadcast, Gatekeeper |
| `desktop` | `task` | `Task` | Automatisierungs-Template | "Docs ins Wiki" |
| — | `bundle` | `Bundle` | Meta-Ressource, fasst beliebige Ressourcen zusammen | server-minimal, desktop-full |

**Bundles** können Pakete aus mehreren Kategorien zusammenfassen (z.B. `server-minimal` enthält Server-Pakete + das Node-Binary).

**Theme-Ressourcen** gehören zur Kategorie `desktop` und sind selbst Bundles aus bis zu 8 Theme-Teilpaketen:

### Theme-Ressourcen (einzeln ladbar)

Ein **Theme** ist kein einzelner Typ, sondern ein **Bundle** aus bis zu 8 Theme-Ressourcen:

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

**Libraries** (`fsn-*` Crates) sind KEINE eigenständigen Ressourcen. Sie sind Abhängigkeiten die mit den Anwendungen mitkommen. Sie leben in einem shared-Ordner, Git handled das.

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
- `category` (server, app, desktop) — Ober-Kategorie für Store-UI-Tabs
- `type` (container_app, app, widget, language, bot, bridge, task, bundle) — spezifischer Typ
- `description` (Kurzbeschreibung)
- `tags` (für Suche — muss aussagekräftig sein)
- `icon` (SVG oder Icon-Name; PFLICHT, wenn fehlt → generisches Icon)

**Jedes Paket ist ein Objekt.** Überall wo es angezeigt wird sieht man Icon, Name, Version, Tags.

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
auto = "ollama-backend"   # Container App Manager legt internes Netz automatisch an

[volumes]
ollama_data    = { target = "/root/.ollama",        s3_path = "media/ollama" }
openwebui_data = { target = "/app/backend/data",    s3_path = "media/open-webui" }
```

Dieses Manifest plus eine zugehörige `podman-compose.yml` ergibt ein vollständiges Container-App-Paket. Der [Container App Manager](../programme/container_app/README.md) validiert und installiert es, der Store verwaltet Versionen und Abhängigkeiten.

---

## Variable-System (für Container-Pakete)

Variablen haben Basis-Typen und Rollen. Siehe [Container App Manager](../programme/container_app/README.md) für die Analyse und [Rollen](rollen.md) für die Hierarchie.

Secrets werden mit `age` verschlüsselt. Rollen-typisierte Variablen werden automatisch befüllt wenn ein passender Service installiert ist.

---

Weiter: [Store](../programme/store/README.md) | [Container App Manager](../programme/container_app/README.md) | [Rollen](rollen.md)
