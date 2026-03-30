# fs-llm

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-llm` · `/home/kal/Server/fs-llm/`
**Typ:** Library-Crate (kein laufender Prozess)
**Capabilities:** `llm.trait`, `llm.ollama`, `llm.claude`, `llm.openai-compat`

---

## Was ist das?

`fs-llm` ist die LLM-Abstraktion für FreeSynergy.
Einheitlicher `LlmProvider`-Trait für Ollama, Anthropic Claude und OpenAI-kompatible APIs.
Genutzt von `fs-ai`, `fs-bots` und anderen Diensten.

---

## Design

```
LlmProvider (Trait)
    ├── OllamaProvider       — Ollama HTTP API (Feature `ollama`)
    ├── ClaudeProvider       — Anthropic Claude API (Feature `claude`)
    └── OpenAiCompatProvider — OpenAI-kompatibler Endpunkt (Feature `openai-compat`)

tasks — High-Level-Helfer (arbeiten mit &dyn LlmProvider)
```

---

## LlmProvider-Trait

```rust
#[async_trait]
pub trait LlmProvider: Send + Sync {
    fn name(&self) -> &str;
    async fn complete(&self, messages: Vec<Message>) -> Result<LlmResponse, LlmError>;
    async fn ask(&self, prompt: &str)                -> Result<String, LlmError>;
    async fn ask_with_system(&self, system: &str, prompt: &str) -> Result<String, LlmError>;
    async fn embed(&self, input: &str)               -> Result<EmbedResponse, LlmError>;
    async fn list_models(&self)                      -> Result<Vec<ModelInfo>, LlmError>;
}
```

| Methode            | Beschreibung                                          |
|--------------------|-------------------------------------------------------|
| `name`             | Provider-Name (`"ollama"`, `"claude"`, …)             |
| `complete`         | Vollständige Conversation → Antwort                   |
| `ask`              | Einzelne User-Nachricht → Antwort-Text                |
| `ask_with_system`  | System-Prompt + User-Nachricht → Antwort-Text         |
| `embed`            | Text → Embedding-Vektor (optional, default: Error)    |
| `list_models`      | Verfügbare Modelle (default: `[{id: name}]`)          |

---

## Nachrichtentypen

```rust
Message::system("You are a helpful assistant.")
Message::user("What is 2 + 2?")
Message::assistant("4")

// Antwort
let resp: LlmResponse = provider.complete(messages).await?;
println!("{}", resp.content);  // "4"
println!("{}", resp.model);    // "llama3"
```

---

## Adapter

### Ollama (Feature `ollama`)

```rust
use fs_llm::OllamaProvider;

let provider = OllamaProvider::new("http://localhost:11434", "llama3");
let reply = provider.ask("Summarize this log.").await?;
```

### Claude (Feature `claude`)

```rust
use fs_llm::ClaudeProvider;

let provider = ClaudeProvider::new("ANTHROPIC_API_KEY", "claude-3-haiku-20240307");
let reply = provider.ask("What is Rust?").await?;
```

### OpenAI-kompatibel (Feature `openai-compat`)

```rust
use fs_llm::OpenAiCompatProvider;

let provider = OpenAiCompatProvider::new(
    "http://localhost:8080/v1",
    "API_KEY",
    "mistral-7b",
);
let reply = provider.ask("Hello!").await?;
```

---

## High-Level Tasks

```rust
use fs_llm::tasks;

let summary     = tasks::summarize_logs(&provider, &log_text).await?;
let cmd         = tasks::interpret_command(&provider, &user_input).await?;
let explanation = tasks::explain_error(&provider, &error_message).await?;
```

---

## Eigenen Provider implementieren

```rust
use async_trait::async_trait;
use fs_llm::{LlmError, LlmProvider, LlmResponse, Message};

struct MyProvider;

#[async_trait]
impl LlmProvider for MyProvider {
    fn name(&self) -> &str { "my-provider" }

    async fn complete(&self, messages: Vec<Message>) -> Result<LlmResponse, LlmError> {
        // ...
        Ok(LlmResponse::new("response text", "my-model"))
    }
}
```

---

## Cargo.toml

```toml
[dependencies]
fs-llm = { path = "../fs-llm" }

# Mit Ollama:
fs-llm = { path = "../fs-llm", features = ["ollama"] }

# Alle Provider:
fs-llm = { path = "../fs-llm", features = ["ollama", "claude", "openai-compat"] }
```
