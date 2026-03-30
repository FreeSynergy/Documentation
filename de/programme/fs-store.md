# fs-store

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-store` · `/home/kal/Server/fs-store/`
**Typ:** Program (Workspace: 3 Crates)
**Capabilities:** `store.catalog`, `store.search`, `store.install`, `store.update`

---

## Was ist das?

`fs-store` ist der Paketmanager von FreeSynergy.
Er liest den Katalog aus `FreeSynergy/Store` (Git-Repo), sucht Pakete, installiert, deinstalliert und aktualisiert sie.
Erkennt automatisch ob ein Display vorhanden ist — GUI wenn ja, CLI sonst.

---

## Workspace-Struktur

```
fs-store/crates/
├── fs-store      — Core-Library: Katalog, Suche, Install-Engine
├── fs-store-app  — GUI-App (iced via fs-gui-engine-iced)
└── fs-store-cli  — CLI-Binary
```

---

## Design

```
Store (Facade)
    ├── CatalogReader (Trait)     — search / list / get / info / check-updates
    ├── PackageInstaller (Trait)  — install / uninstall / update / validate
    ├── BundleLoader (Trait)      — load-bundle / list-bundles / install-bundle
    └── PackagePointer            — id + sources-URLs + interfaces
```

**Pointer-Model:** `package.toml` im Store-Katalog zeigt auf Source-Repos —
der Store speichert keine Binaries, nur Metadaten.

---

## Pakettypen

| Typ        | Beschreibung                                          |
|------------|-------------------------------------------------------|
| `bundle`   | Gruppe von Paketen (z.B. "Workstation", "Server")     |
| `program`  | Laufender Container-Dienst                            |
| `adapter`  | Implementiert einen Trait, registriert Capability     |
| `artifact` | Reine Daten (Sprachen, Themes, Icon-Sets)             |

---

## CLI

```bash
# Pakete suchen
fs-store-cli search kanidm

# Paket installieren
fs-store-cli install fs-node

# Bundle installieren
fs-store-cli install workstation

# Update prüfen
fs-store-cli check-updates

# Alle installierten Pakete
fs-store-cli list

# Paket deinstallieren
fs-store-cli uninstall fs-node
```

---

## Store-Katalog (`FreeSynergy/Store`)

Der Katalog ist ein separates Git-Repo (`/home/kal/Server/Store/`).
`fs-init` klont ihn nach `~/.local/share/fsn/store/`.
`fs-store` liest ihn von dort.

```
Store/
├── bundles/workstation.toml
├── bundles/server.toml
├── packages/fs-node/package.toml
├── packages/fs-desktop/package.toml
└── …
```
