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
A1. [X] SQLite-Speicherung implementieren
    - [x] fsn-shared.db: übergreifende Settings, i18n-Auswahl, Audit-Log — SeaORM via fsd-db
    - [x] fsn-desktop.db: Widget-Positionen, Shortcuts, Profil, Layout, aktives Theme — SeaORM via fsd-db
    - fsn-conductor.db: Service-Konfigurationen, Quadlets, Variablen
    - fsn-store.db: Installierte Pakete, Versionen, Cache
    - fsn-core.db: Hosts, Projekte, Einladungen, Federation
    - fsn-bus.db: Event-Log, Routing-Regeln, Standing Orders, Subscriptions
    - Tabellen definieren, Migrations, SeaORM-Entities
    - JEDE Einstellung MUSS in SQLite landen
    - Beim Start: DB lesen → UI befüllen

A2. [X] Window X-Button fixen

A3. [X] FsnObject-System implementieren (siehe technik/ui-objekte.md)
    - Resize: 5px Toleranz, alle Kanten + Ecken, Fullscreen-Overlay
    - Drag: Am Kopf, Fullscreen-Overlay
    - Minimize: → Icon mit pulsierendem grünem Punkt
    - Close: Unsaved-Changes-Dialog
    - Object-Sidebar: Icons / Icons+Text bei Hover
    - EINMAL definieren, für Window/Widget/Modal/Panel nutzen

A4. [X] Scrollbars global (.fsn-scrollable überall)
```

## Phase B: Theme-System reparieren

```
B1. [x] CSS-Variablen-Namenskonvention (siehe technik/css.md)
    - Shared-Themes OHNE Prefix (Store-Format)
    - Programm-spezifisch MIT Prefix (--fsn-*)
    - Parser: prüft Pflicht-Variablen (validate_theme_vars in theme_loader.rs + appearance.rs)

B2. [x] Kontraste fixen (WCAG AA: 4.5:1 Text, 3:1 Muted)
    - Nordic text-muted: #4C566A → #7B8FA6 (ratio 1.82 → 3.96)
    - Cupertino text-muted: #aeaeb2 → #686868 (ratio 1.94 → 4.60)
    - Cloud-White text-muted: #94a3b8 → #626d79 (ratio 2.30 → 4.52)
    - Cloud-White text-secondary: #475569 → #334155 (verbessert)

B3. [x] Theme-Prefixing (beim Laden dynamisch hinzufügen)
    - prefix_theme_css() in fsd-shell/src/theme_loader.rs

B4. [x] Theme-Editor in Settings (ändern, preview, speichern, hochladen)
    - CSS-Editor Textarea in AppearanceSettings, Live-Validation, Apply-Preview

B5. [x] Theme-Aspekte konfigurierbar (Window-Chrome, Buttons, Animationen, Mauszeiger)
    - Animations-Toggle (--fsn-anim-duration: 0ms/180ms)
    - Window-Transparency Slider (--fsn-window-bg opacity)
    - Beide als Dioxus-Context in Desktop, editierbar in Settings
```

## Phase C: Store funktionsfähig

```
C1. [X] Store-Catalog TOML fixen
C2. [X] Sprachen installierbar (Download, DB-Registrierung, Dropdown)
C3. [X] Themes installierbar (Download, DB, Live-Wechsel, Lösch-Dialog)
C4. [X] Widgets installierbar
C5. [X] Store-UI: Alle Pakettypen, Tabs, Suche über Tags, Detail-View
```

## Phase D: Widgets & Desktop

```
D1. [ ] DraggableWidget (basierend auf FsnObject, Position in SQLite)
D2. [ ] ResizableContainer (basierend auf FsnObject, Größe in SQLite)
D3. [ ] Widget-Bearbeitungsmodus (Rechtsklick → Edit Desktop)
D4. [ ] Desktop-Hintergrund (Bild, URL, Farbe, Gradient)
D5. [ ] Basis-Widgets (Clock, SysInfo, Messages-Placeholder, Tasks-Placeholder)
```

## Phase E: Conductor neu bauen

```
E1. [ ] Alten Conductor-Code LÖSCHEN (podman.sock, bollard)
E2. [ ] YAML-Parser (Services, Subservices, Volumes, Networks, Ports)
E3. [ ] Variablen-Analyse
    - Typ-Erkennung (*_HOST, *_PASSWORD, *_URL)
    - Rollen-Erkennung mit Ober-/Unterrollen
    - Redis-kompatible: Redis, Dragonfly, KeyDB, Valkey, Memcached
    - Datenbanken: Postgres, MySQL, MariaDB, MongoDB, SQLite, CockroachDB
    - Konfidenz-Angabe
E4. [ ] Dry-Run + Validierung (Syntax, Healthcheck, Netzwerke, Ports)
E5. [ ] Quadlet-Generator (YAML → .container, Tera-Templates, kein Podman-Socket)
E6. [ ] Store-Integration (ergänzen nicht überschreiben, offline-fähig)
E7. [ ] Instanz-Namen vergeben
    - Benutzer vergibt Namen an jeden Service
    - Subservices: Auto-Prefix/Postfix (z.B. outline-postgres)
    - In SQLite speichern
```

## Phase F: Message Bus

```
F1. [ ] Bus-Grundstruktur (fsn-bus Crate)
    - Pub/Sub Event-System
    - BusEvent Struct (siehe konzepte/bus.md)
    - DeliveryType: FireAndForget, Guaranteed, StandingOrder
    - StorageType: NoStore, UntilAck, Persistent
    - In-Process Channel (tokio::broadcast) für lokalen Bus

F2. [ ] Rollen-basierte Adressierung
    - Events IMMER über Rollen adressiert, nie direkt
    - Subscription-Filter: Rolle + Topic + Instanz-Tag (kombinierbar)
    - Rollen-Hierarchie: Ober-Rolle = alle Unterrollen

F3. [ ] Subscription-Manager
    - Services registrieren sich mit Rolle + Topic-Filter
    - Rechte-Prüfung: Hat der Abonnent read-Recht auf den Sender?
    - Subscriptions in SQLite (fsn-bus.db)

F4. [ ] Nachrichtentypen-Handling
    - Fire & Forget: Sofort zustellen, dann verwerfen
    - Guaranteed Delivery: Queue, warten auf ACK, Retry
    - Standing Order: Dauerhaft aktiv, feuert bei passender Service-Installation

F5. [ ] Speicherungs-Layer
    - NoStore: Nur im RAM
    - UntilAck: In SQLite bis ACK, dann löschen
    - Persistent: In SQLite, nie automatisch löschen (Audit-Log)

F6. [ ] Konfigurierbare Default-Zuordnung
    - Regelbasierte TOML-Konfiguration
    - WENN Topic X UND Quelle Y → DeliveryType + StorageType
    - Editierbar im Conductor
    - Default-Regeln mitliefern (search=Fire&Forget, rights=Guaranteed+Persistent, etc.)

F7. [ ] Standing Orders Engine
    - Standing Orders in SQLite speichern
    - Bei Service-Installation: Prüfe ob passende Standing Orders existieren
    - Nur prüfen wenn Rolle des neuen Service zu einer Standing Order passt

F8. [ ] Bridges (Bus-zu-Bus Verbindung)
    - Bridge = aktiver Filter zwischen zwei Bus-Ebenen
    - Filtert nach Rechte-Kaskade (read, write, execute, search)
    - Doppelter Schutz: Bridge filtert + Empfänger-Bus hat eigene Regeln
    - Konfigurierbar: Welche Events dürfen durch

F9. [ ] Bus-API
    - CLI: fsn bus publish, fsn bus subscribe, fsn bus rules, fsn bus log
    - API: POST /api/bus/publish, GET /api/bus/subscribe (WebSocket)
    - UI: Bus-Monitor im Desktop (Events live sehen)
```

## Phase G: Lenses

```
G1. [ ] Lens-Datenmodell (Name, Thema, Service-Filter, Layout → SQLite)
G2. [ ] Lens-View (Ergebnisse gruppiert nach Rolle, Summary + Link)
G3. [ ] Lens als Desktop-Icon (platzierbar, öffnet als FsnObject-Fenster)
```

## Phase H: Search

```
H1. [ ] Search-View (Suchfeld, Ergebnisse gruppiert, Preview + Link + Quelle)
H2. [ ] Service-Suche (Ebene 1: jeder Service durchsucht eigene Daten)
H3. [ ] Host-Suche (Ebene 2: Bus aggregiert, dedupliziert)
H4. [ ] Föderale Suche (Ebene 3-4: nur mit search-Recht, über Bridges)
```

## Phase I: Bots

```
I1. [ ] Bot-Framework (BotDefinition, BotConfig in SQLite)
I2. [ ] Broadcast-Bot (Telegram zuerst)
I3. [ ] Gatekeeper-Bot (Join-Event → Verifikation → Approve/Deny)
```

## Phase J: Tasks

```
J1. [ ] Data Offers/Accepts System (konkrete Felder mit Typen)
J2. [ ] Task Builder UI (Feld-Mapping, Trigger: Event/Scheduled/Manual)
J3. [ ] Task-Templates aus Store
```

## Phase K: Node (Invite, Federation)

```
K1. [ ] Invite-System (Token, verschlüsselte TOML, Port pro Einladung)
K2. [ ] Federation-Grundstruktur (Beitritt, Domain-Pflicht, Auth-Broker)
K3. [ ] Rechte-Kaskade (read/write/execute/search, Audit-Log, Warnungen)
K4. [ ] Föderaler Bus (Bridge-Konfiguration zwischen Projekt und Föderation)
```

## Phase L: Shortcuts, Menü, Profil, Polish

```
L1. [ ] Action Registry (ID + Default-Shortcut)
L2. [ ] Konfigurierbare Shortcuts in Settings
L3. [ ] Hilfe: Auto-generierte Shortcut-Referenz
L4. [ ] Menü: JEDER Punkt ruft echte Aktion auf
L5. [ ] Profil: IAM-Daten + editierbar + Account-Linking
L6. [ ] Notification Bell
L7. [ ] Context-Menüs
L8. [ ] Animationen konfigurierbar
L9. [ ] Alle Stubs/toten Code entfernen
```

---

## Reihenfolge

```
Prio 1:  A1-A4     Fundament (SQLite, X-Button, FsnObject, Scrollbars)
Prio 2:  B1-B5     Theme-System reparieren + Kontraste
Prio 3:  C1-C5     Store funktionsfähig
Prio 4:  D1-D5     Widgets & Desktop
Prio 5:  E1-E7     Conductor neu bauen
Prio 6:  F1-F9     Message Bus
Prio 7:  G1-G3     Lenses
Prio 8:  H1-H4     Search
Prio 9:  I1-I3     Bots
Prio 10: J1-J3     Tasks
Prio 11: K1-K4     Node (Invite + Federation)
Prio 12: L1-L9     Polish
```
