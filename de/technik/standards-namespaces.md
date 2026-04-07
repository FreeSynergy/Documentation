# FS Namespace Registry

[← Standards Übersicht](standards.md) | [← Index](../INDEX.md)

---

## Übersicht

FreeSynergy nutzt eigene Schema-URNs für Extensions die noch keinen Standard haben.
Das Muster folgt SCIM Enterprise Extensions (RFC 7643 §3.3).

---

## Primärer Namespace

```
urn:freeSynergy:schemas:1.0
```

Alle FS-eigenen Schema-Erweiterungen beginnen mit diesem Prefix.

---

## Schema-URNs

| URN | Typ | Dokument |
|---|---|---|
| `urn:freeSynergy:schemas:1.0:FsPerson` | SCIM Extension | [standards-person.md](standards-person.md) |
| `urn:freeSynergy:schemas:1.0:FsGroup` | SCIM Extension | [standards-group.md](standards-group.md) |
| `urn:freeSynergy:schemas:1.0:FsOrg` | Custom Schema | [standards-org.md](standards-org.md) |

---

## Verwendung in SCIM

SCIM-Responses listen alle aktiven Schemas in `schemas[]`:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User",
    "urn:freeSynergy:schemas:1.0:FsPerson"
  ],
  "id": "2819c223-7f76-453a-919d-413861904646",
  "userName": "bjensen@example.com",
  "name": {
    "givenName": "Barbara",
    "familyName": "Jensen",
    "formatted": "Barbara Jensen"
  },
  "emails": [
    {"value": "bjensen@example.com", "type": "work", "primary": true}
  ],
  "urn:freeSynergy:schemas:1.0:FsPerson": {
    "dataProfiles": ["employees"],
    "fsCapabilities": ["federation"],
    "nodeOrigin": "node-1.example.com"
  }
}
```

---

## Verwendung in vCard

vCard 4.0 erlaubt `X-`-Extensions für eigene Felder:

```
BEGIN:VCARD
VERSION:4.0
FN:Barbara Jensen
N:Jensen;Barbara;;;
EMAIL;TYPE=work:bjensen@example.com
X-FS-DATA-PROFILES:employees
X-FS-NODE-ORIGIN:node-1.example.com
END:VCARD
```

---

## Verwendung in JSON-LD / schema.org

```json
{
  "@context": [
    "https://schema.org",
    {"fs": "urn:freeSynergy:schemas:1.0"}
  ],
  "@type": "Person",
  "givenName": "Barbara",
  "familyName": "Jensen",
  "email": "bjensen@example.com",
  "fs:dataProfiles": ["employees"],
  "fs:nodeOrigin": "node-1.example.com"
}
```

---

## Versionierung

| Version | Status | Beschreibung |
|---|---|---|
| `1.0` | Aktiv (intern) | Erste Version — noch kein IETF Draft |

**Strategie:**
1. `1.0` intern stabilisieren + öffentlich dokumentieren
2. Community nutzt es → Implementierungen entstehen
3. Wenn stabil → IETF Draft einreichen

---

## Externe Standards (Referenz)

| Standard | URN / URL | Was wir davon nutzen |
|---|---|---|
| SCIM Core | `urn:ietf:params:scim:schemas:core:2.0:User` | Basis für FsPerson |
| SCIM Groups | `urn:ietf:params:scim:schemas:core:2.0:Group` | Basis für FsGroup |
| SCIM Enterprise | `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User` | Dept, Manager, OrgRef |
| vCard 4.0 | RFC 6350 | Name, Address, Phone, Email |
| schema.org/Person | `https://schema.org/Person` | givenName, familyName, worksFor |
| schema.org/Organization | `https://schema.org/Organization` | legalName, department |
| OpenID Connect | OIDC Core 1.0 §5.1 | given_name, family_name, email, locale |
| ActivityPub | W3C Rec | federation_id, actor URLs |
