# FreeSynergy — Offene Aufgaben

[← Zurück zum Index](../INDEX.md)

---

## Regeln für jede Session

- TODO vollständig lesen zu Beginn
- Erledigte Items sofort löschen — kein [x] behalten
- Nie eigenmächtig handeln — Entscheidungen im Chat klären
- Standard-Checkliste gilt für JEDE Aufgabe (siehe unten)

---

## Standard-Checkliste

```
1. Design Pattern festlegen (Strategy / MVC / Command / Observer / …)
2. OOP: Traits + Structs zuerst → dann Impl; Traits statt match-Blöcke
3. i18n von Anfang an: jeder user-facing Text bekommt FTL-Key (KEINE Ausnahme)
4. cargo build nach jeder Änderung grün
5. Abschluss: cargo fmt --check && cargo clippy && cargo test (alle grün)
5b. Bibliotheken aktuell halten (bei jeder Arbeit am Repo):
    → cargo upgrade --incompatible (cargo-edit) oder cargo update
    → Breaking Changes prüfen + anpassen
    → Build + Tests danach wieder grün sicherstellen
    → Dependency-Update als separater Commit vor dem Feature-Commit
6. Doku: konzepte/ oder programme/ oder technik/ anlegen/aktualisieren
   → INDEX.md aktualisieren → fs-documentation committen+pushen
   → danach erst Modul-Repo committen+pushen
```

---

## Architektur (Kurzreferenz)

```
Standalone-First:    Jedes Repo funktioniert ohne den Rest. Integration via gRPC/REST/Bus.
Store-First:         Was nicht im Store ist, existiert für das System nicht.
Engine-Laufzeit:     fs-info ermittelt Capabilities → passende Render-Engine wird geladen.
Trait-First:         Traits definieren → alle Engines implementieren sie.
Activity-First:      Mensch denkt in Absichten, nicht in Programmen.

API:     gRPC (tonic) primär · REST (axum) zusätzlich · OpenAPI (utoipa) auto
DB:      immer über fs-db Repository<T> + Filter<T> · Engine kommt als Artifact
i18n:    alles über FTL-Keys · Fallback: en · keys.rs für Konstanten
Icons:   Dual-Icon (Action + Identity) · Sets austauschbar · Layout konfigurierbar
```

---

## Wartung — Gilt für ALLE Repos (wiederkehrend)

```
[ ] Bibliotheks-Updates prüfen (bei jeder Arbeit an einem Repo):
    → cargo upgrade --incompatible
    → Changelog der geänderten Crates lesen
    → Breaking Changes beheben
    → cargo build + cargo test grün
    → separater Commit: "chore: update dependencies"

[ ] Rust-Toolchain prüfen (monatlich):
    → rustup update
    → cargo clippy nach Update grün halten
```

---

---

# Offene Aufgaben — Nach Modul

---

## fs-icons — Icon-Bibliothek

> Traits + Baum + i18n für das Icon-System. Grundlage für alle anderen Module.

```
[ ] FsIcon struct: id (Pfad im Baum), label_key, group, singular/plural Variante
[ ] FsIconGroup struct: id, label_key, parent (für Baum-Navigation)
[ ] FsIconSet trait: id(), name_key(), resolve(icon_id) → Option<SvgPath>
[ ] FsIconLayout trait: render(action, identity, config) → LayoutedIcon
[ ] FsIconLayoutConfig: Modus (Default/SideBySide/Overlay/Custom) + Custom-Parameter
[ ] Icon-Baum als Rust-Enums oder Konstanten (places, people, things, actions, …)
    alle 20 Gruppen aus konzepte/icon-baum.md abbilden
[ ] Design Pattern: Strategy (Layout) + Composite (Dual-Icon) + Registry (Sets)
[ ] cargo build + clippy + tests grün
[ ] Doku: programme/fs-icons.md erweitern (Traits, Layout-System)
[ ] fs-icons committen + pushen
```

## fs-icon-sets — Native SVG-Sets

> Repo für austauschbare Icon-Sets. Heute: homarrlabs + we10x (extern).
> Neu: natives FreeSynergy-Set mit allen Icons aus dem Baum.

```
[ ] FreeSynergy-Set anlegen: freesynergy/ Verzeichnis in fs-icons/
    → SVG für jedes Icon im Baum (konzepte/icon-baum.md als Vorlage)
    → Namensschema: {gruppe}-{name}.svg (z.B. places-home.svg, people-person.svg)
    → Dark-Varianten: {name}-dark.svg
[ ] manifest.toml: FreeSynergy-Set als builtin = true eintragen
[ ] Store-Eintrag: Store/ catalog.toml — icon-set als artifact-Typ
[ ] Doku: programme/icons/icon-set-erstellen.md aktualisieren
[ ] fs-icons committen + pushen
```

## fs-i18n — Icon-Übersetzungen

> Jedes Icon bekommt FTL-Keys (Name + Beschreibung) in allen Sprachen.

```
[ ] icons.ftl in locales/de/ anlegen:
    → icon-{name} = Anzeigename
    → icon-{name}-description = Wann/wofür dieses Icon eingesetzt wird
    → alle Icons aus icon-baum.md (20 Gruppen)
[ ] icons.ftl in locales/en/ anlegen (Basis-Übersetzung)
[ ] keys.rs in fs-i18n: Konstanten für alle Icon-Keys
[ ] fs-i18n committen + pushen
```

## fs-desktop — Icon-Settings

> Settings-Seite für Icon-Konfiguration im Desktop.

```
[ ] Icon-Settings-Seite in fs-settings:
    → Aktives Set auswählen (Liste aller installierten Sets)
    → Layout-Modus wählen (Default / Side-by-Side / Overlay / Custom)
    → Custom-Layout: Editor für Position, Größe, Z-Ebene je Icon
    → Vorschau: Live-Rendering mit gewähltem Set + Layout
[ ] Design Pattern: MVC (Settings) + Strategy (Layout-Modi)
[ ] i18n: icons.ftl (de+en) in fs-i18n — Settings-Keys ergänzen
[ ] fs-desktop committen + pushen

[ ] Accounts-Settings: IAM via Kanidm (OIDC-Login-Flow)
[ ] Shortcuts-Settings: Tastaturkürzel-Editor
```

---

## fs-standards — Canonical Data Model

> Neues Repo. CDM als Superset aus SCIM, vCard, schema.org, OIDC.
> [Doku: technik/standards.md](../technik/standards.md)

```
[ ] GitHub Repo anlegen: FreeSynergy/fs-standards
    lokal: /home/kal/Server/fs-standards/
[ ] Rust-Crate: fs-standards (library, kein binary)
[ ] StandardAdapter<External>-Trait:
      fn standard_id() -> &'static str   (z.B. "RFC 7643")
      fn to_external(&FsPerson) -> External
      fn from_external(&External) -> Result<FsPerson, FsError>
[ ] ProvisioningAdapter-Trait:
      fn service_id() -> &'static str
      async fn provision(&FsPerson, &DataProfile) -> Result<()>
      async fn deprovision(&FsId) -> Result<()>
      async fn sync(&FsPerson, &DataProfile) -> Result<SyncDiff>
[ ] DataProfile + DataProfileEntry + FieldConsent Structs
[ ] spec/ Verzeichnis: Doku aus technik/standards.md übernehmen
[ ] CONTRIBUTING.md: wie externe Entwickler mitmachen
[ ] cargo build + clippy + tests grün
[ ] fs-standards committen + pushen
```

## fs-types — Kanonische Typen

> Abhängigkeit: fs-standards (Traits müssen definiert sein)

```
[ ] FsPerson struct (SCIM + vCard + schema.org + OIDC Superset)
      FsName, FsEmail, FsPhone, FsAddress, FsPersonExtension
[ ] FsGroup struct (SCIM Groups + vCard KIND:group)
      FsGroupMember, FsGroupExtension, DataProfile, DataProfileEntry
[ ] FsOrg struct (schema.org/Organization + vCard + LDAP)
      FsOrgExtension
[ ] FsPersonField<T>: value + visibility + consent + encrypted
      Visibility: Public | Group(FsId) | Private | Owner
      ServiceConsent: service_id + allowed
[ ] serde (Serialize/Deserialize), Clone, Debug für alle Typen
[ ] cargo build + clippy + tests grün
[ ] fs-types committen + pushen
```

---

## fs-auth — SCIM + LDAP

> Abhängigkeit: fs-standards + fs-types

```
[ ] ScimProvider-Trait vollständig implementieren:
      GET /scim/v2/Users · GET /scim/v2/Users/{id}
      POST /scim/v2/Users · PUT · PATCH · DELETE
      GET /scim/v2/Groups · analog
      Filter: ?filter=userName eq "bjensen"
[ ] FS-Extension im SCIM-Response: urn:freeSynergy:schemas:1.0:FsPerson
[ ] SCIM-Token-Auth: Bearer Token via Kanidm
[ ] OpenAPI: utoipa Spec für SCIM Endpoints
[ ] ScimUserAdapter implements StandardAdapter<ScimUser>
[ ] ScimGroupAdapter implements StandardAdapter<ScimGroup>
[ ] LdapAdapter implements StandardAdapter<LdapEntry>
[ ] fs-auth committen + pushen
```

---

## fs-managers — Manager-Integration + Provisioning

```
Manager-Aktionen:
[ ] UpdatePodConfig: ManagerAction::UpdatePodConfig → PodConfigurator::apply()
    → podman play kube --replace <generated.yml>
[ ] EditConfig: ManagerAction::EditConfig → AppConfigurator::apply()
[ ] Role-Switching: fs-registry Capability-Eintrag umschreiben wenn set_active()
[ ] Update-Check: update_available() mit echtem Store-Lookup implementieren

Forgejo:
[ ] Store-Eintrag: forgejo catalog.toml + pod.yml
[ ] S3: Repository-Storage via opendal
[ ] ForgejoAdapter → fs-registry wiring (Service Role: git)

Provisioning (Abhängigkeit: fs-standards + fs-types):
[ ] ProvisioningCoordinator:
      on UserJoinedGroup → DataProfile holen → ProvisionUser Events senden
      on UserLeftGroup   → DeprovisionUser Events senden
[ ] KanidmProvisioningAdapter implements ProvisioningAdapter
[ ] OutlineProvisioningAdapter implements ProvisioningAdapter
[ ] MatrixProvisioningAdapter implements ProvisioningAdapter
[ ] DataProfile UI: Gruppe bearbeiten → DataProfile-Tab
      Checkboxen: welche Services aktiv
      Feld-Auswahl: welche Felder gehen zu welchem Service

Bus-Topics (in konzepte/bus-api-namespaces.md ergänzen):
[ ] fs.identity.user.joined_group  { user_id, group_id }
[ ] fs.identity.user.left_group    { user_id, group_id }
[ ] fs.identity.person.updated     { person_id, changed_fields }
[ ] fs.identity.provision.request  { user_id, service_id, profile }

[ ] fs-managers committen + pushen
```

---

## fs-standards — Adapter-Implementierungen

> Abhängigkeit: fs-standards Repo + fs-types

```
[ ] VCardAdapter implements StandardAdapter<VCard>
      → FsPerson ↔ vCard RFC 6350
[ ] SchemaOrgPersonAdapter implements StandardAdapter<JsonLdPerson>
      → FsPerson ↔ schema.org/Person JSON-LD
[ ] fs-standards committen + pushen
```

## fs-federation — ActivityPub-Adapter

> Abhängigkeit: fs-standards + fs-types

```
[ ] ActivityPubPersonAdapter implements StandardAdapter<ApActor>
      → FsPerson ↔ ActivityPub Actor
[ ] Echte ActivityPub-Impl (G1+): über G-S.3 Adapter
[ ] WireGuard (nach AP-Grundstruktur)
[ ] Hickory DNS (nach AP-Grundstruktur)
[ ] fs-federation committen + pushen
```

---

## fs-lenses — Suche erweitern

```
[ ] Host-Suche: Service-seitige search::query Handler (in Manager-Repos)
[ ] Föderale Suche (nach fs-federation stabil)
[ ] fs-lenses committen + pushen
```

---

## Integration-Tests — Alle Container

> Laufen erst wenn jeweiliger Dienst via Store deployt ist.

```
[ ] Kanidm-Integration-Test (env vars setzen)
[ ] Zentinel-Integration-Test (env vars setzen)
[ ] Stalwart-Integration-Test (env vars setzen)
[ ] Tuwunel-Integration-Test (env vars setzen)
[ ] Telegram-Test (echtes Bot-Token)
[ ] gRPC subscribe Telegram: Streaming-Impl (aktuell REST long-poll)
```

---

## Technisches Debt — Wartet auf Upstream

```
[ ] matrix-sdk: rustc ≥ 1.94 Fix (matrix-sdk 0.16) — blockiert live Feature
[ ] matrix-sdk: PostgreSQL State Store (recursion-overflow Fix upstream)
[ ] Zentinel: S3-Integration via opendal für Konfigurations-Storage
[ ] JMAP-Login-Flow: in fs-settings konfigurierbar machen
```

---

---

# Roadmap

```
── Jetzt ────────────────────────────────────────────────────────
  fs-icons:      Icon-Bibliothek (Traits + Baum)
  fs-icon-sets:  Native FreeSynergy SVG-Set
  fs-i18n:       icons.ftl (alle Gruppen, de+en)
  fs-desktop:    Icon-Settings-Seite

── Parallel startbar ────────────────────────────────────────────
  fs-standards:  Repo + Traits + Adapters
  fs-types:      FsPerson, FsGroup, FsOrg

── Danach ───────────────────────────────────────────────────────
  fs-auth:       SCIM Provider + LdapAdapter
  fs-managers:   Provisioning + Manager-Aktionen + Forgejo

── G2 ───────────────────────────────────────────────────────────
  fs-gui-engine-iced:  libcosmic vollständige Integration
  fs-gui-engine-tui:   Navigation-Traits (Corner/Side in TUI)
  fs-gui-engine-bevy:  Navigation-Traits (3D-fähig)
  fs-render:           Dioxus komplett raus
  ProgramView::Binding — Workflow-Editor

── Langfristig ──────────────────────────────────────────────────
  fs-federation:       Echte ActivityPub-Impl
  IETF Draft:          urn:freeSynergy:schemas:1.0
```

---

## Archiviert

```
fs-apps:     archiviert 2026-04-01 (alle Apps in eigene Repos migriert)
fs-builder:  archiviert 2026-04-02 (Pipeline-Pattern in fs-managers)
CHANGELOG.md: nicht mehr gepflegt (seit 2026-03)
```
