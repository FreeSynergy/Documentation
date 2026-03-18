# Bibliotheken

[← Zurück zum Index](../INDEX.md)

---

## fsn-* Crates (FreeSynergy/Lib)

Alle wiederverwendbaren Libraries. Können auch von Drittprojekten (Wiki.rs, Decidim.rs) genutzt werden. Libraries sind KEINE eigenständigen Store-Pakete — sie sind Abhängigkeiten die mit den Programmen mitkommen.

| Crate | Zweck | Wichtige Dependencies |
|---|---|---|
| `fsn-types` | Shared Types, Enums, IDs | `serde`, `uuid` |
| `fsn-error` | Fehlerbehandlung, Error-Typen | `thiserror`, `miette` |
| `fsn-config` | TOML laden/speichern | `serde`, `toml` |
| `fsn-i18n` | Sprach-Snippets, Fluent | `fluent`, `fluent-bundle`, `unic-langid` |
| `fsn-db` | SQLite-Abstraktion | `sea-orm`, `rusqlite`, `sqlx` |
| `fsn-sync` | CRDT-Synchronisation | `automerge` |
| `fsn-store` | Store-Client | `reqwest`, `toml`, `gix` |
| `fsn-pkg` | Paket-Management, Abhängigkeiten | `semver` |
| `fsn-plugin-sdk` | Plugin-API für WASM | `wit-bindgen` |
| `fsn-plugin-runtime` | WASM-Host | `wasmtime` |
| `fsn-federation` | OIDC, SCIM, ActivityPub | `activitypub_federation`, `openidconnect` |
| `fsn-auth` | OAuth2, JWT, Tokens | `oauth2`, `jsonwebtoken` |
| `fsn-bus` | Message Bus (Pub/Sub + Direct) | `tokio::broadcast`, `serde_json` |
| `fsn-channel` | Messenger-Adapter | Feature-flagged (siehe unten) |
| `fsn-bot` | Bot-Framework | `fsn-bus`, `fsn-channel` |
| `fsn-llm` | LLM-Integration | `reqwest` (Ollama API) |
| `fsn-bridge-sdk` | Service-zu-Service-Bridges | `fsn-bus` |
| `fsn-container` | Quadlet/systemctl-Interface | `tokio::process` |
| `fsn-template` | Tera-Templates | `tera` |
| `fsn-health` | Health-Checks | `reqwest`, `tokio` |
| `fsn-crypto` | age-Encryption, mTLS, Signierung | `age`, `rcgen`, `ed25519-dalek` |
| `fsn-theme` | Theme-Laden, Prefix-System | `toml`, `fsn-config` |
| `fsn-help` | Hilfe-System | `fsn-i18n` |
| `fsn-ui` | Dioxus-Komponenten | `dioxus`, `fsn-theme` |

## Externe Dependencies

### Kern-Infrastruktur

| Crate | Wofür | Benutzt von |
|---|---|---|
| `s3s` | S3-Server-Framework (Samsung) | Node (eingebauter S3-Server) |
| `s3s-fs` | S3 Dateisystem-Backend | Node |
| `opendal` | 40+ Storage-Backends (Apache) | Node (S3-Backend: lokal, SFTP, S3, ...) |
| `rust-s3` | S3-Client | Programme die mit S3 reden |
| `gix` (gitoxide) | Git in reinem Rust | Init, Store |

### Frameworks

| Crate | Wofür | Benutzt von |
|---|---|---|
| `dioxus` 0.7.x | UI-Framework | Desktop |
| `axum` | HTTP-Server/API | Node, Container App Manager, Store |
| `clap` | CLI | Alle Programme |
| `tokio` | Async Runtime | Alle |

### Serialisierung & Daten

| Crate | Wofür | Benutzt von |
|---|---|---|
| `serde` / `serde_json` / `toml` | Serialisierung | Alle |
| `serde_yaml` | YAML parsen | Container App Manager |
| `sea-orm` | ORM für SQLite/Postgres | Alle mit DB |
| `automerge` | CRDT | Sync, Offline-First |
| `semver` | Versionen vergleichen | Store, Pkg |

### Netzwerk & Sicherheit

| Crate | Wofür | Benutzt von |
|---|---|---|
| `reqwest` | HTTP-Client | Store, Desktop, LLM |
| `russh` | SSH-Client | Node (Host-Management) |
| `age` | Encryption | Crypto, Secrets |
| `ed25519-dalek` | Signierung (Default) | Crypto, Paket-Signierung |
| `rcgen` | Zertifikats-Generierung | mTLS |

### Templates & i18n

| Crate | Wofür | Benutzt von |
|---|---|---|
| `tera` | Template-Engine | Container App Manager, Templates |
| `fluent` | i18n Localization | i18n |

### Messenger (Feature-Flags bei fsn-channel)

| Crate | Wofür | Feature-Flag |
|---|---|---|
| `grammers` | Telegram UserBot (MTProto, volle Kontrolle) | `telegram-userbot` |
| `teloxide` | Telegram Bot-API (eingeschränkte Rechte) | `telegram` |
| `matrix-sdk` | Matrix Client | `matrix` |
| `serenity` + `poise` | Discord Bot | `discord` |
| `slack-morphism` | Slack Client | `slack` |
| `lettre` | Email senden | `email` |
| `xmpp-rs` | XMPP | `xmpp` |
| `irc` | IRC | `irc` |

### WASM

| Crate | Wofür | Benutzt von |
|---|---|---|
| `wasmtime` | WASM-Runtime | Plugin-Runtime |
| `wit-bindgen` | WASM Interface Types | Plugin-SDK |

---

Weiter: [Datenspeicherung](datenspeicherung.md) | [Storage-Layer](storage.md) | [i18n](i18n.md)
