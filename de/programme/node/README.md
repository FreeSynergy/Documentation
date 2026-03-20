# Node — Der Projektverwalter

[← Zurück zum Index](../../INDEX.md) | [Container App Manager](../container_app/README.md) | [Architektur](../../architektur/uebersicht.md)

---

## Was Node macht

Node ist nicht nur ein Verwaltungsprogramm — Node ist der **Knoten im Netz**. Der Punkt über den alles läuft: Infrastruktur, Identitäten, Verbindungen, Zugriffsrechte.

Node verwaltet:

- **Hosts** — Server die zum Projekt gehören
- **Services** — Installation und Verwaltung über den [Container App Manager](../container_app/README.md)
- **S3-Storage** — Eingebauter S3-Server ([Storage-Layer](../../technik/storage.md))
- **Projekte** — Ein Projekt = eine Gruppe von Hosts + Services + Konfiguration
- **Einladungen** — Join-Tokens für neue Hosts und Desktop-Clients
- **VPN** — WireGuard-Verbindungen für mobile Geräte und Remote-Zugriff ([VPN & Netzwerk](../../konzepte/vpn.md))
- **Föderation** — Beitritt zu und Verwaltung von [Föderationen](../../konzepte/foederation.md)
- **Netzwerk-Topologie** — Wer darf mit wem reden, Routing, Firewall-Regeln

## Plattformen

**Node läuft ausschließlich auf Linux.** Das ist eine bewusste Entscheidung, keine Einschränkung.

| OS | Als Node | Als Desktop-Client |
|---|---|---|
| Linux | ✅ Voll unterstützt | ✅ |
| macOS | ❌ (kein systemd) | ✅ |
| Windows | ❌ (kein systemd) | ✅ |
| iOS / Android | ❌ | ✅ (später) |

**Warum Linux-only?** Container App Manager braucht Podman + Quadlet + systemd. Nur Linux hat das. Wenn wir irgendwann andere Plattformen unterstützen wollen, ist Node die **einzige Stelle die geändert werden muss** — alle anderen Programme bleiben unberührt.

Für Heimnutzer ohne Linux-Server: [Plattform-Szenarien](../../konzepte/node-plattformen.md)

## S3-Server (eingebaut)

Node IST der S3-Server. Kein externer Container (kein Garage, kein MinIO). Beim Node-Start wird der S3-Server automatisch gestartet.

Siehe [Storage-Layer](../../technik/storage.md) für Details.

## Interfaces

| Interface | Beispiel |
|---|---|
| CLI | `fsn host add`, `fsn invite create`, `fsn project list` |
| API | `GET /api/hosts`, `POST /api/invite`, `GET /api/services` |
| UI | Über [Desktop](../desktop/README.md) (ruft API auf) |

## Invite-System

Node generiert verschlüsselte Invite-Dateien:

```toml
[invite]
token = "a1b2c3.x9y8z7w6v5u4t3s2"
ca_hash = "sha256:4f8a..."
host = "node1.example.com:9443"
network_id = "uuid-..."
created_at = "2026-03-16T12:00:00Z"
expires_at = "2026-03-17T12:00:00Z"
permissions = "full"
```

Der Port kann für jede Einladung unterschiedlich sein — wird nur solange geöffnet wie die Einladung gültig ist.

## Eigenständigkeit

Node läuft OHNE Desktop, OHNE Store, OHNE Internet. Wenn Store verfügbar ist, nutzt es ihn. Wenn Desktop angeschlossen wird, bedient es dessen API-Anfragen.

## Repo

https://github.com/FreeSynergy/Node

## Bibliotheken

| Crate | Zweck |
|---|---|
| `s3s` | S3-Server-Framework |
| `opendal` | Storage-Backends (lokal, SFTP, S3, ...) |
| `fsn-types` | Shared Types |
| `fsn-config` | TOML-Konfiguration |
| `fsn-db` | SQLite (fsn-core.db) |
| `fsn-sync` | Automerge CRDT |
| `fsn-crypto` | age, mTLS, Tokens |
| `fsn-auth` | OAuth2, JWT |
| `fsn-federation` | OIDC, SCIM, ActivityPub |
| `fsn-bus` | Message Bus |
| `russh` | SSH für Remote-Host-Management |
| `clap` | CLI-Framework |
| `axum` | HTTP-API-Server |

---

Weiter: [Container App Manager](../container_app/README.md) | [Storage-Layer](../../technik/storage.md) | [Installation](../../technik/installation.md)
