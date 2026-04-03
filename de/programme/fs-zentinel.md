# Zentinel (Reverse-Proxy)

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

Zentinel ist ein Rust-nativer Reverse-Proxy und API-Gateway.
FreeSynergy betreibt einen Zentinel-Fork als Ingress-Layer vor allen Container-Services.

- **Zentinel** — Proxy-Instanz (TLS, HTTP/2, Routing, Rate-Limiting)
- **Zentinel Control Plane** — verwaltet eine Flotte von Zentinel-Instanzen (Routen, TLS-Certs, Health)

---

## Architektur

```
                    Browser / Client
                          │
                          ▼
                    Zentinel (Port 8080/8443)
                    TLS-Termination, Routing
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
       Kanidm          Stalwart         Forgejo
    /auth → 8443    /mail → 25/143   /git → 3000

                          │
             Zentinel Control Plane (Port 9090)
             REST-API für Routen-Konfiguration
                          │
                  ZentinelManager (Facade)
                  ← ZentinelBusHandler →
                  fs-bus (registry::service::*)
```

---

## Design Pattern: Facade

`ZentinelManager` ist eine Facade über die Zentinel Control Plane API.
Aufrufer arbeiten mit `RouteConfig` und Service-IDs — nie mit rohen HTTP-Calls.

---

## RouteConfig

| Feld          | Typ             | Beschreibung                               |
|---------------|-----------------|--------------------------------------------|
| `id`          | `String`        | Eindeutige Routen-ID, z. B. `kanidm-main`  |
| `service_id`  | `String`        | Registry-Service-ID, z. B. `kanidm`        |
| `path`        | `String`        | Pfad-Prefix, z. B. `/auth`                 |
| `upstream`    | `String`        | Upstream-URL, z. B. `https://kanidm:8443`  |
| `strip_path`  | `bool`          | Pfad-Prefix vor dem Forward entfernen      |
| `protocol`    | `RouteProtocol` | `Http` | `Https` | `Tcp`                   |
| `description` | `String`        | Anzeige im Zentinel-Dashboard              |

---

## Auto-Routing (ZentinelBusHandler)

Wenn ein Dienst sich in `fs-registry` registriert, feuert der Bus das Event
`registry::service::registered`. Der `ZentinelBusHandler` abonniert dieses Event
und ruft `ZentinelManager::auto_route_for_service` auf.

**Capability → Pfad-Mapping:**

| Capability              | Standard-Pfad  |
|-------------------------|----------------|
| `iam`, `iam.*`          | `/auth`        |
| `mail`                  | `/mail`        |
| `git`                   | `/git`         |
| `wiki`                  | `/wiki`        |
| `chat`                  | `/chat`        |
| `storage`, `s3`         | `/storage`     |
| `proxy.control-plane`   | `/_zentinel`   |
| (alle anderen)          | `/`            |

Bei `registry::service::stopped` werden alle Routen des Dienstes entfernt.

---

## i18n

- Englisch: `fs-i18n/locales/en/zentinel.ftl`
- Deutsch:  `fs-i18n/locales/de/zentinel.ftl`

---

## Store-Einträge

- `packages/containers/zentinel/catalog.toml` — Zentinel Proxy-Container
- `packages/containers/zentinel-plane/catalog.toml` — Control Plane Container

Beide als Fork-Pakete (`FreeSynergy/zentinel`, `FreeSynergy/zentinel-control-plane`).

---

## Repos

- Fork: `/home/kal/Server/fs-zentinel/` → `git@github.com:FreeSynergy/zentinel.git`
- Fork: `/home/kal/Server/fs-zentinel-plane/` → `git@github.com:FreeSynergy/zentinel-control-plane.git`
- Manager: `/home/kal/Server/fs-managers/zentinel/` (`fs-manager-zentinel`)

---

Weiter: [← Index](../INDEX.md)
