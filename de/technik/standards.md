# FreeSynergy — Daten-Standards Strategie

[← Zurück zum Index](../INDEX.md)

---

## Grundprinzip: Superset der vorhandenen Standards

FreeSynergy erfindet kein eigenes Datenformat.
Stattdessen gilt: **Was ein RFC/ISO/W3C-Standard definiert, übernehmen wir 1:1.**
Was noch kein Standard hat, kommt in den `urn:freeSynergy:schemas:1.0` Namespace.

Das Modell ist ODF: XML + SVG + MathML kombiniert → kein Neuerfinden, sondern gezieltes Zusammenführen.

```
SCIM (RFC 7643)  +  vCard (RFC 6350)  +  schema.org
         ↘                  ↓                ↙
                       FsPerson
                  (Superset + FS-Extensions)
                         ↙    ↘
              FsGroup         FsOrg
```

---

## Warum Standards?

1. **Interoperabilität**: Externe Systeme sprechen SCIM, vCard, LDAP — unsere Adapter sind trivial.
2. **Daten sammeln, bevor das Programm da ist**: Standards-konforme Daten können später von
   neuen Programmen sofort genutzt werden.
3. **Community**: Programmierer müssen keine proprietären Formate lernen.
4. **Ziel — Standardisierung**: Wenn FS-Extensions sich etablieren → IETF Draft einreichen
   (wie SCIM: draft-ietf → RFC 7643/7644).

---

## Kanonische Typen (Canonical Data Model)

Die kanonischen Typen leben in `fs-types`. Sie sind das einzige interne Format.
Kein Programm spricht SCIM direkt — es spricht `FsPerson`. Adapter übersetzen.

| Typ | Basis-Standards | FS-Extension |
|---|---|---|
| `FsPerson` | SCIM RFC 7643, vCard RFC 6350, schema.org/Person, OIDC | `FsPersonExtension` |
| `FsGroup` | SCIM RFC 7643 Groups, vCard KIND:group | `FsGroupExtension` |
| `FsOrg` | schema.org/Organization, LDAP organizationalUnit | `FsOrgExtension` |

---

## Regel: Welcher Name wird kanonisch?

> Der am weitesten verbreitete RFC/ISO-Name wird kanonisch. Nie erfinden.

Beispiel "Vorname":

| Standard | Feldname |
|---|---|
| SCIM RFC 7643 | `name.givenName` |
| schema.org | `givenName` |
| OpenID Connect | `given_name` |
| LDAP | `givenName` |
| vCard RFC 6350 | `N` component `GIVEN` |

**Kanonisch:** `given_name` (OIDC + snake_case Rust-Konvention)

---

## 3 Fälle beim Field-Mapping

| Fall | Lösung |
|---|---|
| Gleiche Semantik, anderer Name | Kanonischer Name = verbreitetster Standard |
| Gleiche Semantik, anderes Modell (flat vs. nested) | Reichhaltigstes Modell kanonisch, Adapter vereinfacht |
| Nur FS hat das Feld | `urn:freeSynergy:schemas:1.0` Namespace |

Verlust beim Roundtrip (z.B. LDAP kennt nur eine E-Mail, wir kennen mehrere) ist
**dokumentiert und akzeptiert** — Adapater-Seite vereinfacht beim Export.

---

## FS-Namespace

SCIM unterstützt Enterprise Extensions via eigene Schema-URN. FS nutzt dieses Muster:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User",
    "urn:freeSynergy:schemas:1.0:FsPerson"
  ],
  "userName": "kai",
  "emails": [{"value": "kai@example.com", "type": "work"}],

  "urn:freeSynergy:schemas:1.0:FsPerson": {
    "dataProfiles": ["iam", "wiki", "chat"],
    "fsCapabilities": ["federation", "sync"]
  }
}
```

Basis-Felder (SCIM/vCard) bleiben stabil und extern kompatibel.
FS-Extensions sind dynamisch erweiterbar — neues Feature = neues Feld im Extension-Struct.

---

## StandardAdapter-Trait

```rust
pub trait StandardAdapter<External> {
    /// z.B. "RFC 6350", "RFC 7643", "schema.org/Person"
    fn standard_id(&self) -> &'static str;
    fn to_external(&self, person: &FsPerson) -> External;
    fn from_external(&self, ext: &External) -> Result<FsPerson, FsError>;
}
```

Jeder Standard bekommt genau einen Adapter. Nie N×M Konverter.

---

## Feldweiser Datenschutz (ABAC)

```rust
pub struct FsPersonField<T> {
    pub value: T,
    pub visibility: Visibility,       // Public | Group(id) | Private | Owner
    pub consent: Vec<ServiceConsent>, // "sync to iam: yes", "wiki: no"
    pub encrypted: bool,
}
```

Nicht nur "darf ich diese Person sehen?" sondern "welche Felder darf welcher Service lesen/schreiben?".

---

## Group-driven Provisioning

Gruppen tragen ein `DataProfile` — sie definieren, welche Felder an welche Services gehen:

```
Gruppe "Mitarbeiter":
  ✅ IAM (Kanidm)   → given_name, family_name, email[work], role
  ✅ Wiki (Outline)  → given_name, avatar
  ✅ Chat (Matrix)   → given_name, avatar, email[work]
  ☐  Kalender        → nicht aktiviert

User tritt "Mitarbeiter" bei:
  fs-bus Event: UserJoinedGroup
    → KanidmProvisioningAdapter.provision(user, profile)
    → OutlineProvisioningAdapter.provision(user, profile)
    → MatrixProvisioningAdapter.provision(user, profile)
```

Das folgt dem **Provisioning-Pipeline-Pattern** (Observer via Bus):

```rust
pub trait ProvisioningAdapter {
    fn service_id(&self) -> &'static str;
    async fn provision(&self, person: &FsPerson, profile: &DataProfile) -> Result<()>;
    async fn deprovision(&self, person_id: &FsId) -> Result<()>;
    async fn sync(&self, person: &FsPerson, profile: &DataProfile) -> Result<SyncDiff>;
}
```

---

## Repos

| Repo | Inhalt |
|---|---|
| `fs-types` | Kanonische Typen: FsPerson, FsGroup, FsOrg + Extensions |
| `fs-standards` (neu) | Adapter-Traits + Standard-Mapping-Spec (kein Code außer Traits) |
| `fs-auth` | SCIM Provider Trait (schon geplant) — nutzt FsPerson |
| `fs-managers` | ProvisioningAdapter Impls per Service |
| `fs-bus` | Events: UserJoinedGroup, UserLeftGroup, PersonUpdated |
| `fs-federation` | ActivityPub Adapter (nutzt FsPerson) |

---

## Standardisierungs-Roadmap

```
Jetzt:    urn:freeSynergy:schemas:1.0 — dokumentiert, öffentlich
→ wenn stabil: Community nutzt es, Implementierungen entstehen
→ Ziel:   IETF Draft einreichen (wie SCIM: draft-ietf → RFC)
```

---

## Weiterführend

- [FsPerson Field Mappings](standards-person.md)
- [FsGroup Field Mappings](standards-group.md)
- [FsOrg Field Mappings](standards-org.md)
- [FS Namespace Registry](standards-namespaces.md)
