# FsGroup — Field Mappings

[← Standards Übersicht](standards.md) | [← Index](../INDEX.md)

---

## Kanonische Felder

### Identität

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 (KIND:group) | LDAP | ActivityPub |
|---|---|---|---|---|---|
| `id` | `FsId` | `id` (UUID) | `UID` | `entryUUID` | `@id` |
| `display_name` | `String` | `displayName` | `FN` | `cn` | `name` |
| `external_id` | `Option<String>` | `externalId` | — | — | — |

### Mitglieder

| FS (kanonisch) | Typ | SCIM RFC 7643 | vCard RFC 6350 | LDAP |
|---|---|---|---|---|
| `members` | `Vec<FsGroupMember>` | `members[{value,display,type}]` | `MEMBER` URIs | `member` (DN-Liste) |

#### FsGroupMember

| FS | SCIM | LDAP | Notiz |
|---|---|---|---|
| `ref_id` | `value` (UUID) | DN | |
| `display` | `display` | — | Anzeigename (denormalisiert) |
| `member_type` | `type` (User/Group) | — | Nested Groups via type=Group |

### Meta

| FS (kanonisch) | Typ | SCIM RFC 7643 | Notiz |
|---|---|---|---|
| `created_at` | `DateTime<Utc>` | `meta.created` | |
| `updated_at` | `DateTime<Utc>` | `meta.lastModified` | |

---

## FS-Extension (`FsGroupExtension`)

Felder ohne existierenden Standard — `urn:freeSynergy:schemas:1.0:FsGroup`:

| Feld | Typ | Bedeutung |
|---|---|---|
| `data_profile` | `Option<DataProfile>` | Welche Services Gruppen-Mitglieder bekommen |
| `auto_provision` | `bool` | Provisioning bei Eintritt/Austritt via Bus |
| `group_type` | `FsGroupType` | Team / Project / Role / Department / External |
| `federation_id` | `Option<FsUrl>` | ActivityPub Group-URL (für föderale Gruppen) |
| `node_origin` | `Option<FsNodeRef>` | Von welchem Node importiert |

---

## DataProfile — Group-driven Provisioning

Ein `DataProfile` beschreibt, welche Felder einer Person an welche Services synchronisiert werden,
wenn sie Mitglied dieser Gruppe ist:

```rust
pub struct DataProfile {
    pub id: FsId,
    pub name: String,
    pub entries: Vec<DataProfileEntry>,
}

pub struct DataProfileEntry {
    pub service_id: String,           // "iam", "wiki", "chat", "calendar"
    pub enabled: bool,
    pub fields: Vec<FieldConsent>,    // welche FsPerson-Felder gehen raus
    pub on_join: ProvisionAction,     // Create | Update | Skip
    pub on_leave: DeprovisionAction,  // Deactivate | Delete | Skip
}
```

### Beispiel: Gruppe "Mitarbeiter"

```toml
[data_profile]
id = "uuid-..."
name = "Mitarbeiter"

[[data_profile.entries]]
service_id = "iam"
enabled = true
fields = ["given_name", "family_name", "email.work", "job_title"]
on_join = "Create"
on_leave = "Deactivate"

[[data_profile.entries]]
service_id = "wiki"
enabled = true
fields = ["given_name", "avatar_url"]
on_join = "Create"
on_leave = "Deactivate"

[[data_profile.entries]]
service_id = "chat"
enabled = true
fields = ["given_name", "avatar_url", "email.work"]
on_join = "Create"
on_leave = "Deactivate"

[[data_profile.entries]]
service_id = "calendar"
enabled = false
```

### Bus-Event-Flow bei Gruppen-Beitritt

```
UserJoinedGroup { user_id, group_id }
  → ProvisioningCoordinator holt DataProfile der Gruppe
  → für jeden enabled Entry:
      Bus-Event: ProvisionUser { user_id, service_id, fields, profile }
        → KanidmProvisioningAdapter
        → OutlineProvisioningAdapter
        → MatrixProvisioningAdapter
```

---

## Roundtrip-Verluste (dokumentiert)

| Richtung | Verlust |
|---|---|
| `FsGroup` → LDAP | `data_profile` geht verloren (kein LDAP-Äquivalent) |
| `FsGroup` → SCIM | `data_profile` als Enterprise Extension exportierbar |
| vCard GROUP → `FsGroup` | Nur Mitglieder-URIs, kein DataProfile |
