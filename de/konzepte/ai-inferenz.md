# AI-Inferenz (lokale LLMs)

FreeSynergy integriert lokale LLM-Inferenz über **Mistral.rs** — einen nativen Rust-Binary mit OpenAI-kompatibler API. Es läuft direkt auf der Hardware, ohne Container-Overhead.

**Wichtig:** Mistral.rs ist eine **App** — installierbar auf jedem System, nicht nur auf FSN-Node-Servern. Dasselbe gilt für alle AI/LLM-Tools: Sie sind eigenständige Programme, die überall laufen können.

---

## Installation

```bash
# Als eigenständige App (überall installierbar)
fsn store install mistral

# Oder als Hintergrunddienst via FSN Node (empfohlen für Server)
fsn store install ai/mistral
```

| Eigenschaft   | Wert                                  |
|---|---|
| Store-ID (App)    | `mistral`                         |
| Store-ID (Dienst) | `ai/mistral` (Node-Modul)         |
| Port              | `1234`                            |
| API               | OpenAI-kompatibel (`/v1/chat/completions` etc.) |
| Quelle            | `FreeSynergy/mistral.rs` (Fork)   |

---

## Modell-Empfehlung (CPU, 16 GB RAM)

### ISQ-Modus (empfohlen)

mistral.rs lädt das Modell direkt von HuggingFace und quantisiert es **in-memory** (In-Situ Quantization). Kein manueller Download einer `.gguf`-Datei nötig.

| Modell            | ISQ-Typ | RAM     | Geschwindigkeit | Wofür                       |
|---|---|---|---|---|
| `Qwen/Qwen3-4B`   | `q4k`   | ~3–4 GB | schnell         | Code, Chat, Übersetzungen   |
| `Qwen/Qwen3-8B`   | `q4k`   | ~6 GB   | mittel          | Komplexere Aufgaben         |

**Standard-Konfiguration:**
```
Model ID: Qwen/Qwen3-4B
ISQ:      q4k
```

Beim ersten Start lädt mistral.rs das Modell herunter (~4 GB) und cached es lokal. Jeder weitere Start ist sofort.

### GGUF-Modus (alternativ)

Wer ein bereits heruntergeladenes `.gguf`-File hat:
- `mistral_model_id` = absoluter Pfad zur Datei (z.B. `/srv/fsn/data/mistral/models/qwen.gguf`)
- `mistral_isq` = leer lassen

---

## Start / Stop

```bash
fsn start ai/mistral     # starten (lädt Modell in RAM)
fsn stop ai/mistral      # stoppen (gibt RAM frei)
fsn status ai/mistral    # Status anzeigen
```

Mistral.rs kann jederzeit gestoppt werden, um RAM freizugeben — z.B. für andere speicherintensive Tasks. Beim nächsten Start dauert das Laden des Modells ~5–20 Sekunden.

---

## Workflow: Claude + Mistral + Entwickler

```
Entwickler
   │
   ├── Claude Code (Anthropic API)
   │     └── Wofür: Architektur, komplexe Bugs, neue Features,
   │                Cross-Repo-Arbeit, unbekannte Probleme,
   │                Planung + Konzepte
   │
   └── Mistral.rs (lokal, localhost:1234)
         └── Wofür: Code-Completions im Editor,
                    Kleine Fragen + Erklärungen,
                    Boilerplate generieren,
                    Docs-Drafts, Übersetzungen,
                    SVGs, Templates
```

**Prinzip:** Mistral zuerst für alles Kleine → spart Claude-Tokens für das Wesentliche.

---

## Editor-Integration (Continue.dev)

Continue.dev in VSCode auf den lokalen Endpunkt zeigen:

```json
{
  "models": [
    {
      "title": "Qwen3 (lokal)",
      "provider": "openai",
      "model": "qwen3-4b",
      "apiBase": "http://localhost:1234/v1",
      "apiKey": "none"
    }
  ]
}
```

---

## Build (FSN Fork)

Mistral.rs wird aus `FreeSynergy/mistral.rs` gebaut:

- CI: `.github/workflows/fsn-build.yml`
- Targets: Linux x86\_64/aarch64, macOS x86\_64 (Accelerate), macOS aarch64 (Metal), Windows x86\_64
- Release-Tag: `v{version}-cpu`

```bash
# Build auslösen:
git tag v0.7.1 && git push origin v0.7.1
```
