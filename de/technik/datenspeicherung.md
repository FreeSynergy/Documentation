# Datenspeicherung

[← Zurück zum Index](../INDEX.md)

---

## Grundregel

**Alles wird in SQLite gespeichert.** Keine losen Dateien für Zustandsdaten.

## Datenbanken

| Datenbank | Programm | Inhalt |
|---|---|---|
| `fsn-core.db` | Node | Hosts, Projekte, Federation, Einladungen |
| `fsn-container-app.db` | Container App Manager | Service-Konfigurationen, Quadlets, Variablen |
| `fsn-desktop.db` | Desktop | Widgets, Layout, Shortcuts, Profil |
| `fsn-store.db` | Store | Installierte Pakete, Cache, Versionen |
| `fsn-bus.db` | Bus | Event-Log, Routing-Regeln |
| `fsn-shared.db` | Alle | Übergreifende Settings, i18n-Auswahl, Audit-Log |

Jedes Programm hat seine eigene DB. `fsn-shared.db` ist für Daten die programmübergreifend sind.

## Was wo gespeichert wird

| Datenart | Speicherort | Warum |
|---|---|---|
| Konfiguration (TOML) | Dateisystem | Menschenlesbar, versionierbar |
| Runtime-Zustand | SQLite | Schnell, relational, querybar |
| Paket-Manifeste | SQLite + Dateisystem | Manifest als Datei, Metadaten in DB |
| Widget-Positionen | SQLite | Persönliche Einstellungen |
| Installierte Pakete | SQLite | Was installiert, welche Version |
| Shortcuts | SQLite | Benutzerdefinierte Tastenkürzel |
| Theme-Auswahl | SQLite | Aktives Theme |
| Profil-Daten | SQLite | Name, Links, Social |
| Secrets | SQLite (age-verschlüsselt) | Passwörter, Tokens, Keys |

## ORM

SeaORM 2.0 mit:
- `rusqlite` (sync) für CLI-Tools
- `sqlx` (async) für Server/API

---

Weiter: [Bibliotheken](bibliotheken.md) | [Sicherheit](sicherheit.md)
