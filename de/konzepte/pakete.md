# Pakete

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Rollen](rollen.md)

---

## Paket-Typen

| Typ | Inhalt | Beispiele |
|---|---|---|
| `app` | FreeSynergy-Kernanwendung | Node, Desktop, Conductor |
| `container` | Service-Modul (Quadlet + Config) | Kanidm, Forgejo, Outline |
| `bundle` | Meta-Paket, fasst beliebige Pakete zusammen | server-minimal, desktop-full |
| `language` | Sprach-Snippets (.ftl) | Deutsch, Französisch |
| `theme` | Visuelles Theme | Midnight Blue, Nordic |
| `widget` | Desktop-Widget | Uhr, System-Info |
| `bot` | Bot-Definition | Broadcast, Gatekeeper |
| `bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `task` | Automatisierungs-Template | "Docs ins Wiki" |

**Libraries** (`fsn-*` Crates) sind KEINE eigenständigen Pakete. Sie sind Abhängigkeiten die mit den Anwendungen mitkommen. Sie leben in einem shared-Ordner, Git handled das.

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
- `type` (app, container, bundle, language, theme, widget, bot, bridge, task)
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
auto = "ollama-backend"   # Conductor legt internes Netz automatisch an

[volumes]
ollama_data    = { target = "/root/.ollama",        s3_path = "media/ollama" }
openwebui_data = { target = "/app/backend/data",    s3_path = "media/open-webui" }
```

Dieses Manifest plus eine zugehörige `podman-compose.yml` ergibt ein vollständiges Container-App-Paket. Der [Conductor](../programme/conductor/README.md) validiert und installiert es, der Store verwaltet Versionen und Abhängigkeiten.

---

## Variable-System (für Container-Pakete)

Variablen haben Basis-Typen und Rollen. Siehe [Conductor](../programme/conductor/README.md) für die Analyse und [Rollen](rollen.md) für die Hierarchie.

Secrets werden mit `age` verschlüsselt. Rollen-typisierte Variablen werden automatisch befüllt wenn ein passender Service installiert ist.

---

Weiter: [Store](../programme/store/README.md) | [Conductor](../programme/conductor/README.md) | [Rollen](rollen.md)
