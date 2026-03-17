# Init — Der Bootstrap

[← Zurück zum Index](../../INDEX.md) | [Store](../store/README.md)

---

## Was Init macht

FreeSynergy.Init ist ein **minimales Binary** das genau eine Aufgabe hat: Den Store installieren. Mehr nicht.

```
User lädt Init herunter (vorkompiliert oder Source-Build)
  → Init klont den Store via gitoxide (reines Rust-Git)
    → Init installiert den Store-CLI
      → Store übernimmt ALLES weitere
        → Init wird nie wieder gebraucht
```

## Warum Init?

Jedes System braucht einen Startpunkt. Man kann keinen Paketmanager mit sich selbst installieren. Init ist dieser Startpunkt — so minimal wie möglich, danach übernimmt der Store.

## Technisch

- **Sprache:** Rust
- **Git:** `gitoxide` (gix) — reine Rust-Implementierung, keine externe Git-Abhängigkeit
- **Größe:** So klein wie möglich — nur Git-Klon + Store-Binary starten
- **Kein Netzwerk außer Git:** Kein HTTP, kein REST, kein API-Server

## Distribution

Vorkompilierte Binaries über GitHub Actions für:

| Target | Plattform |
|---|---|
| `x86_64-unknown-linux-gnu` | Desktop Linux, Server |
| `aarch64-unknown-linux-gnu` | Raspberry Pi, ARM Server |
| `x86_64-apple-darwin` | macOS Intel |
| `aarch64-apple-darwin` | macOS Apple Silicon |
| `x86_64-pc-windows-msvc` | Windows |

Alternativ: Source-Build mit `cargo install freesynergy-init`.

## Nach dem Init

Init ist fertig. Der Store ist da. Ab jetzt:

```bash
# Was will ich installieren?
fsn store search --type group     # Zeige Installations-Gruppen

# Server-Installation
fsn store install server-minimal  # Node + Conductor + Proxy + S3
fsn store install server-full     # + Mail + Wiki + Chat + Git + ...

# Desktop-Installation
fsn store install desktop         # Desktop + Standard-Themes + DE + EN

# Einzeln
fsn store install kanidm
fsn store install outline
```

## Repo

https://github.com/FreeSynergy/Init

## Bibliotheken

| Crate | Zweck |
|---|---|
| `gix` (gitoxide) | Git-Klon des Store-Repos |
| `clap` | CLI (minimal: `freesynergy-init [store-url]`) |

---

Weiter: [Store](../store/README.md) | [Installation](../../technik/installation.md)
