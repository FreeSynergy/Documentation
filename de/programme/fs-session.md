# fs-session

**User Session Management für FreeSynergy**

Verwaltet den Laufzeitkontext eines eingeloggten Benutzers: wer ist aktiv,
welche Anwendungen sind geöffnet, und in welchem Fensterzustand befinden sie sich.

**Repository:** `FreeSynergy/fs-session` — Library-Crate (kein eigener Prozess)

---

## Das Minimize-Problem (und wie fs-session es löst)

**Ohne Session:**
```
User minimiert fs-store → Desktop vergisst das Fenster
User klickt auf Icon  → Desktop startet fs-store NEU
Ergebnis: zwei Instanzen offen, State verloren
```

**Mit fs-session:**
```
User minimiert fs-store → Session setzt AppState auf Minimized
User klickt auf Icon  → Desktop fragt Session: "Ist fs-store offen?"
                       → Session: "Ja, Minimized"
                       → Desktop: Fenster restore (kein Neustart)
Ergebnis: eine Instanz, State erhalten
```

---

## Architektur

```
SessionStore (Trait)
    │
    └── SqliteSessionStore  — SQLite-Backend (fs-session.db)

SessionTracker (Trait)
    │
    └── StoreBackedTracker  — leitet Desktop-Events an SessionStore weiter

Session
    ├── id, user_id, display_name, started_at
    └── Vec<AppSession>
            ├── app_id     (z. B. "fs-store")
            ├── state      (Open | Minimized | Focused)
            └── opened_at
```

### Traits

#### `SessionStore`

Interface für alle Session-Storage-Backends. Consumer-Code darf nur gegen
diesen Trait programmieren — nie gegen `SqliteSessionStore` direkt.

| Methode | Beschreibung |
|---|---|
| `create(user_id, display_name)` | Neue Session erstellen (Login) |
| `get(session_id)` | Session laden |
| `get_for_user(user_id)` | Aktuelle Session eines Users |
| `list()` | Alle aktiven Sessions |
| `active_user()` | Zuletzt gestartete Session (Single-User: die einzige) |
| `close(session_id)` | Session löschen (Logout) |
| `open_app(session_id, app_id)` | App als geöffnet markieren |
| `minimize_app(session_id, app_id)` | App minimieren |
| `restore_app(session_id, app_id)` | App wiederherstellen |
| `close_app(session_id, app_id)` | App schließen |

#### `SessionTracker`

Empfängt Fenster-Events vom Desktop und übersetzt sie in `SessionStore`-Aufrufe.

| Methode | Desktop-Event |
|---|---|
| `on_app_opened` | Fenster erstmals geöffnet |
| `on_app_minimized` | Fenster in Taskleiste minimiert |
| `on_app_focused` | Fenster erhält Fokus |
| `on_app_closed` | Fenster vom User geschlossen |

`StoreBackedTracker<S: SessionStore>` ist die Standard-Implementierung.

---

## API-Verwendung

```rust
use fs_session::{SessionStore, SqliteSessionStore};

let store = SqliteSessionStore::open("fs-session.db").await?;

// Login
let session = store.create("user-42", "Alice").await?;

// App öffnen
store.open_app(session.id(), "fs-store").await?;

// Minimieren
store.minimize_app(session.id(), "fs-store").await?;

// Restore (kein Neustart)
store.restore_app(session.id(), "fs-store").await?;

// Logout
store.close(session.id()).await?;
```

---

## Desktop-Integration via SessionTracker

```rust
use fs_session::{SqliteSessionStore, SessionTracker, StoreBackedTracker};

let store = SqliteSessionStore::open("fs-session.db").await?;
let tracker = StoreBackedTracker::new(store);

// Desktop schickt Events:
tracker.on_app_opened(session_id, "fs-lenses").await?;
tracker.on_app_minimized(session_id, "fs-lenses").await?;
tracker.on_app_focused(session_id, "fs-lenses").await?;
tracker.on_app_closed(session_id, "fs-lenses").await?;
```

---

## Datenbank

- Datei: `fs-session.db` (SQLite via SeaORM)
- Eine Tabelle: `sessions` — JSON-Spalte `apps` für `Vec<AppSession>`
- Index auf `user_id` und `started_at`

---

## Abgrenzung

| | fs-session | fs-inventory | fs-registry |
|---|---|---|---|
| Frage | Wer ist eingeloggt + was läuft? | Was ist installiert? | Welche Services laufen? |
| Lebensdauer | Login bis Logout | Persistent | Laufzeit |
| User-abhängig | Ja | Nein | Nein |

---

[← Index](../INDEX.md) | [Konzept: Session](../konzepte/session.md)
