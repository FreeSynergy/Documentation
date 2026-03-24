# TODO-Liste

[← Zurück zum Index](../INDEX.md) | [Bugs](BUGS.md)

**Regeln:**
- Jeder Punkt wird KOMPLETT umgesetzt. Keine Stubs.
- IMMER `cargo build` vor und nach jeder Änderung.
- Altes das nicht passt → LÖSCHEN und neu machen.
- Kein Menüpunkt ohne Funktion. Kein Button der nichts tut.
- Begriffe: **App** (nicht Programm), **Container-App** (nicht Container-Service), **Bundle** (nicht Gruppe), **Ressource** (alles im Store)

---

## Phase N0: matrix-sdk State Store (Blocker: fs-db Postgres-Migration)

```
N0. [ ] matrix-sdk PostgreSQL State Store implementieren
    - matrix-sdk 0.16 hat keinen offiziellen Postgres-Store (nur SQLite, Sled, In-Memory)
    - Erst wenn fs-db vollständig auf Postgres migriert ist (sea-orm postgres feature)
    - Dann: eigenen StateStore-Trait-Impl auf sea-orm/sqlx-postgres Basis
    - Bis dahin: matrix-sdk läuft mit In-Memory-Store (fs-channel matrix feature)
    - Kein sqlx-sqlite-Konflikt solange In-Memory genutzt wird
    - Bekannter Bug: matrix feature → recursion overflow in rustc ≥1.94 (matrix-sdk 0.16 upstream)
```

---

## Phase M: Search

```
M1. [ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview + Link)
M2. [ ] Service-Suche (Ebene 1)
M3. [ ] Host-Suche (Ebene 2, Bus-aggregiert)
M4. [ ] Föderale Suche (Ebene 3-4, nur mit search-Recht)
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

Prio 1:  M1-M4      Search
Prio 2:  —          Bots vollständig erledigt (N1–N14)
Prio 3:  O1-O3      Tasks
Prio 4:  P1-P4      Node (Invite + Federation)
Prio 5:  R1-R5      Mail (Stalwart-Fork, Multi-Tenant)
Prio 6:  S1-S5      Kontakte & Kalender
Prio 7:  T1-T5      Infrastruktur-Apps (Vault, Push, Video, VPN, DNS)
Prio 8:  Q1-Q8      Polish
```
