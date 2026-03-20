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

Siehe Git-History. Fundament, Themes, Store, S3, Widgets, Container App Manager — alles implementiert.

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
N1. [x] fsn-channel Crate (FreeSynergy.Lib)
    - Channel-Trait: create_room, invite, kick, send, delete_room, get_members
    - Inventory-Integration: services_with_role("chat") → aktive Channels
    - ChannelRegistry::build(kind, config) + active_channels() via Inventory
    - Adapter (alle hinter Channel-Trait):
        TelegramChannel      (feature = "telegram", Telegram Bot HTTP API)
        MatrixChannel        (feature = "matrix", Client-Server API via reqwest)
        DiscordChannel       (feature = "discord", Discord REST API v10)
        RocketChatChannel    (REST, immer kompiliert)
        MattermostChannel    (REST, immer kompiliert)
        XmppChannel          (feature = "xmpp", REST Bridge)
        ZulipChannel         (REST, immer kompiliert)
        RevoltChannel        (REST, immer kompiliert)
        NextcloudTalkChannel (REST, immer kompiliert)
        IrcChannel           (feature = "irc", REST Bridge)
        SlackChannel         (feature = "slack", Slack Web API)
        TeamsChannel         (REST, Graph API, immer kompiliert)
        ViberChannel         (REST, Viber Bot API)
        LineChannel          (REST, LINE Messaging API)
        WhatsAppChannel      (REST, Meta Cloud API)
        SignalChannel        (feature = "signal", signal-cli REST API)
        ThreemaChannel       (REST, Threema Gateway)
        WireChannel          (REST, Wire Bot API)
        DiscourseChannel     (REST, Discourse API)
        LemmyChannel         (REST, Lemmy API v3)
        MastodonChannel      (REST, Mastodon API v1)

N2. [x] Messenger-Adapter als Store-Pakete (type = "messenger-adapter")
    - MessengerAdapterResource in fsn-types (ResourceType::MessengerAdapter)
    - MessengerKind (21 Plattformen), AdapterAuthMethod, ChannelFeature enums
    - Validate: erfordert tokens_required + ChannelFeature::Send
    - Inventory: services_with_role("chat") liefert aktive Adapter
    - Store-Verzeichnis: shared/messenger-adapters/

N3. [ ] BotCommand-Trait (FreeSynergy.Lib)
    - Einheitliche Schnittstelle für alle Bot-Module und Bot-Typen:
        fn name() → &str
        fn description() → &str
        fn required_right() → Right
        fn execute(ctx: CommandContext) → BotResponse
    - Bot sammelt Befehle aus allen aktiven Modulen automatisch
    - /help generiert sich selbst aus registrierten Befehlen

N4. [ ] Bot-Kern (FreeSynergy.Managers, bots/)
    - Jede Bot-Instanz läuft als eigener Prozess (eigenes Binary)
    - Runtime: startet, hält Verbindungen zu allen konfigurierten Messengern
    - Module-Loader: lädt bot-Pakete aus Inventory, initialisiert Module
    - Command-Dispatcher: /command → passendes Modul, Rechte-Check (FSN, nicht Messenger)
    - Trigger-Engine: Bus-Events → Modul-Handler aufrufen
    - Bot-Registry in fsn-botmanager.db
    - Bus-Client: publiziert Events, empfängt Events (bot.*, chat.*, calendar.*)
    - Secrets: Token-Referenzen aus Secrets-Store, nie Klartext
    - Steuerung drei Ebenen:
        FSN-Ebene:        private / group / public
        Messenger-Ebene:  everyone / admins / nobody
        Control-Ebene:    optional — nur Control Bot steuert
    - Rechte pro Funktion pro Gruppe konfigurierbar
    - Audit-Log: jede Aktion (User/System, Aktion, Ziel, Ergebnis, UTC-Timestamp)
    - Mitglieder-Interaktion: Menüs / Inline-Tastaturen / DMs je nach Messenger

N5. [ ] Control Bot (bot-control Paket im Store)
    - Kann andere Bot-Instanzen erstellen
    - Vererbung: gibt Messenger-Accounts + Gruppen + Einstellungen an Kind-Bots weiter
    - Kind-Bots haben keine eigenen Tokens — kommunizieren über Control Bot
    - Kind-Bots arbeiten autonom weiter wenn Control Bot offline (Bus-Events puffern)
    - Control Bot kann auf Logs aller Kind-Bots zugreifen

N6. [ ] Broadcast-Modul (bot-broadcast Paket im Store)
    - /subscribe <topic> Command → Subscription in Bus registrieren
    - /unsubscribe <topic> Command
    - /subscriptions Command → Liste aktiver Subscriptions
    - Bus-Listener: empfängt subscribte Events → sendet in Gruppe
    - Subscription-Storage: SQLite (group_id, topic, messenger)
    - Standalone-fähig (kein Control Bot nötig)

N7. [ ] Gatekeeper-Modul (bot-gatekeeper Paket im Store)
    - Trigger: chat.join_request → publiziert Event an BotManager
    - /verify <user_id> Command → IAM-Check via Bus (Rolle: iam)
    - /approve <request_id> + /deny <request_id> Commands
    - Join-Request-Queue: SQLite (request_id, user, group, status, iam_result)
    - IAM-Integration: via Bus-Event (nicht direkt) → Bridge → Kanidm
    - required_roles = [{ roles = ["iam"], mode = "ANY" }]
    - Standalone-fähig

N8. [ ] Calendar-Modul (bot-calendar Paket im Store)
    - Bus-Listener: calendar.event.upcoming → postet Erinnerung in Gruppe
    - /termine Command → zeigt kommende Termine aus Desktop-Calendar
    - DM-Erinnerungen an Teilnehmer (wenn Rechte vorhanden)
    - required_roles = [{ roles = ["tasks"], mode = "ANY" }]
    - Standalone-fähig

N9. [ ] Gruppen-Verwaltung + Filter (im Bot-Programm und BotManager)
    - Gruppen-Import beim Verbinden eines Messenger-Accounts (bis 500 Gruppen)
    - Filter: Größe / Aktivität / Name / Messenger / aktive Module / Bot-Status
    - Manuelle Collections (keine Automatik)
    - Eine Gruppe kann in mehreren Collections sein
    - Bulk-Aktionen auf gefilterte Gruppen oder Collections

N10. [ ] Room-Sync-Modul (bot-room-sync Paket im Store)
    - Räume/Gruppen synchronisieren (gleicher Messenger: vollständig)
    - Nachrichten-Sync cross-Messenger (via fsn-channel)
    - Mitglieder-Sync cross-Messenger nur wenn User in IAM mit allen Konten
    - Standalone-fähig

N11. [ ] BotManager-Programm (FreeSynergy.Managers, bots/)
    - Dashboard: alle Bot-Instanzen mit Status
    - Bot-Instanzen erstellen (Meta → Bot-Typ → Messenger → Steuerung)
    - Fertige Instanz erscheint als eigenes Programm im Desktop (Icon, Name, Widget)
    - Broadcast-View, Subscriptions-View, Gatekeeper-View, Module-View
    - Log-View aggregiert (alle Bots)
    - CLI: fsn bot status / broadcast / gatekeeper list / gatekeeper approve <id>
    - Bus-Client: publiziert bot.*, empfängt chat.join_request + bot.status.response

N12. [ ] Desktop-Integration
    - BotManager als eingebettete App im Desktop (app-botmanager)
    - Jede Bot-Instanz als eigenes Icon im Launcher (app-bot-<name>)
    - Optionales Widget je Instanz (Status, letzte Aktivität, Schnellaktionen)
    - Sidebar-Tab 🤖 Bots → öffnet BotManager

N13. [ ] Store-Integration (bot- und adapter-Pakete)
    - BotResource + MessengerAdapterResource in fsn-types
    - Store-Verzeichnis: shared/bots/ + shared/messenger-adapters/
    - Store-UI: Bot-Pakete + Adapter-Pakete browsebar (type-Filter)
    - required_roles → Modul inaktiv wenn Rolle fehlt, auto-aktiv wenn Rolle verfügbar
    - bundle-bots-all: installiert alle bot-* Pakete

N14. [ ] Updates (manuell, Automatismus folgt später)
    - Updates über Store manuell einspielen (wie alle anderen Pakete)
    - TODO (spätere Phase): Update-Benachrichtigung, One-Click-Update, Rollback,
      Versions-Kompatibilitäts-Check für laufende Instanzen
```

## Hinweis: Kanidm + Tuwunel als native Apps

Kanidm (IAM) und Tuwunel (Matrix-Server) sind Rust-Projekte und werden **nicht** als Container-Apps
installiert, sondern als native **FSN-Apps** (fork → kompilieren → als App-Paket verteilen).
Für beide existieren bereits Upstream-Repos — wir forken, bauen eigene FSN-App-Pakete, und halten
die Forks per GitHub Actions automatisch mit Upstream synchron.

---

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

## Phase R: Mail-Server (Stalwart, kein Fork)

```
Architektur-Entscheidung:
  Kein Fork. Stalwart wird unverändert als FSN-App-Paket installiert.
  "Multi-Tenancy" entsteht durch die FSN-Federation: jeder Node hat seine eigene
  Stalwart-Instanz mit eigener Domain. Das ist dasselbe Prinzip wie bei E-Mail
  und Matrix — dezentral statt zentral multi-tenant.
  → Keine Lizenzfragen, keine Fork-Divergenz, automatische Updates über den Store.

R1. [ ] Stalwart als FSN-App-Paket
    - Stalwart (freie Version, AGPL) unverändert verwenden
    - AppResource mit Rollen: smtp, imap, jmap
    - Installation via FSN Store auf jedem Node

R2. [ ] IAM-Integration (Kanidm via OIDC/LDAP)
    - Nutzer aus Kanidm automatisch als Mailbox-Nutzer
    - Stalwart unterstützt OIDC/LDAP in der freien Version
    - SSO: Login via Kanidm, kein separates Mail-Passwort

R3. [ ] Bridge implementieren
    - smtp_bridge(), imap_bridge() in fsn-bridge/catalog.rs
    - Rollen: smtp (senden), imap (empfangen/lesen)

R4. [ ] Domain-Konfiguration pro Node
    - Jeder Node konfiguriert seine eigene Mail-Domain
    - Standard-Setup via Node-Wizard (Phase P)
```

## Phase S: Kontakte & Kalender

```
Entscheidung: Rustical als Basis (CalDAV + CardDAV in Rust, open source)
Alternativ: eigene Implementierung auf Basis von `icalendar` + `vcard4` Crates

S1. [ ] Rustical evaluieren (CalDAV + CardDAV)
    - Lizenz prüfen
    - Wie Stalwart: pro Node eine Instanz, kein zentrales Multi-Tenant nötig
    - Entscheidung: unverändert nutzen oder eigene Implementierung

S2. [ ] Kontakte-Backend
    - vCard 4.0 Speicherung (SQLite + S3 für Avatare)
    - CardDAV-Protokoll (sync mit Clients: DAVx⁵, Thunderbird, etc.)
    - Gruppen/Org-weite Kontaktbücher

S3. [ ] Kalender-Backend
    - iCal/CalDAV (sync mit Clients)
    - `icalendar` Crate für Parsing
    - Wiederkehrende Termine, Einladungen (iTIP/iMIP)
    - Bus-Integration: calendar.event.upcoming → Bot-Module (N8)

S4. [ ] IAM-Integration
    - Nutzer aus Kanidm → automatisch Kalender + Adressbuch
    - Org-weite geteilte Kalender/Kontaktbücher pro Tenant

S5. [ ] FSN-App-Pakete
    - contacts-server AppResource (Rolle: contacts)
    - calendar-server AppResource (Rolle: calendar)
    - Bridges: contacts_bridge(), calendar_bridge() in fsn-bridge/catalog.rs
```

## Phase T: Infrastruktur-Apps

```
T1. [ ] Vaultwarden (Passwort-Manager)
    - Bitwarden-kompatibler Server in Rust (AGPL)
    - Als FSN-App-Paket
    - Rolle: secrets
    - SSO via Kanidm (OIDC)
    - Multi-Tenant: Org-Vaults getrennt

T2. [ ] UnifiedPush / Ntfy (Push-Benachrichtigungen)
    - Ntfy: push notification relay in Go (Apache 2.0)
    - Alternativ: eigener UnifiedPush-Distributor in Rust
    - Für Mobile-Apps: Benachrichtigungen ohne Google/Apple
    - Matrix/Tuwunel-Integration (Mobile-Clients)
    - Rolle: push

T3. [ ] Element Call / TURN-Server (Video-Calls)
    - Element Call läuft auf Tuwunel-Matrix-Server
    - coturn als TURN-Server (für NAT-Traversal)
    - Beide als FSN-App-Pakete
    - Keine separate Infrastruktur wenn Tuwunel läuft

T4. [ ] WireGuard (Node-zu-Node VPN)
    - Nur relevant wenn Nodes über fremde Netzwerke kommunizieren
    - `wireguard-control` Crate
    - Automatische Peer-Discovery via Node-Registry
    - Optional — erst wenn Federation (Phase P) implementiert

T5. [ ] Hickory DNS (Internes DNS / Service Discovery)
    - Erst nach Federation relevant
    - Interne Service-Discovery für Node-Netzwerk
    - Autoritativer DNS für *.node.local-Domains
    - Öffentliches DNS: weiterhin extern (zu risikoreich selbst zu hosten)
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
✅ Erledigt: A-F  (Fundament, Themes, Store, S3, Widgets, Container App Manager)
✅ Erledigt: G    (Ressourcen-System & Inventory)
✅ Erledigt: H    (Bridges & Rollen-APIs)
✅ Erledigt: I    (Resource Builder — Builder-UI, CLI, Analyse-Pipeline)
✅ Erledigt: J    (Message Bus)
✅ Erledigt: K    (Browser)
✅ Erledigt: L    (Lenses)
✅ Erledigt: N1   (fsn-channel Crate — Channel-Trait + 21 Adapter)
✅ Erledigt: N2   (MessengerAdapterResource — Store-Paket-Typ)

Prio 1:  M1-M4      Search
Prio 2:  N3-N14     Bots (BotCommand-Trait, Bot-Kern, Module, BotManager, …)
Prio 3:  O1-O3      Tasks
Prio 4:  P1-P4      Node (Invite + Federation)
Prio 5:  R1-R5      Mail (Stalwart-Fork, Multi-Tenant)
Prio 6:  S1-S5      Kontakte & Kalender
Prio 7:  T1-T5      Infrastruktur-Apps (Vault, Push, Video, VPN, DNS)
Prio 8:  Q1-Q8      Polish
```
