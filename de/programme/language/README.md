# Language Manager

[← Zurück zum Index](../../INDEX.md) | [Manager-Konzept](../../konzepte/manager.md) | [i18n-Technik](../../technik/i18n.md)

---

## Was der Language Manager tut

Der Language Manager ist das zentrale Stück zwischen gespeicherter Sprachpräferenz, den geladenen Übersetzungsdateien und der laufenden UI.

Er ist in zwei Teile aufgeteilt:

| Teil | Repo | Zuständigkeit |
|---|---|---|
| `fsn-manager-language` | FreeSynergy.Managers | Lesen/Schreiben der Präferenzen, Format-Utilities |
| `fsd-settings` (LanguageSettings) | FreeSynergy.Desktop | UI: Tab-Struktur, Sprache wechseln, Packs installieren, Übersetzen |

---

## Datenspeicherung

Alle Sprachpräferenzen landen in einer einzigen Datei:

```
~/.config/fsn/locale_settings.toml
```

Beispiel:

```toml
language = "de"
fallback_language = "en"
date_format = "d_m_y"
time_format = "h24"
number_format = "europe_dot"
auto_update_packs = true
```

- **Schreiben:** `LanguageManager::set_active()` / `LanguageManager::save_settings()`
- **Lesen:** `LanguageManager::effective_settings()` — Store-Defaults + Inventory-Overrides gemergt
- **Startup:** `load_active_language()` in fsd-settings liest über `LanguageManager`

---

## API (`fsn-manager-language`)

```rust
use fsn_manager_language::{LanguageManager, language_from_code};

let mgr = LanguageManager::new();

// Aktive Sprache lesen
let lang = mgr.active();          // Language { id: "de", display_name: "Deutsch", locale: "de-DE" }

// Sprache wechseln (schreibt locale_settings.toml)
mgr.set_active("de")?;

// Locale-Settings (nach Merge: Store-Default + User-Override)
let s = mgr.effective_settings();
s.format_date(2026, 3, 20)        // "20.03.2026" / "03/20/2026" / "2026-03-20"
s.format_time(14, 5)              // "14:05" / "02:05 PM"
s.format_integer(1_234_567)       // "1.234.567" / "1,234,567" / "1 234 567"
s.format_decimal(1234.56, 2)      // "1.234,56" / "1,234.56" / "1 234,56"

// Language-Objekt aus Code bauen
let lang = language_from_code("fr");
// → Language { id: "fr", display_name: "Français", locale: "fr-FR" }
```

Bekannte Sprachen in `language_from_code()`: en, de, fr, es, it, pt, nl, pl, ru, ja, zh, ko, ar.
Unbekannte Codes werden durchgereicht (id = display_name = locale).

---

## UI: Drei-Tab-Struktur (`fsd-settings`)

### Tab 1 — Active Language

- Liste aller installierten Sprachpakete (built-in "en" + PackageRegistry `kind = "language"`)
- Auswahl + Apply-Button
- Apply: lädt den Pack aus `~/.local/share/fsn/i18n/{lang}/ui.toml`, schreibt `locale_settings.toml`,
  schaltet `fsn_i18n` um, schreibt `LangContext`-Signal → Desktop re-rendert

**Locale-Formate** (direkt im selben Tab):
- Fallback-Sprache (Dropdown)
- Datum-, Zeit-, Zahlenformat (Button-Gruppe mit Vorschau-Beispiel)
- Auto-Update-Packs (Checkbox)

Alle Formate speichern sofort (kein Apply-Button nötig).

### Tab 2 — Install

Direkte Store-Ansicht gefiltert auf `resource_type = "language"`:
- Download + Registrierung im PackageRegistry
- Nach Installation sofort in Tab 1 sichtbar

### Tab 3 — Edit / Create

Für Übersetzer und Mitwirkende:

- **GitHub-SSH-Status-Badge** (oben rechts im TranslationEditor):
  - `✓ GitHub: @username` — SSH-Authentifizierung OK, "Commit & Push" sichtbar
  - `✕ No SSH key` — nur "Export .toml" verfügbar
  - `… Checking` — läuft im Hintergrund, gecacht 7 Tage

- **TranslationEditor** öffnet als Vollbild-Overlay:
  - Side-by-Side: EN (readonly) | Zielsprache (editierbar)
  - Fortschrittsbalken: X / Y Schlüssel übersetzt
  - Filter: nur fehlende Schlüssel, Schlüssel-Suche
  - **Export .toml** → `~/Downloads/fsn-{lang}.toml` + lokale Installation
  - **Commit & Push** (nur mit SSH) → in `~/FreeSynergy.Node` committen und pushen
    (Pfad überschreibbar mit `FSN_NODE_REPO_PATH`)

#### LLM-Assist

Erscheint nur, wenn in den **Service Roles** ein Eintrag mit dem Schlüssel `"llm"` konfiguriert ist
(`load_role_assignments().get("llm")`).

Bei Klick auf "🤖 AI Translate":
- Aufklapp-Panel mit Prompt (bis zu 50 fehlende Keys + EN-Referenz, automatisch vorausgefüllt)
- User kann den Prompt ergänzen (z. B. "Halte technische Begriffe auf Englisch")
- Sendet an:
  - `ollama` → `http://localhost:11434/api/generate`, Modell `llama3.2`
  - alles andere → `http://localhost:8080/v1/chat/completions` (OpenAI-kompatibel)
- Antwort wird als TOML geparst → Vorschlagsliste
- Einzelne Vorschläge mit ✓ akzeptieren oder "Accept All"
- Nichts wird automatisch gespeichert

---

## Re-Render bei Sprachwechsel

Das `LangContext`-Signal (`Signal<String>`) ist der Reaktivitäts-Anker:

1. Apply → `fsn_i18n::set_active_lang()` + `LangContext.write(active_lang)`
2. `Desktop` liest `LangContext` → subscribed → re-rendert bei Änderung
3. `data-lang`-Attribut auf `#fsd-desktop` ändert sich → Dioxus-Diff aktualisiert alle inline RSX-Kinder (Header, Sidebar, Taskbar)
4. App-Fenster, die selbst `use_context::<LangContext>()` lesen, rendern ebenfalls neu

---

## GitContributorCheck

`fsn-manager-language::git_contributor::GitContributorCheck`:

1. Prüft ob `~/.ssh/id_ed25519` (oder ähnliche) existiert
2. Führt `ssh -T git@github.com` aus → erwartet `"Hi <username>!"`
3. Cached 7 Tage in `~/.config/fsn/git_contributor.toml`

---

Weiter: [Manager-Konzept](../../konzepte/manager.md) | [i18n-Technik](../../technik/i18n.md) | [Store](../store/README.md)
