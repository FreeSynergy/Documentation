# fs-managers

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-managers` · `/home/kal/Server/fs-managers/`
**Typ:** Workspace (7 Manager-Crates)
**Capabilities:** `managers.language`, `managers.theme`, `managers.icons`, `managers.cursor`, `managers.ai`, `managers.bots`, `managers.container`

---

## Was ist das?

`fs-managers` enthält die Backend-Crates für alle FreeSynergy-Manager.
Jeder Manager ist eine eigenständige Library mit CLI-Binaries, die von `fs-apps` (UI-Crates) genutzt werden.

---

## Manager im Workspace

| Crate                    | Capability           | Beschreibung                                              |
|--------------------------|----------------------|-----------------------------------------------------------|
| `fs-manager-language`    | `managers.language`  | Sprachpakete installieren, aktivieren, herunterladen      |
| `fs-manager-theme`       | `managers.theme`     | Themes laden, aktivieren, als Artifact nachladen          |
| `fs-manager-icons`       | `managers.icons`     | Icon-Sets verwalten, nach Theme filtern                   |
| `fs-manager-cursor`      | `managers.cursor`    | Mauszeiger-Sets verwalten                                 |
| `fs-manager-ai`          | `managers.ai`        | LLM-Modelle verwalten (Download, Aktivierung)             |
| `fs-manager-bots`        | `managers.bots`      | Bot-Instanzen konfigurieren                               |
| `fs-manager-container`   | `managers.container` | Container-Apps konfigurieren                              |

---

## Workspace-Struktur

```
fs-managers/
├── language/    — Sprachpack-Manager (fs-manager-language)
├── theme/       — Theme-Manager
├── icons/       — Icon-Set-Manager
├── cursor/      — Cursor-Manager
├── ai/          — AI-Modell-Manager
├── bots/        — Bot-Konfigurations-Manager
└── container/   — Container-App-Manager
```

---

## fs-manager-language — Domain-Modell

```rust
pub struct Language {
    pub id: String,           // ISO 639-1 Code ("de", "en")
    pub display_name: String, // Nativer Name ("Deutsch")
    pub locale: String,       // BCP-47 Locale ("de-DE")
}

pub enum DateFormat { DmY, MdY, Ymd }   // DD.MM.YYYY / MM/DD/YYYY / YYYY-MM-DD
pub enum TimeFormat { H24, H12 }         // 24h / 12h
pub enum NumberFormat { EuropeDot, UsComma, SpaceComma }
```

---

## Bekannter Bug

`fs-manager-language`: Nutzt alte gix-API (pre-gix 0.65).
`prepare_push` und `SignatureRef` müssen auf neue gix-API migriert werden.
Betrifft: `language/src/git.rs`

---

## Dependencies

| Dependency    | Zweck                                         |
|---------------|-----------------------------------------------|
| `fs-i18n`     | Sprachmetadaten, FTL-Strings                  |
| `gix`         | Git-Operationen (Language-Packs pushen)        |
| `fs-db`       | DbEngine-Trait (Spracheinstellungen speichern) |
