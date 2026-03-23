# Language Manager

[← Zurück zum Index](../../INDEX.md) | [Manager-Konzept](../../konzepte/manager.md) | [i18n-Technik](../../technik/i18n.md)

---

## Was der Language Manager tut

Der Language Manager verbindet Sprachpräferenzen, heruntergeladene Übersetzungsdateien und die laufende UI.

Er ist in zwei Teile aufgeteilt:

| Teil | Repo | Zuständigkeit |
|---|---|---|
| `fs-manager-language` | `fs-managers` | Domain-Logik: Präferenzen, Packs, Format-Utilities |
| `language_panel.rs` | `fs-desktop/crates/fs-managers` | UI: LayoutB (Sidebar + Detail), drei Tabs |

---

## Drei-Schichten-Modell

```
1. Built-in (Compile-Time)
   fs-i18n enthält Snippets für "en" und "de" — immer verfügbar, kein Download nötig.
   Jedes Crate kann via SnippetPlugin eigene Keys hinzufügen.

2. Store Language Packs
   Pro installiertem Package werden .toml-Dateien für jede abonnierte Sprache gespeichert.
   Pfad: ~/.local/share/fs/i18n/{package_id}/{lang_code}.toml
   Registry: ~/.local/share/fs/i18n/registry.toml

3. Abonnements (subscribed_languages)
   Der User abonniert Sprachen (Standard: ["en", "de"]).
   Bei jeder Package-Installation werden automatisch Packs für alle abonnierten Sprachen geladen.
```

---

## Datenspeicherung

### Benutzereinstellungen

```
~/.config/fs/locale_settings.toml
```

Zwei-Schichten-Merge: Store-Default + Inventory-Override (Union bei `subscribed_languages`).

```toml
language = "de"
fallback_language = "en"
date_format = "d_m_y"
time_format = "h24"
number_format = "europe_dot"
auto_update_packs = true
subscribed_languages = ["de", "en", "fr"]
```

### Pack-Registry

```
~/.local/share/fs/i18n/registry.toml
```

Verfolgt alle installierten Language Packs (Sprache, Package-ID, Version, Dateipfad).
Getrennt von den Benutzereinstellungen, da sie installierte Metadaten enthält.

---

## Domain-Typen (`fs-manager-language`)

### `Language`

```rust
use fs_manager_language::{Language, HasFlag};

let lang = Language::from_code("fr");
// Verwendet fs_i18n::language_meta() — 50 Sprachen aus LanguageMeta-Registry
// → Language { id: "fr", display_name: "Français", locale: "fr-FR" }

lang.flag_svg()           // SVG-Markup der Landesflagge (via HasFlag-Trait)
lang.meta()               // Option<&LanguageMeta> mit Name, Schrift, Familie, Kontinent
lang.direction_label()    // "Left-to-right" / "Right-to-left"
```

**`HasFlag`-Trait** (OOP statt freier Funktion):
```rust
pub trait HasFlag {
    fn flag_svg(&self) -> &'static str;
}
impl HasFlag for Language { ... }
```

### `InstalledLanguagePack` + `LanguagePackRegistry`

```rust
let registry = mgr.registry();

registry.packs_for_lang("de")          // alle Packs für Deutsch
registry.packs_for_package("fs-store") // alle Packs dieses Packages
registry.is_installed("fr", "fs-ui")   // bool
registry.installed_lang_codes()        // Vec<String>
```

---

## API (`fs-manager-language`)

```rust
use fs_manager_language::LanguageManager;

let mgr = LanguageManager::new();

// Aktive Sprache
let lang = mgr.active();
mgr.set_active("de")?;

// Abonnements verwalten
let subs = mgr.subscribed_languages();   // ["en", "de", "fr"]
mgr.subscribe("fr")?;
mgr.unsubscribe("fr")?;

// Pack-Verwaltung
let packs = mgr.installed_packs();
mgr.is_installed("de", "fs-ui");
mgr.download_for_package("fs-store")?;  // lädt Packs für alle abonnierten Sprachen
mgr.download_for_language("fr")?;       // lädt Packs aller installierten Packages für "fr"
mgr.load_into_i18n("fs-store", &mut i18n); // lädt aktiven Pack in I18n-Instanz

// Verfügbare Sprachen (aktiv + abonniert + installiert + "en")
let available = mgr.available();

// Locale-Formate
let s = mgr.effective_settings();
s.format_date(2026, 3, 20)      // "20.03.2026" / "03/20/2026"
s.format_time(14, 5)            // "14:05" / "02:05 PM"
s.format_integer(1_234_567)     // "1.234.567" / "1,234,567"
s.format_decimal(1234.56, 2)    // "1.234,56" / "1,234.56"
```

---

## UI: LayoutB (Sidebar + Detail)

```
┌──────────────────────────────────────────────────────────┐
│  [AKTIV]  Deutsch (de)     ×  Abbestellen   Einstellungen│
│  ──────────────────────────                              │
│  Abonniert                 │  Info │ Konfiguration │ Übersetzen │
│  > Deutsch (de)  [AKTIV]   │                              │
│    English (en)  [Eingebaut│  Name:    Deutsch            │
│    Français (fr)           │  Eigenname: Deutsch          │
│  ──────────────────────────│  Schrift: Lateinisch         │
│  + Abonnieren              │  Familie: Germanisch         │
│  ──────────────────────────│  Kontinent: Europa           │
│  ⚙ Datumsformate           │                              │
│                            │  Pakete: 3 installiert       │
│                            │  [Fehlende herunterladen]    │
└──────────────────────────────────────────────────────────┘
```

### Sidebar

- **AKTIV-Chip** oben — zeigt aktive Sprache mit Flagge
- **Abonniert-Liste** — alle abonnierten Sprachen, klickbar für Detail
- **+ Abonnieren** — öffnet `SubscribeView` (50-Sprachen-Suche aus `fs_i18n::all_languages()`)
- **⚙ Datumsformate** — öffnet `FormatsView` (Datum/Zeit/Zahlenformat global)

### Tab: Info

- Metadaten der ausgewählten Sprache (Name, Eigenname, Schrift, Familie, Kontinent, Schreibrichtung)
- Liste installierter Packs + "Fehlende herunterladen"-Button
- "Aktivieren"- / "Abbestellen"-Aktionen

### Tab: Konfiguration

- Datum-, Zeit-, Zahlenformat (Button-Gruppe + Live-Vorschau)
- Fallback-Sprache (Dropdown)
- Auto-Update-Packs (Checkbox)
- Speichert sofort via `LanguageManager`

### Tab: Übersetzen (Builder)

- **GitHub-SSH-Status-Badge** (via `GitContributorCheck::cached()`):
  - `✓ GitHub: @username` — SSH-Auth OK, "Commit & Push" verfügbar
  - `✕ No SSH key` — nur Export
  - `… Checking` — läuft im Hintergrund, gecacht 7 Tage
- Übersetzungseditor-Platzhalter (wird in Phase G vollständig implementiert)

---

## LangSwitcher (Taskbar)

Ein `LangSwitcher`-Badge in der System-Tray-Zone der Taskbar:

```
[DE]  ← zeigt aktiven Sprachcode, klickbar
```

Öffnet ein Upward-Dropdown mit allen abonnierten Sprachen.
Klick auf einen Eintrag ruft `on_switch_lang` auf → `LanguageManager::set_active()`.

```rust
// In Taskbar einbinden:
Taskbar {
    apps: apps,
    on_launch: handler,
    active_lang: Some("de".into()),
    subscribed_langs: vec!["de".into(), "en".into(), "fr".into()],
    on_switch_lang: Some(switch_handler),
}
```

---

## Re-Render bei Sprachwechsel

Das `LangContext`-Signal (`Signal<String>`) ist der Reaktivitäts-Anker:

1. Sprachwechsel → `fs_i18n::set_active_lang()` + `LangContext.write(active_lang)`
2. `Desktop` liest `LangContext` → re-rendert bei Änderung
3. `data-lang`-Attribut auf `#fs-desktop` ändert sich → Dioxus-Diff aktualisiert alle Kinder
4. App-Fenster mit `use_context::<LangContext>()` rendern ebenfalls neu

---

## GitContributorCheck

`fs_manager_language::git_contributor::GitContributorCheck`:

1. Prüft ob `~/.ssh/id_ed25519` (oder ähnliche) existiert
2. Führt `ssh -T git@github.com` aus → erwartet `"Hi <username>!"`
3. Cached 7 Tage in `~/.config/fs/git_contributor.toml`

---

Weiter: [Manager-Konzept](../../konzepte/manager.md) | [i18n-Technik](../../technik/i18n.md) | [Store](../store/README.md)
