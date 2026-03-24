# Capabilities — Pakete deklarieren was sie bieten

[← Zurück zum Index](../INDEX.md) | [Rollen-System](rollen.md) | [Message Bus](bus.md) | [Pakete](pakete.md)

---

## Grundidee

Jedes Paket deklariert zwei Dinge:

1. **Was es bietet** (`[provides]`) — welche Fähigkeiten es hat, welche Bus-Messages es emittiert
2. **Was es braucht** (`[requires]`) — welche Fähigkeiten andere Pakete bereitstellen müssen

Ein Bundle oder Programm sagt nicht "ich brauche kanidm", sondern "ich brauche etwas das `iam` kann".
Das ist **Dependency Injection auf Paket-Ebene** — Services sind austauschbar, Fähigkeiten sind stabil.

---

## Deklaration im Paket-Manifest

    [package]
    id   = "kanidm"
    name = "Kanidm"
    type = "app"

    [provides]
    capabilities = ["iam", "iam.oidc-provider", "iam.scim-server", "iam.webauthn"]

    [messages]
    emits   = ["user.created", "user.deleted", "auth.success", "auth.failed"]
    listens = ["auth.request", "user.query", "user.sync"]

Ein Container-Paket das andere Dienste braucht:

    [package]
    id      = "outline"
    name    = "Outline"
    type    = "container"
    summary = "Collaborative wiki and knowledge base with OIDC login"

    [provides]
    capabilities = ["wiki", "wiki.create-page", "wiki.search", "wiki.api"]

    [messages]
    emits   = ["page.created", "page.updated", "page.deleted"]
    listens = ["user.created", "user.deleted"]

    # Capability, nicht konkretes Paket
    [requires]
    capabilities = ["iam.oidc-provider", "database.postgres"]
    optional     = ["cache.redis", "smtp.sender"]

---

## Bundles nutzen Capabilities statt Paketnamen

    [bundle]
    name = "server"

    # Konkret
    [[packages]]
    id      = "fs-node"
    version = ">=0.5.0"

    # Abstrakt: irgendetwas das "iam" kann
    [[capabilities]]
    name = "iam"

    # Gepinnt: darf nicht gelöscht werden solange Bundle aktiv
    [[packages]]
    id      = "zentinel"
    version = "1.2.3"
    pin     = true

**Version-Pins:** `pin = true` blockiert Löschen dieser Version im Store.
Erst wenn kein Bundle mehr pinnt, kann sie entfernt werden.

**Rekursive Bundles:** Bundles können andere Bundles referenzieren:

    [bundle]
    name = "server"

    [[bundles]]
    name = "management-tools"

    [[bundles]]
    name = "monitoring"

---

## Programme nutzen Capabilities

    [program.uses]
    auth     = { capability = "iam" }
    database = { capability = "database" }
    cache    = { capability = "cache.redis" }

    # Oder konkret:
    proxy = { package = "zentinel" }

Der Bus routet automatisch an den Service der die Capability bereitstellt.

---

## Language-Paket als Super-Package

Der Store koordiniert die i18n-Sammlung:

    Language: "Ich brauche alle i18n-Dateien für 'de-DE'"
      → Store: durchsucht Inventory
        → Alle Pakete mit capabilities = ["i18n"]
          → Store holt deren .ftl-Dateien
        → Übergibt alles gesammelt

Warum der Store koordiniert:
- Language kennt keine anderen Pakete
- Pakete sind passiv (haben ihre Dateien, werden nicht gefragt)
- Store ist einzige Quelle der Wahrheit (Inventory)

Jedes Paket mit i18n deklariert:

    [provides]
    capabilities = ["i18n"]

    [i18n]
    locales = ["en", "de", "fr"]
    path    = "i18n/"

---

## Providers (pro Tool)

Providers sind keine globale Liste — jedes Tool bringt seine eigene mit:

    packages/apps/node/zentinel/
      providers/
        dns/
          cloudflare.toml
          hetzner.toml
          ionos.toml
        acme/
          letsencrypt.toml
          zerossl.toml

Jeder Provider ist eine TOML-Datei:

    [provider]
    id   = "cloudflare"
    name = "Cloudflare"
    kind = "dns"

    [[variables]]
    name  = "CF_DNS_API_TOKEN"
    label = "API Token"
    kind  = "secret"

Zentinel liest sein `providers/`-Verzeichnis und bietet alles an was dort liegt.
Neuer Provider = neue TOML-Datei, fertig.

---

## Capability-Hierarchie

Capabilities erweitern die [Rollen](rollen.md). Jede Rolle ist eine Capability. Zusätzlich:

    i18n           — Paket bringt Übersetzungsdateien mit
    i18n.de        — speziell Deutsch
    i18n.rtl       — Rechts-nach-links

    icon-set       — Paket ist ein Icon-Set
    theme          — Paket ist ein Theme
    widget         — Paket ist ein Widget

    bus.messages   — Paket kommuniziert über den Bus
    bus.provider   — Paket kann als Bus-Backend dienen

---

## Capability-Registry (Store-API)

    GET /api/store/capabilities
    GET /api/store/capabilities/iam
    GET /api/store/capabilities/compatible?needs=["iam.oidc-provider","database"]

---

Weiter: [Rollen-System](rollen.md) | [Message Bus](bus.md) | [Pakete](pakete.md) | [Store](../programme/store/README.md)
