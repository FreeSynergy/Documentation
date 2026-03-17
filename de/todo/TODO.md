# TODO-Liste

[← Zurück zum Index](../INDEX.md) | [Bugs](BUGS.md)

**Regeln:**
- Jeder Punkt wird KOMPLETT umgesetzt. Keine Stubs.
- IMMER `cargo build` vor und nach jeder Änderung.
- Altes das nicht passt → LÖSCHEN und neu machen.
- Kein Menüpunkt ohne Funktion. Kein Button der nichts tut.

---

## Phase A: Fundament (MUSS ZUERST)

```
A1. [x] SQLite-Speicherung implementieren
    - fsn-shared.db: übergreifende Settings, i18n-Auswahl, Audit-Log
    - fsn-desktop.db: Widget-Positionen, Shortcuts, Profil, Layout, aktives Theme
    - fsn-conductor.db: Service-Konfigurationen, Quadlets, Variablen
    - fsn-store.db: Installierte Pakete, Versionen, Cache, Signaturen
    - fsn-core.db: Hosts, Projekte, Einladungen, Federation
    - fsn-bus.db: Event-Log, Routing-Regeln, Standing Orders, Subscriptions

A2. [x] Window X-Button fixen

A3. [x] FsnObject-System implementieren (siehe technik/ui-objekte.md)
    - Resize: 5px Toleranz, alle Kanten + Ecken
    - Drag: Am Kopf
    - Minimize: → Icon mit pulsierendem grünem Punkt
    - Close: Unsaved-Changes-Dialog
    - Object-Sidebar: Icons / Icons+Text bei Hover

A4. [x] Scrollbars global (.fsn-scrollable überall)
```

## Phase B: Theme-System reparieren

```
B1. [ ] CSS-Variablen-Namenskonvention (siehe technik/css.md)
    - Shared-Themes OHNE Prefix im Store
    - Programm-spezifisch MIT Prefix beim Laden
    - Parser: prüft Pflicht-Variablen

B2. [ ] Kontraste fixen (WCAG AA: 4.5:1 Text, 3:1 Muted)

B3. [ ] Theme-Prefixing (beim Laden dynamisch)

B4. [ ] Theme-Editor in Settings

B5. [ ] Theme-Aspekte konfigurierbar (Chrome, Buttons, Animationen, Mauszeiger)
```

## Phase C: Init + Store als Paketmanager

```
C1. [ ] FreeSynergy.Init erstellen
    - Minimales Rust-Binary
    - gitoxide (gix) für Git-Klon des Store-Repos
    - GitHub Actions: Cross-Compilation für Linux/macOS/Windows
    - Repo: FreeSynergy/Init

C2. [ ] Store-Paketmanager: Kern
    - Paket-Typen: program, container, group, language, theme, widget, bot, bridge, task
    - Pflicht-Metadaten: id, name, version, type, description, icon, tags
    - catalog.toml: maschinenlesbarer Index
    - SQLite: installierte Pakete, Versionen, Status

C3. [ ] Store: Versionierung
    - SemVer + Git-Tags
    - Parallele Versionen (installiert mit Versionsnummer)
    - Release-Channels: stable, testing, nightly
    - Rollback auf vorherige Version

C4. [ ] Store: Paket-Signierung
    - ed25519 als Default (austauschbar: ed448, RSA)
    - Signatur-Prüfung vor Installation
    - --trust-unsigned Flag (mit Warnung)

C5. [ ] Store: Abhängigkeits-Auflösung
    - Dependency-Solver
    - Konflikte erkennen und warnen

C6. [ ] Store: Gruppen (Meta-Pakete)
    - server-minimal, server-full, desktop, desktop-full
    - packages + optional Liste

C7. [ ] Store: Install-Scripts
    - pre_install, post_install, pre_remove, post_remove
    - Alte fsn-install.sh aufteilen in Paket-Scripts

C8. [ ] Store: CLI komplett
    - search, info, install, update, rollback, remove, list, sync

C9. [ ] Store: API komplett
    - REST-Endpoints für alle CLI-Funktionen
    - /api/store/know/* Wissens-Endpoints

C10. [ ] Store: Sprachen installierbar
    - .ftl-Dateien laden, registrieren
    - Nur installierte Sprachen im Dropdown
    - Englisch vorinstalliert

C11. [ ] Store: Themes installierbar (Download, Live-Wechsel)

C12. [ ] Store: Widgets installierbar

C13. [ ] Store-UI: Alle Pakettypen, Tabs, Tags, Suche, Detail-View
```

## Phase D: S3-Storage-Layer

```
D1. [ ] S3-Server in Node einbauen
    - s3s Crate: S3-API bereitstellen
    - s3s-fs: Lokales Dateisystem als erstes Backend
    - Automatisch beim Node-Start starten

D2. [ ] opendal Integration
    - Backend austauschbar: lokal, SFTP, S3
    - Hetzner Storagebox als SFTP-Backend
    - Konfigurierbar in Node-Config

D3. [ ] Bucket-Struktur
    - /profiles/ (öffentlich lesbar über Proxy)
    - /backups/ (nur intern)
    - /media/{service}/ (pro Service)
    - /packages/ (Store-Cache)
    - /shared/ (geteilte Dateien)

D4. [ ] Öffentliche Profile (Visitenkarten)
    - JSON + Avatar pro User
    - Über Zentinel extern erreichbar
    - Automatisch abrufbar bei Föderation-Beitritt

D5. [ ] Verteilter Storage über mehrere Hosts
    - Nodes reden untereinander S3
    - opendal verbindet mehrere Backends
```

## Phase E: Widgets & Desktop

```
E1. [ ] DraggableWidget (basierend auf FsnObject, Position in SQLite)
E2. [ ] ResizableContainer (basierend auf FsnObject, Größe in SQLite)
E3. [ ] Widget-Bearbeitungsmodus (Rechtsklick → Edit Desktop)
E4. [ ] Desktop-Hintergrund (Bild, URL, Farbe, Gradient)
E5. [ ] Basis-Widgets (Clock, SysInfo, Messages, Tasks)
```

## Phase F: Conductor neu bauen

```
F1. [ ] Alten Conductor-Code LÖSCHEN
F2. [ ] YAML-Parser (Services, Subservices, Volumes, Networks, Ports)
F3. [ ] Variablen-Analyse (Typen, Rollen, Ober-/Unterrollen, Konfidenz)
F4. [ ] Dry-Run + Validierung
F5. [ ] Quadlet-Generator (kein Podman-Socket)
F6. [ ] Store-Integration (ergänzen, nicht überschreiben)
F7. [ ] Instanz-Namen (Benutzer vergibt, Subservices Auto-Prefix)
```

## Phase G: Message Bus

```
G1. [ ] Bus-Grundstruktur (Pub/Sub + Direct Messages)
G2. [ ] Rollen-basierte Adressierung (nie direkt an Service)
G3. [ ] Subscription-Manager (Rolle + Topic + Instanz-Tag Filter)
G4. [ ] Nachrichtentypen (Fire&Forget, Guaranteed, Standing Order)
G5. [ ] Speicherungs-Layer (NoStore, UntilAck, Persistent)
G6. [ ] Konfigurierbare Default-Zuordnung (regelbasiert, TOML)
G7. [ ] Standing Orders Engine
G8. [ ] Bridges (Bus-zu-Bus, Rechte-Kaskade, doppelter Checkpoint)
G9. [ ] Bus-API (CLI, REST, WebSocket)
```

## Phase H: Lenses

```
H1. [ ] Lens-Datenmodell (SQLite)
H2. [ ] Lens-View (gruppiert nach Rolle, Summary + Link)
H3. [ ] Lens als Desktop-Icon
```

## Phase I: Search

```
I1. [ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview + Link)
I2. [ ] Service-Suche (Ebene 1)
I3. [ ] Host-Suche (Ebene 2, Bus-aggregiert)
I4. [ ] Föderale Suche (Ebene 3-4, nur mit search-Recht)
```

## Phase J: Bots

```
J1. [ ] Bot-Framework
J2. [ ] Broadcast-Bot (Telegram)
J3. [ ] Gatekeeper-Bot
```

## Phase K: Tasks

```
K1. [ ] Data Offers/Accepts
K2. [ ] Task Builder UI
K3. [ ] Task-Templates aus Store
```

## Phase L: Node (Invite, Federation)

```
L1. [ ] Invite-System (Token, verschlüsselte TOML, Port pro Einladung)
L2. [ ] Federation-Grundstruktur (Domain-Pflicht, Auth-Broker)
L3. [ ] Rechte-Kaskade (read/write/execute/search, Audit-Log)
L4. [ ] Föderaler Bus (Bridge-Konfiguration)
```

## Phase M: Shortcuts, Menü, Profil, Polish

```
M1. [ ] Action Registry + konfigurierbare Shortcuts
M2. [ ] Hilfe: Auto-generierte Shortcut-Referenz
M3. [ ] Menü: JEDER Punkt ruft echte Aktion auf
M4. [ ] Profil: IAM + editierbar + Account-Linking + S3-Visitenkarte
M5. [ ] Notification Bell
M6. [ ] Context-Menüs
M7. [ ] Animationen konfigurierbar
M8. [ ] Alle Stubs/toten Code entfernen
```

---

## Reihenfolge

```
Prio 1:  A1-A4       Fundament (SQLite, X-Button, FsnObject, Scrollbars)
Prio 2:  B1-B5       Theme-System reparieren
Prio 3:  C1-C13      Init + Store als Paketmanager
Prio 4:  D1-D5       S3-Storage-Layer
Prio 5:  E1-E5       Widgets & Desktop
Prio 6:  F1-F7       Conductor neu
Prio 7:  G1-G9       Message Bus
Prio 8:  H1-H3       Lenses
Prio 9:  I1-I4       Search
Prio 10: J1-J3       Bots
Prio 11: K1-K3       Tasks
Prio 12: L1-L4       Node (Invite + Federation)
Prio 13: M1-M8       Polish
```
