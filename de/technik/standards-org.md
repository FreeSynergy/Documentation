# FsOrg — Field Mappings

[← Standards Übersicht](standards.md) | [← Index](../INDEX.md)

---

## Kanonische Felder

### Identität

| FS (kanonisch) | Typ | schema.org/Organization | vCard KIND:org | LDAP | ActivityPub |
|---|---|---|---|---|---|
| `id` | `FsId` | `@id` | `UID` | `entryUUID` | `@id` |
| `name` | `String` | `name` | `FN` / `ORG` (first) | `o` | `name` |
| `legal_name` | `Option<String>` | `legalName` | — | — | — |
| `display_name` | `String` | `name` | `FN` | `cn` | `name` |
| `external_id` | `Option<String>` | — | — | — | — |

### Kontakt

| FS (kanonisch) | Typ | schema.org | vCard | LDAP |
|---|---|---|---|---|
| `emails` | `Vec<FsEmail>` | `email` | `EMAIL` | `mail` |
| `phones` | `Vec<FsPhone>` | `telephone` | `TEL` | `telephoneNumber` |
| `addresses` | `Vec<FsAddress>` | `address` (PostalAddress) | `ADR` | `postalAddress` |
| `url` | `Option<FsUrl>` | `url` | `URL` | `labeledURI` |

### Struktur

| FS (kanonisch) | Typ | schema.org | LDAP | Notiz |
|---|---|---|---|---|
| `parent_org` | `Option<FsOrgRef>` | `parentOrganization` | `o` ancestor | Hierarchie |
| `sub_orgs` | `Vec<FsOrgRef>` | `subOrganization` | `ou` children | Abteilungen |
| `members` | `Vec<FsPersonRef>` | `member` | `member` | Direkte Mitglieder |
| `departments` | `Vec<FsGroupRef>` | `department` | `ou` | Teams/Abteilungen als Groups |

### Meta

| FS (kanonisch) | Typ | schema.org | Notiz |
|---|---|---|---|
| `founded` | `Option<NaiveDate>` | `foundingDate` | ISO 8601 |
| `description` | `Option<String>` | `description` | |
| `logo_url` | `Option<FsUrl>` | `logo` | |
| `created_at` | `DateTime<Utc>` | — | intern |
| `updated_at` | `DateTime<Utc>` | — | intern |

---

## FS-Extension (`FsOrgExtension`)

| Feld | Typ | Bedeutung |
|---|---|---|
| `node_origin` | `Option<FsNodeRef>` | Von welchem Node importiert |
| `federation_id` | `Option<FsUrl>` | ActivityPub Organization-URL |
| `default_data_profile` | `Option<DataProfileRef>` | Wird neuen Mitgliedern zugewiesen |
| `org_type` | `FsOrgType` | Company / NGO / Government / Community / Internal |

---

## Beziehungen zwischen den kanonischen Typen

```
FsOrg
  ├── departments: Vec<FsGroup>
  │     └── members: Vec<FsPerson>  (mit DataProfile → Provisioning)
  └── members: Vec<FsPerson>        (direkte Mitglieder ohne Gruppe)

FsPerson
  ├── org_ref: Option<FsOrg>
  ├── groups: Vec<FsGroup>
  └── manager: Option<FsPerson>
```

---

## Roundtrip-Verluste (dokumentiert)

| Richtung | Verlust |
|---|---|
| `FsOrg` → LDAP | Nur `o`/`ou` Attribute — keine Extensions, keine Hierarchie-Tiefe |
| `FsOrg` → SCIM | SCIM hat kein Org-Schema — als Custom Extension exportieren |
| schema.org → `FsOrg` | JSON-LD `@context` geht verloren — nur Felder übernommen |
