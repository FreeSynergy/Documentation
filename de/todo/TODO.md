# TODO-Liste

[← Zurück zum Index](../INDEX.md) | [Bugs](BUGS.md)

**Regeln:**
- Jeder Punkt wird KOMPLETT umgesetzt. Keine Stubs.
- IMMER `cargo build` vor und nach jeder Änderung.
- Altes das nicht passt → LÖSCHEN und neu machen.
- Kein Menüpunkt ohne Funktion. Kein Button der nichts tut.
- Begriffe: **App** (nicht Programm), **Container-App** (nicht Container-Service), **Bundle** (nicht Gruppe), **Ressource** (alles im Store)

---

## Phase A–F: ✅ ERLEDIGT

Siehe Git-History. Fundament, Themes, Store, S3, Widgets, Conductor — alles implementiert.

---

## Phase G: Ressourcen-System & Inventory ✅

```
G1. [x] ResourceMeta Struct (fsn-types/resources/meta.rs)
    - ResourceType enum (16 Varianten: App, ContainerApp, Bundle, Widget, Bot, Bridge,
      Task, Language, ColorScheme, Style, FontSet, CursorSet, IconSet, ButtonStyle,
      WindowChrome, AnimationSet)
    - ValidationStatus (Ok, Incomplete, Broken) mit badge() ✅⚠️❌
    - Dependency struct (required/optional, version_req)
    - Role newtype wrapper

G2. [x] Typ-spezifische Structs (fsn-types/resources/)
    - AppResource (platforms, binary_name, locales, cli_commands, api_endpoints,
      roles_provided, roles_required)
    - ContainerAppResource (compose_yaml, services[], variables[], networks, volumes,
      apis, roles_provided, roles_required)
      inkl. ContainerVariable (VarType, AutoSource: InternalService/RoleVariable)
    - WidgetResource (required_roles mit ANY/ALL Modus, data_sources, sizes)
    - BotResource (channels, commands, triggers, tokens_required)
    - BridgeResource (target_role, target_service, methods[] mit FieldMapping)
    - BundleResource (packages[], theme mit ThemeBundleRefs)
    - ColorScheme, FontSet, CursorSet, IconSet, ButtonStyle, WindowChrome, AnimationSet

G3. [x] Style als standardisierte Ressource (StyleResource)
    - Striktes Schema: 19 Pflichtfelder (radius, spacing, shadows, borders,
      scrollbar, sidebar, transitions)
    - Validator: Alle Felder müssen vorhanden sein → sonst Broken
    - StyleTokens::default_tokens() liefert FreeSynergy-Standardwerte
    - Im Store einzeln ladbar

G4. [x] fsn-inventory Crate (eigene SQLite: fsn-inventory.db)
    - InstalledResource (resource_type, version, channel, status, config_path,
      data_path, validation)
    - ServiceInstance (instance_name, roles_provided, roles_required, bridges,
      variables, network, port, s3_paths)
    - BridgeInstance (role, service_instance, api_base_url, status)
    - Sea-ORM Entities + Migrations im SCHEMA const
    - Inventory::services_with_role(role) / bridges_for_role(role)

G5. [x] Validate trait für alle Ressource-Typen (fsn-types/resources/validator.rs)
    - Einheitliches Validate trait mit validate() → meta.status wird gesetzt
    - required_methods_for_role(role) definiert Minimal-API für iam/wiki/git/chat/
      database/cache/smtp/llm/map/tasks/monitoring
    - Bridge: alle Pflicht-Methoden gemappt → sonst Incomplete
    - Bundle: mindestens eine Referenz → sonst Broken
    - Container-App: YAML + Services + Healthcheck + Variablen-Beschreibungen
    - 61 Unit-Tests grün
```

## Phase H: Bridges & Rollen-APIs ✅

```
H1. [x] Rollen-API-Definitionen (fsn-types/resources/role_api.rs)
    - Typed request/response structs für alle 11 Rollen
    - IamUser*, WikiPage*, GitRepo*, ChatChannel*, DbQuery*, Cache*, Mail*, Llm*, Map*, Task*, Alert*

H2. [x] Bridge-Framework (fsn-bridge Crate in FreeSynergy.Lib)
    - BridgeExecutor: nimmt BridgeResource + base_url, führt HTTP-Calls aus
    - FieldMapping wird auf Request + Response angewendet
    - BridgeError Enum

H3. [x] Erste Bridges implementiert (fsn-bridge/src/catalog.rs)
    - kanidm_iam_bridge() → BridgeResource mit allen 8 IAM-Methoden
    - outline_wiki_bridge() → BridgeResource mit allen 4 Wiki-Methoden
    - forgejo_git_bridge() → BridgeResource mit allen 4 Git-Methoden
    - Alle validieren als ✅ Ok (Tests grün)

H4. [x] BridgeDispatcher (Inventory ↔ Bridge Integration)
    - BridgeDispatcher::execute(role, method, params) → Value
    - Sucht aktive BridgeInstance im Inventory für gegebene Rolle
    - Lädt BridgeResource aus Katalog, delegiert an BridgeExecutor
    - Bus-Integration folgt in Phase J
```

## Phase I: Resource Builder ✅

```
I1. [x] Builder-CLI (cli/crates/fsn-builder/)
    - fsn-builder analyze <compose.yml> → ContainerAppResource (TOML/JSON)
    - fsn-builder validate <dir> → ✅⚠️❌ mit Message
    - fsn-builder publish <dir> → signieren + git clone/push zum Store

I2. [x] Container-App-Analyse-Pipeline (fsn-builder/src/analyze.rs)
    - Docker Compose YAML → ContainerAppResource (vollständig typisiert)
    - Primary-Service-Erkennung (Infra vs. App)
    - Variablen-Analyse: Typ (Secret/Url/Hostname/Port/…) + Rolle + Konfidenz
    - AutoSource: InternalService (Sibling) oder RoleVariable (Inventory)
    - Netzwerk auto-generieren ({servicename}-backend)
    - Volumes → S3-Pfade vorschlagen
    - 5 Unit-Tests grün

I3. [x] Builder-UI im Desktop (fsd-studio → fsd-builder umbenannt)
    - Tab "Container App": YAML-Paste → ContainerAppResource, Variablen-Tabelle
      mit Typ + Rolle + Pflicht-Badge, Validierungs-Anzeige ✅⚠️❌
    - Tab "Bridge": BridgeBuilder — Rolle wählen, Methoden-Checkliste, validieren
    - "Save Locally" + "Publish to Store" Buttons (publish via CLI)
    - i18n Editor + Resource Browser bleiben erhalten
    - AI Assist (Ollama) für Metadaten-Anreicherung
```

## Phase J: Message Bus ✅

```
J1. [x] Bus-Grundstruktur (Pub/Sub + Direct Messages)
    - MessageBus struct (fsn-bus/message_bus.rs) — orchestriert Router, Subscriptions, StandingOrders, Config
    - BusMessage wrapper (delivery + storage policy)

J2. [x] Rollen-basierte Adressierung (nie direkt an Service)
    - Subscription(subscriber_role, topic_filter, inst_tag)
    - SubscriptionManager — gibt matching roles für jedes Event zurück

J3. [x] Subscription-Manager (Rolle + Topic + Instanz-Tag Filter)
    - SubscriptionManager::matching(topic, source_inst)
    - add / remove / for_role / iter

J4. [x] Nachrichtentypen (Fire&Forget, Guaranteed, Standing Order)
    - DeliveryType enum (fire-and-forget / guaranteed / standing-order)
    - StorageType enum (no-store / until-ack / persistent)

J5. [x] Speicherungs-Layer (NoStore, UntilAck, Persistent)
    - Schema definiert in fsd-db/schemas/bus.rs (event_log, subscriptions, routing_rules, standing_orders)
    - StorageType per BusMessage wählbar, per RoutingConfig override-fähig

J6. [x] Konfigurierbare Default-Zuordnung (regelbasiert, TOML)
    - RoutingConfig (TOML) mit RoutingRule (topic_pattern, source_role, delivery, storage, priority)
    - MessageBus::load_config_toml / load_config_file

J7. [x] Standing Orders Engine
    - StandingOrder (trigger_role, topic, payload, enabled)
    - StandingOrdersEngine::trigger_for_role / trigger_for_topic

J8. [x] Bridges (Bus-zu-Bus, Rechte-Kaskade, doppelter Checkpoint)
    - BusBridge (fsn-bus/bridge.rs, feature = "bridge")
    - BusBridgeConfig (remote_url, allowed_topics, auth_token, require_read_right)
    - Implements TopicHandler → direkt in Router registrierbar

J9. [x] Bus-API (CLI, REST, WebSocket)
    - `fsn bus serve` → axum REST API + WebSocket (Port 8081)
    - POST /api/bus/publish, GET /api/bus/subscriptions, POST/DELETE /api/bus/subscribe
    - GET/POST/DELETE /api/bus/standing-orders, GET /api/bus/events, GET /api/bus/ws
    - POST /api/bus/role/:role/trigger
    - `fsn bus status` / `fsn bus publish --topic X --source Y --payload J`
```

## Phase K: Browser ✅

```
K1. [x] Browser-View im Desktop (fsd-browser Crate)
    - URL-Leiste + Back/Forward/Reload Buttons
    - iframe-basiertes Rendering (Dioxus WebView = Wry)
    - Multi-Tab mit Add/Close, Tab-Titel-Anzeige

K2. [x] Download-Handler
    - DownloadEntry Model + DownloadsPanel UI
    - S3-Path /shared/downloads/ vorbereitet
    - DB-Schema in fsd-db/schemas/browser.rs (downloads-Tabelle)

K3. [x] Service-Integration
    - BrowserUrlRequest Context (Signal<Option<String>>)
    - use_context → Desktop setzt Kontext, Browser empfängt und navigiert
    - Lenses können URLs im Browser öffnen (on_open_url EventHandler)

K4. [x] Lesezeichen + History (in-memory + DB-Schema)
    - Bookmark / HistoryEntry Model
    - BookmarksPanel / HistoryPanel mit Add/Remove/Clear
    - fsd-db/schemas/browser.rs: bookmarks + history Tabellen
```

## Phase L: Lenses ✅

```
L1. [x] Lens-Datenmodell
    - Lens struct (id, name, query, items, last_refreshed, loading)
    - LensItem (role, summary, link, source)
    - LensRole enum (Wiki/Chat/Git/Map/Tasks/Iam/Other) mit icon() + label()
    - DB-Schema: fsd-db/schemas/lenses.rs (lenses + lens_items)

L2. [x] Lens-View (gruppiert nach Rolle, Summary + Link → öffnet im Browser)
    - LensesApp (fsd-lenses/src/app.rs)
    - LensDetail: grouped() → Gruppen nach Rolle
    - LensListRow: Refresh (↺) + Delete, loading-Indikator
    - on_open_url → BrowserUrlRequest Context → Browser öffnet URL
    - query::refresh_lens() → Bus-Publish (lens.query) + Demo-Fallback

L3. [x] Lens als Desktop-Icon
    - Sidebar: 🔍 Lenses → app-lenses
    - AppWindowContent: "app-lenses" → LensesApp in AppShell
    - app_id_to_label: "lenses" → "Lenses"
```

## Phase M: Search

```
M1. [ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview + Link)
M2. [ ] Service-Suche (Ebene 1)
M3. [ ] Host-Suche (Ebene 2, Bus-aggregiert)
M4. [ ] Föderale Suche (Ebene 3-4, nur mit search-Recht)
```

## Phase N: Bots

```
N1. [ ] Bot-Framework
N2. [ ] Broadcast-Bot (Telegram)
N3. [ ] Gatekeeper-Bot
```

## Phase O: Tasks

```
O1. [ ] Data Offers/Accepts
O2. [ ] Task Builder UI
O3. [ ] Task-Templates aus Store
```

## Phase P: Node (Invite, Federation)

```
P1. [ ] Invite-System (Token, verschlüsselte TOML, Port pro Einladung)
P2. [ ] Federation-Grundstruktur (Domain-Pflicht, Auth-Broker)
P3. [ ] Rechte-Kaskade (read/write/execute/search, Audit-Log)
P4. [ ] Föderaler Bus (Bridge-Konfiguration)
```

## Phase Q: Shortcuts, Menü, Profil, Polish

```
Q1. [ ] Action Registry + konfigurierbare Shortcuts
Q2. [ ] Hilfe: Auto-generierte Shortcut-Referenz
Q3. [ ] Menü: JEDER Punkt ruft echte Aktion auf
Q4. [ ] Profil: IAM + editierbar + Account-Linking + S3-Visitenkarte
Q5. [ ] Notification Bell
Q6. [ ] Context-Menüs
Q7. [ ] Animationen konfigurierbar
Q8. [ ] Alle Stubs/toten Code entfernen
```

---

## Reihenfolge

```
✅ Erledigt: A-F  (Fundament, Themes, Store, S3, Widgets, Conductor)
✅ Erledigt: G    (Ressourcen-System & Inventory)
✅ Erledigt: H    (Bridges & Rollen-APIs)
✅ Erledigt: I    (Resource Builder — Builder-UI, CLI, Analyse-Pipeline)

Prio 1:  J1-J9      Message Bus
Prio 2:  K1-K4      Browser
Prio 3:  L1-L3      Lenses
Prio 4:  M1-M4      Search
Prio 5:  N1-N3      Bots
Prio 6:  O1-O3      Tasks
Prio 7:  P1-P4      Node (Invite + Federation)
Prio 8:  Q1-Q8      Polish
Prio 4:  J1-J9      Message Bus
Prio 5:  K1-K4      Browser
Prio 6:  L1-L3      Lenses
Prio 7:  M1-M4      Search
Prio 8:  N1-N3      Bots
Prio 9:  O1-O3      Tasks
Prio 10: P1-P4      Node (Invite + Federation)
Prio 11: Q1-Q8      Polish
```
