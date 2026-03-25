# Bus-API Namespaces — Standardisierte Capability-Adressen

[← Zurück zum Index](../INDEX.md) | [Message Bus](bus.md) | [Capabilities](capabilities.md) | [Pakete](pakete.md)

---

## Grundidee

Jedes Objekt im System, das etwas tun kann, **deklariert seinen Namespace**. Dieser Namespace ist hierarchisch, standardisiert und maschinenlesbar. Er ist der Anknüpfpunkt für den Bus — alle Objekte sprechen dieselbe Sprache, egal ob GUI, CLI, API oder ein anderes Objekt.

```
installer::packages::containers::install
     │          │          │         │
     │          │          │         └── Aktion
     │          │          └──────────── Typ
     │          └─────────────────────── Kategorie
     └────────────────────────────────── Domain
```

---

## Vertragsregel: Level → vollständige Abdeckung

Wer einen Namespace auf **Level N** deklariert, **muss alles auf Level N+1 und tiefer** implementieren — ohne Ausnahme.

```
installer::packages            → MUSS alle Typen können (containers, apps, themes, ...)
installer::packages::containers → NUR Container nötig, andere Typen ignoriert
```

Das Prinzip: ein halber Namespace ist kein Namespace. Wer mehr verspricht, muss mehr liefern.

---

## Namespace: `installer`

Das Standard-Namespace für alles, was installiert, entfernt oder aktualisiert werden kann.

### `installer::packages`

Vollständige Paketverwaltung — wer das deklariert, muss alle Paket-Typen handhaben.

| Namespace | Beschreibung |
|---|---|
| `installer::packages` | Alle Paket-Typen (vollständige Abdeckung Pflicht) |
| `installer::packages::apps` | Desktop/CLI-Anwendungen (Binaries) |
| `installer::packages::containers` | Container-Dienste (Podman/Quadlet) |
| `installer::packages::themes` | Visuelle Themes |
| `installer::packages::widgets` | Desktop-Widgets |
| `installer::packages::languages` | Sprachpakete |
| `installer::packages::tasks` | Automatisierungs-Templates |
| `installer::packages::icons` | Icon-Sets |
| `installer::packages::bundles` | Meta-Pakete (aggregiert andere) |

#### Aktionen pro Typ

| Aktion | Beschreibung |
|---|---|
| `::install` | Paket installieren (neueste oder angegebene Version) |
| `::remove` | Paket entfernen |
| `::update` | Auf neuere Version aktualisieren |
| `::list` | Installierte Pakete auflisten |
| `::info` | Metadaten eines Pakets abrufen |
| `::search` | Im Katalog suchen |
| `::pin` | Version pinnen (kein Auto-Update) |
| `::unpin` | Pin entfernen |

Beispiele:

```
installer::packages::apps::install
installer::packages::apps::remove
installer::packages::apps::update
installer::packages::apps::list
installer::packages::containers::install
installer::packages::containers::remove
installer::packages::containers::features::enable
installer::packages::containers::features::disable
installer::packages::themes::install
installer::packages::themes::apply
installer::packages::languages::install
installer::packages::languages::remove
installer::packages::bundles::install
installer::packages::bundles::expand        ← Bundle auflösen in Einzel-Pakete
```

#### Container-spezifische Aktionen

Container haben erweiterte Aktionen für Dienst-Management:

```
installer::packages::containers::start
installer::packages::containers::stop
installer::packages::containers::restart
installer::packages::containers::status
installer::packages::containers::logs
installer::packages::containers::features::enable
installer::packages::containers::features::disable
installer::packages::containers::features::list
```

### `installer::init`

Für das Bootstrap-Paket — USB-Installer erstellen und verwalten.

```
installer::init::build        ← Image bauen
installer::init::download     ← Aktuelle Version herunterladen
installer::init::info         ← Versions-Info
```

### `installer::sources`

Paketquellen (Store-Repositories) verwalten.

```
installer::sources::add
installer::sources::remove
installer::sources::list
installer::sources::refresh   ← Katalog aktualisieren
installer::sources::trust     ← Quelle als vertrauenswürdig markieren
```

---

## Namespace: `store`

Direkte Interaktion mit dem Store-Katalog (lesen, nicht installieren).

```
store::catalog::fetch         ← Root-Katalog laden
store::catalog::namespace     ← Namespace-Katalog laden
store::catalog::package       ← Einzel-Paket-Metadaten laden
store::icons::fetch           ← Icon eines Pakets laden
store::help::fetch            ← Hilfe-Dateien eines Pakets laden (Locale-aware)
store::search                 ← Volltext-Suche im Katalog
```

---

## Deklaration im Paket-Manifest

Pakete deklarieren welche Namespaces sie **unterstützen** (`provides`) und welche sie **brauchen** (`requires`):

```toml
[provides]
bus = [
    "installer::packages::containers",
    "installer::packages::containers::features",
]

[requires]
bus = [
    "installer::sources::refresh",
]
```

---

## Wie Objekte miteinander reden

Kein Objekt spricht ein anderes direkt an. Alle Nachrichten gehen über den Bus — adressiert an einen **Namespace**, nicht an eine Instanz.

```
GUI → Bus: "installer::packages::containers::install" { id: "forgejo", version: "latest" }
Bus → wer `installer::packages::containers` anbietet: Nachricht zustellen
Container Manager → Bus: "installer::packages::containers::install.done" { id: "forgejo", ... }
Bus → wer das abonniert hat (GUI, Monitoring, ...): Nachricht zustellen
```

Das bedeutet: **der Store-App ist es egal, ob der Container Manager auf dem gleichen Gerät läuft, oder per API auf einem anderen Node.** Der Bus routet.

---

## Rückwärtskompatibilität

Neue Aktionen werden hinzugefügt — bestehende werden nie entfernt oder umbenannt.

Empfänger die eine Aktion nicht kennen: **stillschweigend ignorieren**. Kein Absturz, kein Fehler. Optional: Warning dass eine neuere API-Version vorhanden ist.

Neue Felder in Payloads: ebenfalls stillschweigend ignorieren.

```
Alt:  installer::packages::containers::install { id, version }
Neu:  installer::packages::containers::install { id, version, branch, dry_run }

→ Alter Empfänger: liest id und version, ignoriert branch und dry_run — läuft weiter
```

---

## Master-Liste (laufend gepflegt)

Diese Tabelle ist die **einzige Wahrheit** über alle registrierten Namespaces. Neue Namespaces werden hier zuerst eingetragen — bevor der Code geschrieben wird.

| Namespace | Implementiert von | Status |
|---|---|---|
| `installer::packages` | fs-store | geplant |
| `installer::packages::apps` | fs-store | geplant |
| `installer::packages::containers` | fs-store, fs-node | geplant |
| `installer::packages::themes` | fs-store | geplant |
| `installer::packages::widgets` | fs-store | geplant |
| `installer::packages::languages` | fs-store | geplant |
| `installer::packages::tasks` | fs-store | geplant |
| `installer::packages::icons` | fs-store | geplant |
| `installer::packages::bundles` | fs-store | geplant |
| `installer::packages::containers::features` | fs-store, fs-node | geplant |
| `installer::init` | fs-store | geplant |
| `installer::sources` | fs-store | geplant |
| `store::catalog` | fs-store | geplant |
| `store::icons` | fs-store | geplant |
| `store::help` | fs-store | geplant |
| `store::search` | fs-store | geplant |

---

Weiter: [Message Bus](bus.md) | [Capabilities](capabilities.md) | [Pakete](pakete.md) | [Store](../programme/store/README.md)
