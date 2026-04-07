# FsPerson — Field Mappings

[← Standards Übersicht](standards.md) | [← Index](../INDEX.md)

---

## Kanonische Felder

### Identität

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 | schema.org | OIDC | LDAP |
|---|---|---|---|---|---|---|
| `id` | `FsId` | `id` (UUID) | `UID` | `@id` | `sub` | `entryUUID` |
| `user_name` | `String` | `userName` | `NICKNAME` | — | `preferred_username` | `uid` |
| `external_id` | `Option<String>` | `externalId` | — | — | — | `employeeNumber` |

### Name

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 | schema.org | OIDC | LDAP |
|---|---|---|---|---|---|---|
| `name.given_name` | `String` | `name.givenName` | `N;GIVEN` | `givenName` | `given_name` | `givenName` |
| `name.family_name` | `String` | `name.familyName` | `N;FAMILY` | `familyName` | `family_name` | `sn` |
| `name.middle_name` | `Option<String>` | `name.middleName` | `N;ADDITIONAL` | `additionalName` | `middle_name` | — |
| `name.honorific_prefix` | `Option<String>` | `name.honorificPrefix` | `N;PREFIX` | `honorificPrefix` | — | `title` |
| `name.honorific_suffix` | `Option<String>` | `name.honorificSuffix` | `N;SUFFIX` | `honorificSuffix` | — | — |
| `name.formatted` | `String` | `name.formatted` | `FN` | `name` | `name` | `cn` |
| `display_name` | `String` | `displayName` | `FN` | `name` | `name` | `displayName` |

### Kontakt

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 | schema.org | OIDC | LDAP |
|---|---|---|---|---|---|---|
| `emails` | `Vec<FsEmail>` | `emails[{value,type,primary}]` | `EMAIL;TYPE=...` | `email` | `email` | `mail` |
| `phones` | `Vec<FsPhone>` | `phoneNumbers[{value,type}]` | `TEL;TYPE=...` | `telephone` | `phone_number` | `telephoneNumber` |
| `addresses` | `Vec<FsAddress>` | `addresses[{...}]` | `ADR` | `address` | `address` | `postalAddress` |
| `urls` | `Vec<FsUrl>` | — | `URL` | `url` | `website` | `labeledURI` |

#### FsEmail

| FS | SCIM | vCard | Notiz |
|---|---|---|---|
| `value` | `value` | `EMAIL` value | |
| `email_type` | `type` (work/home/other) | `TYPE=WORK/HOME` | |
| `primary` | `primary` (bool) | `PREF=1` | vCard nutzt Priorität 1-100 |

#### FsAddress

| FS | SCIM | vCard | LDAP |
|---|---|---|---|
| `street_address` | `streetAddress` | `ADR;STREET` | `street` |
| `locality` | `locality` | `ADR;LOCALITY` | `l` |
| `region` | `region` | `ADR;REGION` | `st` |
| `postal_code` | `postalCode` | `ADR;PCODE` | `postalCode` |
| `country` | `country` | `ADR;CTRY` | `c` (ISO 3166-1) |
| `formatted` | `formatted` | `ADR` formatted | — |

### Organisation & Gruppen

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 | schema.org | LDAP |
|---|---|---|---|---|---|
| `org_ref` | `Option<FsOrgRef>` | `enterpriseUser.organization` | `ORG` (first component) | `worksFor` | `o` |
| `department` | `Option<String>` | `enterpriseUser.department` | `ORG` (second component) | `department` | `ou` |
| `job_title` | `Option<String>` | `title` | `TITLE` | `jobTitle` | `title` |
| `manager` | `Option<FsPersonRef>` | `enterpriseUser.manager` | — | — | `manager` |
| `groups` | `Vec<FsGroupRef>` | `groups[{value,display}]` | — | `memberOf` | `memberOf` |

### Status & Meta

| FS (kanonisch) | Typ | SCIM RFC 7643 | Notiz |
|---|---|---|---|
| `active` | `bool` | `active` | |
| `locale` | `Option<LanguageCode>` | `locale` | BCP 47 |
| `timezone` | `Option<String>` | `timezone` | IANA tz |
| `avatar_url` | `Option<FsUrl>` | `photos[type=photo]` | vCard: `PHOTO` |
| `created_at` | `DateTime<Utc>` | `meta.created` | |
| `updated_at` | `DateTime<Utc>` | `meta.lastModified` | |

---

## FS-Extension (`FsPersonExtension`)

Felder ohne existierenden Standard — `urn:freeSynergy:schemas:1.0:FsPerson`:

| Feld | Typ | Bedeutung |
|---|---|---|
| `data_profiles` | `Vec<DataProfileRef>` | Welche Sync-Profile aktiv sind |
| `fs_capabilities` | `Vec<FsCapability>` | z.B. `federation`, `sync`, `ai-chat` |
| `node_origin` | `Option<FsNodeRef>` | Von welchem Node importiert |
| `federation_id` | `Option<FsUrl>` | ActivityPub Actor-URL |
| `consent_log` | `Vec<ConsentEntry>` | Wer hat wann auf welche Felder zugegriffen |

---

## Roundtrip-Verluste (dokumentiert)

| Richtung | Verlust |
|---|---|
| `FsPerson` → LDAP | Nur erste E-Mail (LDAP: `mail` ist einwertig) |
| `FsPerson` → LDAP | Alle `fs:`-Extensions gehen verloren |
| vCard → `FsPerson` | `ORG` zweite Komponente = `department` (heuristisch) |
| OIDC → `FsPerson` | Kein `phones`, kein `addresses` (OIDC-Claims nicht standardisiert) |

Verluste beim Export sind **immer dokumentiert, nie still ignoriert**.

---

## Verschlüsselung sensibler Felder

Felder können per `FsPersonField<T>` einzeln verschlüsselt und Zugriffs-kontrolliert sein:

```rust
pub struct FsPersonField<T> {
    pub value: T,
    pub visibility: Visibility,       // Public | Group(id) | Private | Owner
    pub consent: Vec<ServiceConsent>,
    pub encrypted: bool,
}

// Beispiel:
pub struct FsPersonSensitive {
    pub date_of_birth: Option<FsPersonField<NaiveDate>>,
    pub national_id:   Option<FsPersonField<String>>,
    pub health_notes:  Option<FsPersonField<String>>,
}
```

Standard-Felder (Name, E-Mail) sind nicht per Default verschlüsselt — nur explizit als
sensitiv markierte Felder.
