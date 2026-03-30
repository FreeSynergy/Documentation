# fs-theme

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-theme` ist die Theme-Bibliothek für FreeSynergy — Definitionen, Loader und CSS-Generierung.
Sie ist eine reine Library (kein Daemon) und wird von UI-Komponenten importiert.

---

## Architektur

```
ThemeEngine     ← lädt Theme aus TOML oder CSS, generiert CSS-Output
ThemeRegistry   ← Registry Pattern: bekannte Themes verwalten
Theme           ← Datenstruktur: Farben, Fonts, Abstände, Animationen
Palette         ← Farb-Palette (Primary, Secondary, Surface, …)
```

---

## ThemeEngine-API

| Methode                    | Beschreibung                                    |
|----------------------------|-------------------------------------------------|
| `ThemeEngine::new(theme)`  | Engine aus Theme-Objekt                         |
| `from_default_theme()`     | Default-Theme (Cyan + White)                    |
| `from_toml(path)`          | Theme aus TOML-Datei laden                      |
| `from_toml_str(text)`      | Theme aus TOML-String laden                     |
| `from_css(css, name)`      | Theme aus CSS-Variablen parsen                  |
| `theme()`                  | Geladenes Theme-Objekt referenzieren            |
| `to_css()`                 | CSS-Variablen generieren                        |
| `to_full_css()`            | Vollständiges CSS inkl. Utility-Klassen         |
| `glass_css()`              | Glassmorphism-CSS (statisch)                    |
| `animations_css()`         | Animations-CSS (statisch)                       |
| `to_tailwind_config()`     | Tailwind-kompatible JSON-Config generieren      |

---

## ThemeRegistry

Verwaltet bekannte Themes als benannte Einträge. Themes können als Artifacts
global oder pro-Paket installiert werden (`theme::changed` Bus-Event).

---

## Repo

- Lokal: `/home/kal/Server/fs-theme/`
- GitHub: `git@github.com:FreeSynergy/fs-theme.git`

---

Weiter: [CSS-System](css.md) | [← Index](../INDEX.md)
