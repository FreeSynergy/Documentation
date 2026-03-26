# Build-Workflow

[← Zurück zum Index](../INDEX.md) | [Repository-Übersicht](../architektur/repositories.md)

---

## Grundregel

Kein Code wird hochgeladen der nicht alle Quality-Gates bestanden hat.
Das gilt ohne Ausnahme — für neue Repos, für Änderungen, für Migrationen.

---

## Schritt-für-Schritt Workflow

Jedes Modul, jede Crate, jedes Programm folgt dieser Reihenfolge:

```
1. Design Pattern festlegen
2. Structs + Traits definieren (noch kein Impl-Code)
3. cargo check
4. Impl schreiben (OOP-Regeln beachten, siehe unten)
5. cargo clippy --all-targets -- -D warnings
6. cargo fmt --check
7. Unit Tests schreiben
8. cargo test
9. Doku-Seite aktualisieren (lesen → neu schreiben)
10. commit + push
```

**Schritt 1 und 2 zuerst** — nie direkt mit Impl anfangen.
Wer zuerst die Struktur denkt, schreibt besseren Code.

---

## Schritt 1: Design Pattern festlegen

Vor dem ersten Code kommt die Strategie. Welches Muster löst das Problem?

| Situation | Pattern |
|---|---|
| Verschiedene Implementierungen einer Sache | Strategy / Trait-Objekt |
| Zustandsänderungen propagieren | Observer / Bus-Events |
| Externe Dienste einbinden | Adapter (siehe [Adapter-Pattern](../konzepte/adapter.md)) |
| Komplexe Objekte bauen | Builder |
| Einmalige Instanz (z.B. Registry, Bus) | Singleton via `Arc<Mutex<T>>` |
| UI-Komponenten aus Domain-Objekten | View-Trait auf Domain-Struct |
| Aktionen mit Undo/History | Command |

**Regel:** Erst Pattern, dann Code. Nicht umgekehrt.

---

## Schritt 2: OOP-Regeln

FreeSynergy nutzt Rust — aber OOP-Denkweise:

**Objekte statt Daten:**
```rust
// Falsch — Daten-Struct ohne Verhalten
struct PackageStatus {
    id: String,
    running: bool,
    error: Option<String>,
}

// Richtig — Objekt mit Verhalten
struct Package { ... }
impl Package {
    pub fn is_running(&self) -> bool { ... }
    pub fn start(&mut self) -> Result<(), PackageError> { ... }
    pub fn stop(&mut self) -> Result<(), PackageError> { ... }
}
```

**Traits statt match-Blöcke:**
```rust
// Falsch — match-Block der bei jedem neuen Typ erweitert werden muss
fn render(pkg: &PackageType) -> String {
    match pkg {
        PackageType::App => "app view",
        PackageType::Container => "container view",
        // jeder neue Typ bricht hier auf
    }
}

// Richtig — Trait, jeder Typ rendert sich selbst
trait Renderable {
    fn render(&self) -> String;
}
impl Renderable for AppPackage { fn render(&self) -> String { ... } }
impl Renderable for ContainerPackage { fn render(&self) -> String { ... } }
```

**UI als Trait-Implementierung:**
```rust
// UI ist eine Trait-Impl auf dem Domain-Objekt — kein separates UI-Objekt
trait PackageView {
    fn store_card(&self) -> Element;
    fn detail_page(&self) -> Element;
}
impl PackageView for AppPackage { ... }
impl PackageView for ContainerPackage { ... }
```

**Event-Chains statt imperativem Durchreichen:**
```rust
// Aktionen erzeugen Events — andere reagieren, ohne direkt aufgerufen zu werden
bus.publish("package.installed", PackageInstalledEvent { id: "kanidm" }).await?;
// fs-inventory subscribed auf dieses Event und aktualisiert sich selbst
```

**Kleine Objekte, klare Verantwortlichkeit:**
Ein Struct macht genau eine Sache. Wenn eine Methode mehr als ~20 Zeilen hat,
ist sie wahrscheinlich zu groß — in eine eigene Funktion oder ein eigenes Objekt auslagern.

---

## Quality Gates

Alle Gates müssen grün sein vor jedem Commit:

```bash
# 1. Clippy — keine Warnungen, keine Fehler
cargo clippy --all-targets -- -D warnings

# 2. Format
cargo fmt --check

# 3. Tests
cargo test --all

# 4. Dependency-Check (cargo-deny)
cargo deny check

# 5. Build Release
cargo build --release
```

**Jede `lib.rs` und `main.rs` muss diese Zeilen haben:**
```rust
#![deny(clippy::all, clippy::pedantic)]
#![deny(warnings)]
#![allow(clippy::module_name_repetitions)]  // erlaubt wenn nötig
```

---

## Unit-Test-Regeln

- Mind. 1 Test pro öffentlichem Modul
- Tests heißen `test_<was_getestet_wird>` — klar und beschreibend
- Kein Mock für Sachen die direkt testbar sind
- Adapter-Tests nutzen `MockAdapter` (nie echte HTTP-Calls in Unit-Tests)
- Integration-Tests (echte DB, echte Files) in `tests/` — nicht in `src/`

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_package_status_stopped_when_not_running() {
        let pkg = Package::new("kanidm", ResourceStatus::Stopped);
        assert!(!pkg.is_running());
    }

    #[tokio::test]
    async fn test_inventory_insert_and_retrieve() {
        let inv = Inventory::open(":memory:").await.unwrap();
        let resource = InstalledResource::test_fixture("kanidm");
        inv.insert(&resource).await.unwrap();
        let found = inv.get("kanidm").await.unwrap();
        assert_eq!(found.id, "kanidm");
    }
}
```

---

## Repo-Setup-Checkliste

Jedes neue Repo bekommt diese Dateien — in dieser Reihenfolge:

```
[ ] Cargo.toml (workspace oder single crate)
[ ] rustfmt.toml
[ ] deny.toml
[ ] .gitignore (target/, Cargo.lock bei Libs; Cargo.lock committen bei Binaries)
[ ] LICENSE (MIT)
[ ] README.md (Zweck, Build-Anleitung, Architektur-Überblick)
[ ] CLAUDE.md (diese Regeln, für Claude-Sessions)
[ ] assets/icon.svg (Platzhalter-Icon)
[ ] src/lib.rs oder src/main.rs mit #![deny(...)]
```

**`CLAUDE.md` Inhalt (Vorlage):**
```markdown
# Claude-Regeln für dieses Repo

## Qualitäts-Gates (alle müssen grün sein vor jedem Commit)
- cargo clippy --all-targets -- -D warnings
- cargo fmt --check
- cargo test --all
- cargo deny check

## OOP-Regeln
- Traits statt match-Blöcke
- Objekte statt Daten-Structs
- UI als Trait-Implementierung auf Domain-Objekten
- Event-Chains statt imperativem Durchreichen
- Kleine Objekte, eine Verantwortlichkeit

## Workflow
1. Design Pattern festlegen
2. Structs + Traits definieren
3. cargo check
4. Impl schreiben
5. Quality Gates
6. Unit Tests
7. Doku aktualisieren (lesen → neu schreiben)
8. commit + push
```

---

## Binary-Distribution

Wenn ein Programm fertig ist, wird es über GitHub Releases verteilt:

```
1. git tag v1.0.0 && git push --tags
         │
         ▼
2. GitHub Actions baut:
   - Binary für x86_64-unknown-linux-musl (statisch gelinkt)
   - SHA256-Hash der Binary
   - GitHub Release mit beiden Dateien
         │
         ▼
3. Store/-Katalog aktualisieren (manuell oder via CI/CD):
   packages/apps/{name}/{version}.toml
     [[binaries]]
     name   = "{name}"
     arch   = "x86_64-unknown-linux-musl"
     url    = "https://github.com/FreeSynergy/{repo}/releases/download/v{version}/..."
     sha256 = "{hash}"
         │
         ▼
4. Store/-Repo committen + pushen
         │
         ▼
5. fs-store (StoreReader) liest beim nächsten Refresh den neuen Eintrag
```

**Für Pakete mit mehreren Binaries** (z.B. Bots):
Ein GitHub Release enthält mehrere Artifacts. Der Katalog hat mehrere
`[[binaries]]`-Einträge — alle aus demselben Release-Tag.

---

## Dokumentations-Regel

Nach **jeder** Arbeitssitzung gilt:
1. Betroffene Doku-Seite(n) **komplett lesen**
2. **Neu schreiben** — nicht anhängen
3. Widersprüche aktiv suchen und korrigieren
4. commit + push zu `fs-documentation`

Nie einfach ergänzen. "Einfach anhängen" führt zu widersprüchlichen Seiten.
Das ist bei `konzepte/pakete.md` bereits passiert — deshalb diese Regel.

---

Weiter: [Repository-Übersicht](../architektur/repositories.md) | [Adapter-Pattern](../konzepte/adapter.md) | [Bibliotheken](bibliotheken.md)
