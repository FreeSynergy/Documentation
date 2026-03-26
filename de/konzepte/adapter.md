# Adapter-Pattern

[← Zurück zum Index](../INDEX.md) | [Registry](registry.md) | [Message Bus](bus.md) | [Rollen](rollen.md)

---

## Warum Adapter statt Bridge

FreeSynergy muss mit externen Diensten reden: Kanidm für IAM, Stalwart für Mail,
Tuwunel für Matrix, Forgejo für Git. Jeder Dienst hat eine eigene API.

Früher gab es einen generischen "Bridge-Executor" der APIs dynamisch gemappt hat.
Dieses Konzept ist entfernt (März 2026) — es war schwer zu testen, nicht typsicher,
und erzeugte eine Komplexitätsschicht die niemand wirklich verstanden hat.

**Lösung:** Jede Rolle die FreeSynergy braucht bekommt einen **Standard-Trait**.
Wer diesen Dienst bereitstellt, schreibt einen schlanken Adapter der diesen Trait
implementiert. Der Compiler prüft den Vertrag. Tests nutzen Mock-Adapter.

---

## Standard-Traits

FreeSynergy definiert Traits für alle Capabilities die das System kennt.
Diese Traits leben in den jeweiligen Programm-Repos — nicht in fs-libs.

```rust
// fs-node/src/auth/service.rs
pub trait AuthService: Send + Sync {
    async fn authenticate(&self, token: &str) -> Result<Claims, AuthError>;
    async fn check_permission(&self, user: &str, action: &str) -> Result<bool, AuthError>;
    async fn capabilities(&self) -> Vec<String>;
}

// fs-node/src/mail/service.rs
pub trait MailService: Send + Sync {
    async fn send(&self, message: &MailMessage) -> Result<(), MailError>;
    async fn capabilities(&self) -> Vec<String>;
}

// fs-store/src/storage/service.rs
pub trait StorageService: Send + Sync {
    async fn put(&self, path: &str, data: &[u8]) -> Result<(), StorageError>;
    async fn get(&self, path: &str) -> Result<Vec<u8>, StorageError>;
    async fn delete(&self, path: &str) -> Result<(), StorageError>;
    async fn capabilities(&self) -> Vec<String>;
}
```

---

## Adapter schreiben

Ein Adapter ist ein Struct das den Standard-Trait implementiert.
Er übersetzt zwischen FreeSynergys Sprache und der externen API.

```rust
// Adapter für Kanidm (IAM)
pub struct KanidmAdapter {
    base_url: String,
    client:   reqwest::Client,
}

impl AuthService for KanidmAdapter {
    async fn authenticate(&self, token: &str) -> Result<Claims, AuthError> {
        // kanidm-spezifischer HTTP-Call
        let resp = self.client
            .get(format!("{}/v1/auth/valid", self.base_url))
            .bearer_auth(token)
            .send()
            .await?;
        // ...in Claims übersetzen
    }

    async fn check_permission(&self, user: &str, action: &str) -> Result<bool, AuthError> {
        // kanidm-spezifischer HTTP-Call
    }

    async fn capabilities(&self) -> Vec<String> {
        vec!["iam".into(), "iam.oidc".into(), "iam.scim".into()]
    }
}

// Mock für Tests — kein echter HTTP-Call
pub struct MockAuthAdapter {
    allow_all: bool,
}

impl AuthService for MockAuthAdapter {
    async fn authenticate(&self, _token: &str) -> Result<Claims, AuthError> {
        if self.allow_all {
            Ok(Claims::test_user())
        } else {
            Err(AuthError::Unauthorized)
        }
    }
    // ...
}
```

---

## Ablauf: Programm nutzt eine Capability

```
Programm braucht IAM
  │
  ▼
fragt fs-registry: "Wer bietet Capability 'iam' an?"
  │
  ▼
Registry antwortet: "KanidmAdapter, Endpoint: http://kanidm:8443"
  │
  ▼
Programm ruft AuthService-Methode auf
  │
  ▼
KanidmAdapter übersetzt → kanidm HTTP-API
  │
  ▼
Ergebnis zurück ins aufrufende Programm
```

Kein generischer Dispatcher. Kein dynamisches Mapping.
Der Compiler prüft den Vertrag vollständig.

---

## Registrierung in der Registry

Jeder Adapter registriert sich beim Start in `fs-registry` und
deregistriert sich beim Shutdown:

```rust
// Beim Start
let entry = ServiceEntry::builder()
    .service_id("kanidm")
    .endpoint("http://kanidm:8443")
    .capabilities(adapter.capabilities().await)
    .build();
registry.register(entry).await?;

// Beim Shutdown (via Drop oder Signal-Handler)
registry.deregister("kanidm").await?;
```

Programme fragen die Registry, nie den Adapter direkt.
Der Bus routet ebenfalls über die Registry — kein direktes Adapter-Lookup im Bus.

---

## Mehrere Adapter für dieselbe Capability

Ein Node kann mehrere Dienste für dieselbe Capability haben
(z.B. zwei Mail-Server für verschiedene Domains):

```rust
// Beide registrieren sich mit derselben Capability
registry.register(ServiceEntry {
    id: "stalwart-main", capability: "mail", endpoint: "http://stalwart-1:8080"
}).await?;

registry.register(ServiceEntry {
    id: "stalwart-backup", capability: "mail", endpoint: "http://stalwart-2:8080"
}).await?;

// Abfrage gibt alle zurück
let mail_services = registry.by_capability("mail").await?;
// → [stalwart-main, stalwart-backup]
```

Wer von mehreren gewählt wird, entscheidet das aufrufende Programm
(erste verfügbare, nach Last, nach Konfiguration — je nach Anforderung).

---

## Wann einen Adapter schreiben

| Situation | Lösung |
|---|---|
| Drittanbieter-Dienst erfüllt eine FS-Rolle | Adapter schreiben, in Registry registrieren |
| Eigener FS-Dienst | Standard-Trait direkt implementieren |
| Mehrere Dienste für dieselbe Rolle | Je ein Adapter — alle registrieren sich |
| Tests ohne echten Dienst | `MockAdapter` — implementiert denselben Trait |
| Neuer externer Dienst | Neuen Trait definieren wenn nötig, sonst bestehenden nutzen |

---

## Konventionen

- Adapter-Struct heißt `{ServiceName}Adapter` (z.B. `KanidmAdapter`, `StalwartAdapter`)
- Mock-Struct heißt `Mock{TraitName}` (z.B. `MockAuthService`)
- Traits liegen im jeweiligen Programm-Repo, nicht in `fs-libs`
- Jeder Adapter hat mindestens einen Unit-Test mit Mock-Adapter
- `#[async_trait]` für async Trait-Implementierungen (bis native async traits stabil sind)

---

## Was nicht mehr existiert

- `fs-bridge` — gelöscht (März 2026)
- `fs-bridge-sdk` — gelöscht (März 2026)
- Dynamischer Bridge-Executor — gelöscht (März 2026)
- `BridgeRef` in `ServiceInstance` — ersetzt durch `capabilities_provided/required`

---

Weiter: [Registry](registry.md) | [Message Bus](bus.md) | [Inventory](inventory.md)
