# Message Bus

[← Zurück zum Index](../INDEX.md) | [Rollen](rollen.md) | [Rechte](rechte.md) | [Föderation](foederation.md)

---

## Grundmodell: Pub/Sub + Direct Messages

Der Bus ist ein **Event-System**. Services publizieren Events ("etwas ist passiert") und andere Services können diese Events abonnieren. Der Sender weiß nicht wer reagiert — und das ist gewollt.

```
Forgejo: "Event: neuer Commit in repo helfa-website"
    ↓
Projekt-Bus veröffentlicht das Event
    ↓
Wer sich für Rolle 'git' oder Topic 'git.commit' registriert hat:
  → Matrix-Bot: postet in #dev
  → Monitoring: loggt es
  → Task: triggert Build-Pipeline
  → Lens: aktualisiert Ansicht
  → Niemand: auch OK — Event verfällt je nach Typ
```

**Pub/Sub** ist der Standard. **Direct Messages** sind die Ausnahme — für Fälle wo gezielt ein Empfänger angesprochen werden muss (z.B. ein konkreter Backup-Auftrag an eine bestimmte Datenbank-Instanz).

---

## Adressierung: IMMER über Rollen, NIE direkt

Events werden **nie** an einen konkreten Service adressiert, sondern immer an eine **Rolle** (Gruppe).

```
❌ "Sende an Forgejo"
❌ "Sende an Outline"

✅ "Sende an alle mit Rolle 'git'"
✅ "Sende an alle mit Rolle 'wiki'"
```

**Warum?** Wenn Du Forgejo durch Gitea ersetzt, ändert sich NICHTS am Bus. Beide haben die Rolle `git`. Die Abonnenten merken den Tausch nicht. Services sind austauschbar — Rollen sind stabil.

Das gilt für BEIDE Seiten:
- **Sender:** "Ich (Rolle: `git`) publiziere Event `git.commit`"
- **Empfänger:** "Ich abonniere alle Events von Rolle `git`"

In Tasks definiert man Verbindungen auch über Rollen:
```
Task: "Wenn Rolle 'git' Event 'commit' publiziert →
       sende an Rolle 'chat' als Notification"
```

Nicht: "Wenn Forgejo..." sondern "Wenn irgendetwas mit Rolle git..."

### Rollen-Hierarchie bei Abos

Standard ist die **Ober-Rolle**. Man kann auf **Unter-Rollen** einschränken:

```
"Abonniere Rolle 'database'"
  → Empfängt von Postgres, MySQL, MongoDB, SQLite, ...

"Abonniere Rolle 'database.postgres'"
  → Empfängt NUR von Postgres-Instanzen
```

Konfigurierbar pro Abonnement.

### Instanz-Tags: Wenn Rollen nicht reichen

Problem: Es gibt zwei Postgres-Datenbanken — eine für das Wiki, eine für Git. Beide haben die Rolle `database.postgres`. Ein Backup-Event soll nur die Wiki-DB treffen.

Lösung: **Instanz-Tags.** Jeder Service hat Rollen UND einen eindeutigen Instanz-Namen.

```
Service 1: Rolle database.postgres, Instanz "wiki-db"
Service 2: Rolle database.postgres, Instanz "forgejo-db"
```

Abonnement-Filter (kombinierbar):
```
"Rolle database.postgres"                          → Beide
"Rolle database.postgres, Instanz wiki-db"         → Nur die Wiki-DB
"Rolle database, Instanz forgejo-db"               → Nur die Forgejo-DB
```

**Wer vergibt den Instanz-Namen?**
- **Der Benutzer** vergibt Instanz-Namen an alle Services
- **Subservices** helfen: Sie nehmen den Standard-Namen (z.B. `postgres`) und der übergeordnete Service macht einen Prefix/Postfix dran (z.B. `outline-postgres`, `forgejo-postgres`)
- Der Instanz-Name ist KEIN Service-Name — er ist der Name den DU der konkreten Installation gibst

---

## Nachrichtentypen

Es gibt **zwei Dimensionen** für jede Nachricht:

### Dimension 1: Lebensdauer

| Typ | Verhalten |
|---|---|
| **Fire & Forget** | Einmal senden, wer da ist empfängt, dann weg. Wenn niemand zuhört → verloren (und das ist OK) |
| **Guaranteed Delivery** | Bleibt in der Queue bis der Empfänger bestätigt (ACK). Wird nicht verloren |
| **Standing Order** | Dauerhafte Regel. Feuert IMMER WIEDER wenn ein passender Trigger kommt. Wird nie gelöscht, nur deaktiviert |

### Dimension 2: Speicherung

| Typ | Verhalten |
|---|---|
| **Nicht speichern** | Flüchtig, nach Zustellung weg |
| **Speichern bis ACK** | In DB bis Empfänger bestätigt, dann gelöscht |
| **Immer speichern** | Persistent, Audit-Log, Replay möglich |

### Kombinations-Matrix

Jede Nachricht hat eine Lebensdauer UND eine Speicherung:

| Beispiel | Lebensdauer | Speicherung |
|---|---|---|
| Suche (Anfrage + Ergebnis) | Fire & Forget | Nicht speichern |
| Live-Status "Service X online" | Fire & Forget | Nicht speichern |
| Desktop-Notification "Neuer Commit" | Fire & Forget | Nicht speichern |
| Chat-Nachricht weiterleiten | Fire & Forget | Nicht speichern |
| Monitoring-Metriken (CPU, RAM) | Fire & Forget | Nicht speichern |
| Service installiert → andere informieren | Guaranteed | Speichern bis ACK |
| Task ausführen (Automatisierung) | Guaranteed | Speichern bis ACK |
| Backup-Auftrag | Guaranteed | Immer speichern |
| User-Account synchronisieren (SCIM) | Guaranteed | Immer speichern |
| Bot-Command ausführen | Guaranteed | Speichern bis ACK |
| Einladung verarbeiten | Guaranteed | Immer speichern |
| Rechte-Änderung propagieren | Guaranteed | Immer speichern |
| "Jedes Wiki braucht System-Doku" | Standing Order | Immer speichern |
| "Jeder neue User → Willkommensnachricht" | Standing Order | Immer speichern |
| "Jeder Service → Healthcheck im Monitoring" | Standing Order | Immer speichern |
| "Jedes Repository → README" | Standing Order | Immer speichern |

### Wer bestimmt den Typ?

- **Der Sender** bestimmt den Default-Typ
- Es gibt eine **konfigurierbare Default-Zuordnung** im Container App Manager (regelbasiert)
- Regeln: `WENN Event X UND Quelle Y, DANN Typ Z`

### Standing Orders im Detail

Standing Orders feuern **immer wieder** wenn ein passender Service installiert wird.

Beispiel: Standing Order "Jedes Wiki braucht System-Dokumentation"
1. Neues Wiki wird installiert (Outline)
2. Bus prüft: Gibt es Standing Orders die auf Rolle `wiki` matchen?
3. Ja → Standing Order feuert: "Erstelle System-Dokumentation für Outline"
4. Standing Order bleibt aktiv für zukünftige Wiki-Installationen

Standing Orders werden **nur geprüft wenn ein passender Service installiert wird** — nicht bei jeder Installation.

---

## Ein Bus pro Ebene (Dezentral)

Jede Ebene hat ihren **eigenen, unabhängigen Bus**. Busse sind über **Bridges** verbunden.

```
Projekt "Helfa Köln"
  └── Projekt-Bus (alle lokalen Services reden hier)
        │
        │ ← Bridge (filtert nach Rechte-Kaskade)
        ▼
Föderation "FreeSynergy Deutschland"
  └── Föderations-Bus (nur weitergeleitete Events)
        │
        │ ← Bridge (nochmal gefiltert)
        ▼
Über-Föderation "FreeSynergy Europa"
  └── Über-Föderations-Bus
```

### Bridges sind Checkpoints

Jede Bridge prüft:
1. Hat der Sender die Events zur Weitergabe freigegeben? (Rechte-Kaskade)
2. Welche Rechte hat die Ziel-Ebene? (read? search? write? execute?)
3. Darf dieses Event nach oben? (Konfiguration der Bridge)

Ein Event kann NICHT "aus Versehen" nach oben durchrutschen.

### Doppelter Schutz: Bridge + Empfänger-Bus

Die Bridge filtert beim Übergang (Rechte-Kaskade). Der Empfänger-Bus hat **zusätzlich eigene Regeln**. Zwei Checkpoints.

```
Projekt-Bus
  → Bridge prüft: Darf das Event in die Föderation? (Rechte)
    → Föderations-Bus prüft: Darf ein Abonnent das empfangen? (eigene Regeln)
```

### Warum pro Ebene?

- **Langsamer:** Ja, Events brauchen Zeit um durch Bridges zu kommen
- **Sicherer:** Ja, jede Ebene kontrolliert selbst was rein und raus darf
- **Dezentraler:** Ja, wenn eine Ebene ausfällt, funktionieren die anderen weiter

---

## Abonnement-Regeln

Wer darf was abonnieren?

1. **Rechte sind Pflicht.** Ohne `read`-Recht auf den Sender-Service kein Abonnement
2. **Rollen sind der Standard-Filter.** Man abonniert über Rollen, nicht über Service-Namen
3. **Topics als zusätzlicher Filter.** Event-Typen einschränken (`git.commit`, `wiki.page.created`)
4. **Instanz-Tags als letzter Filter.** Wenn Rollen nicht reichen (zwei Postgres-DBs)

Kombinierbar:
```
Abonniere:
  Rolle: database.postgres
  Topic: database.backup.*
  Instanz: wiki-db
  → Empfängt NUR Backup-Events von der Wiki-Postgres-Instanz
```

---

## Event-Format

```rust
pub struct BusEvent {
    // Identifikation
    pub id: Uuid,
    pub timestamp: DateTime<Utc>,

    // Adressierung (über Rollen, NIE direkt)
    pub source_role: Role,
    pub source_instance: String,
    pub target_role: Option<Role>,           // None = Broadcast an alle Abonnenten
    pub target_instance: Option<String>,     // None = alle Instanzen der Rolle

    // Inhalt
    pub topic: String,                       // z.B. "git.commit", "wiki.page.created"
    pub payload: serde_json::Value,

    // Verhalten
    pub delivery: DeliveryType,
    pub storage: StorageType,

    // Kontext
    pub project_id: Option<ProjectId>,
    pub federation_id: Option<FederationId>,
    pub correlation_id: Option<Uuid>,        // Verknüpfung mit vorherigem Event
}

pub enum DeliveryType {
    FireAndForget,
    Guaranteed,
    StandingOrder,
}

pub enum StorageType {
    NoStore,
    UntilAck,
    Persistent,
}
```

---

## Konfigurierbare Default-Zuordnung

Im Container App Manager gibt es eine **regelbasierte Liste**:

```toml
[[bus.rules]]
condition = { topic = "search.*" }
delivery = "fire_and_forget"
storage = "no_store"

[[bus.rules]]
condition = { topic = "*.install", source_role = "container-app" }
delivery = "guaranteed"
storage = "until_ack"

[[bus.rules]]
condition = { topic = "rights.*" }
delivery = "guaranteed"
storage = "persistent"

[[bus.rules]]
condition = { topic = "backup.*" }
delivery = "guaranteed"
storage = "persistent"

[[bus.rules]]
condition = { topic = "standing.*" }
delivery = "standing_order"
storage = "persistent"

# Default: wenn keine Regel passt
[[bus.rules]]
condition = { topic = "*" }
delivery = "fire_and_forget"
storage = "no_store"
```

Diese Liste ist **editierbar** — Defaults ändern, neue Regeln hinzufügen, bestehende anpassen.

Regeln können auch Quelle und Ziel einschränken:
```toml
[[bus.rules]]
condition = { topic = "database.backup.*", source_role = "database", target_instance = "wiki-db" }
delivery = "guaranteed"
storage = "persistent"
```

---

Weiter: [Tasks](tasks.md) | [Föderation](foederation.md) | [Rechte](rechte.md) | [Search](../programme/search/README.md)
