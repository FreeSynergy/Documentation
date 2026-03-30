# fs-channel

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-channel` · `/home/kal/Server/fs-channel/`
**Typ:** Library-Crate (kein laufender Prozess)
**Capabilities:** `channel.trait`, `channel.matrix`, `channel.telegram`

---

## Was ist das?

`fs-channel` ist die Messaging-Abstraktion für FreeSynergy.
Sie bietet einen einheitlichen `Channel`-Trait für Matrix, Telegram und künftige Adapter.
Genutzt von `fs-bots` und zukünftigen Messaging-Apps.

---

## Design

```
Channel (Trait)
    ├── MatrixAdapter   — Matrix via matrix-sdk (Feature `matrix`)
    └── TelegramAdapter — Telegram via teloxide (Feature `telegram`)

BotChannel (Trait)
    ├── MatrixAdapter   — Bot-Operationen (DMs, Subscribe, Commands)
    └── TelegramAdapter — Bot-Operationen (Polling, Webhooks)
```

---

## Channel-Trait

```rust
#[async_trait]
pub trait Channel: Send + Sync {
    fn adapter_name(&self) -> &str;
    async fn connect(&self)                                             -> Result<(), ChannelError>;
    async fn send(&self, room_id: &str, message: ChannelMessage)       -> Result<(), ChannelError>;
    async fn subscribe(&self, on_message: Box<dyn Fn(IncomingMessage) + Send + Sync>)
                                                                       -> Result<(), ChannelError>;
}
```

| Methode        | Beschreibung                                       |
|----------------|----------------------------------------------------|
| `adapter_name` | Name des Adapters (`"matrix"`, `"telegram"`)       |
| `connect`      | Authentifizierung + persistente Verbindung öffnen  |
| `send`         | Nachricht an einen Raum/Chat senden                |
| `subscribe`    | Eingehende Nachrichten abonnieren (Callback)       |

---

## Nachrichtentypen

### ChannelMessage (ausgehend)

```rust
ChannelMessage::text("Hello!")
ChannelMessage::markdown("**bold**")
ChannelMessage::notice("System notice")
ChannelMessage::code("let x = 1;", Some("rust"))

// Mit Anhang
let msg = ChannelMessage::text("see attached").with_attachment(Attachment {
    filename: "log.txt".into(),
    mime_type: "text/plain".into(),
    data: b"...".to_vec(),
});
```

### IncomingMessage (eingehend)

```rust
// Command-Parsing
let (is_cmd, cmd, args) = IncomingMessage::parse_command("/start arg1 arg2");
// → (true, Some("start"), ["arg1", "arg2"])
```

---

## Adapter

### Matrix (Feature `matrix`)

```rust
use fs_channel::{Channel, ChannelMessage, MatrixAdapter, MatrixConfig};

let adapter = MatrixAdapter::new(MatrixConfig {
    homeserver: "https://matrix.example.com".into(),
    username: "@bot:matrix.example.com".into(),
    password: "secret".into(),
});
adapter.connect().await?;
adapter.send("!room:matrix.example.com", ChannelMessage::text("Hello!")).await?;
```

> **Hinweis:** matrix-sdk 0.16 mit rustc ≥1.94 → bekannter Recursion-Overflow (upstream).
> In-Memory State Store (kein SQLite) bis fs-db auf Postgres migriert.

### Telegram (Feature `telegram`)

```rust
use fs_channel::{Channel, ChannelMessage, TelegramAdapter, TelegramConfig};

let adapter = TelegramAdapter::new(TelegramConfig {
    token: "BOT_TOKEN".into(),
});
adapter.connect().await?;
adapter.send("-100123456789", ChannelMessage::text("Hello!")).await?;
```

---

## BotFeatures

Deklariert optionale Fähigkeiten eines Adapters:

```rust
let features = BotFeatures {
    polling:  true,
    webhooks: false,
    dms:      true,
    menus:    true,
};
```

---

## Cargo.toml

```toml
[dependencies]
fs-channel = { path = "../fs-channel" }

# Mit Matrix-Adapter:
fs-channel = { path = "../fs-channel", features = ["matrix"] }

# Mit Telegram-Adapter:
fs-channel = { path = "../fs-channel", features = ["telegram"] }
```
