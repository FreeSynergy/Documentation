# Architektur-Übersicht

[← Zurück zum Index](../INDEX.md)

---

## Die Philosophie

FreeSynergy basiert auf drei Grundgedanken:

1. **Information vor Werkzeug.** Der Mensch braucht Antworten, nicht Programme. Ob eine Information aus einem Wiki, einem Messenger oder einer Datenbank kommt, ist für den Menschen irrelevant. Die Technik muss in den Hintergrund — was wichtig ist, muss nach vorne. Programme arbeiten im Hintergrund zusammen. Der Mensch sieht nur das Ergebnis. Wenn er tiefer eintauchen will, klickt er sich zum Werkzeug durch — aber erst dann.

2. **Dezentral und freiwillig.** Kein Zentralserver, kein Zwang. Jeder Node ist souverän. Zusammenarbeit ist immer opt-in. Rechte können nur eingeschränkt, nie erweitert werden.

3. **Offene Standards.** WASM, OIDC, SCIM, ActivityPub, OCI, S3, Automerge. Keine proprietären Protokolle. Jedes Teil ist austauschbar.

---

## Programm-Übersicht

```
┌───────────────────────────────────────────────────────────────┐
│                        Mensch                                  │
│                          │                                     │
│                    ┌─────▼─────┐                              │
│                    │  Desktop  │  UI, Widgets, Lenses, Search │
│                    └─────┬─────┘                              │
│                          │ API                                │
│            ┌─────────────┼─────────────┐                      │
│            │             │             │                       │
│      ┌─────▼────┐  ┌────▼─────┐  ┌───▼────┐                 │
│      │   Node   │  │Container Manager │  │  Store  │                 │
│      │Projekte, │  │Services, │  │Wissen,  │                 │
│      │Hosts,    │  │Container,│  │Pakete,  │                 │
│      │Föder.,   │  │Variablen │  │Suche    │                 │
│      │S3-Server │  │          │  │         │                 │
│      └─────┬────┘  └────┬─────┘  └───┬────┘                 │
│            │             │             │                       │
│      ┌─────▼─────────────▼─────────────▼──────┐              │
│      │              Message Bus                │              │
│      │   Pub/Sub, Rollen-basiert, Bridges      │              │
│      └─────┬──────────┬──────────┬────────────┘              │
│            │          │          │                             │
│      ┌─────▼───┐ ┌───▼────┐ ┌──▼──────────┐                 │
│      │Services │ │  Bots  │ │ S3-Storage   │                 │
│      │Kanidm,  │ │Telegram│ │ /profiles/   │                 │
│      │Forgejo, │ │Matrix, │ │ /backups/    │                 │
│      │Outline  │ │Discord │ │ /media/      │                 │
│      └─────────┘ └────────┘ └──────────────┘                 │
└───────────────────────────────────────────────────────────────┘
```

## Programme

| Programm | Aufgabe | Eigenständig? | Repo |
|---|---|---|---|
| [Init](../programme/init/README.md) | Bootstrap: Installiert den Store | Ja (Einmal-Tool) | `FreeSynergy/fs-init` |
| [Node](../programme/node/README.md) | Projektverwalter + S3-Server | Ja | `FreeSynergy/fs-node` |
| [Container Manager](../programme/container/README.md) | Service-Orchestrierer | Teil von Desktop | `FreeSynergy/fs-desktop` |
| [Builder](../programme/builder/README.md) | Ressourcen bauen & validieren | Teil von fs-apps | `FreeSynergy/fs-apps` |
| [Desktop](../programme/desktop/README.md) | Mensch-Maschine-Schnittstelle | Ja (offline-fähig) | `FreeSynergy/fs-desktop` |
| [Browser](../programme/browser/README.md) | Eingebetteter Web-Browser | Ja | `FreeSynergy/fs-apps` |
| [Store](../programme/store/README.md) | Paketmanager + Wissen | Ja (Git-Repo) | `FreeSynergy/Store` |
| [Lenses](../programme/lenses/README.md) | Informations-Betrachter | Nein (braucht Services) | Teil von Desktop |
| [Search](../programme/search/README.md) | Mehrstufige Suche | Nein (braucht Bus) | Teil von Node |

**Jedes eigenständige Programm** hat CLI + API + optional WGUI. Die Business-Logik ist EINMAL implementiert — mehrere Eingänge.

**Regel für eigene Repos:** Ein Programm bekommt ein eigenes Repo wenn es alleine laufen kann — mit eigenem Release-Zyklus und eigener Versionierung. Einzige Ausnahme: Der S3-Server ist Infrastruktur von Node und hat kein eigenes Repo. Shared Libraries leben im Monorepo `FreeSynergy/fs-libs`.

## CLI = API = Objekt-Methoden

**Jede CLI-Aktion ist dieselbe Funktion wie der entsprechende API-Endpoint.**

```
fsn store install kanidm
    ↕ identisch mit
POST /api/store/install {"id": "kanidm"}
    ↕ identisch mit
Store::install("kanidm")   ← Rust-Methode
```

Die CLI ist ein dünner Wrapper der Argumente parsed (clap) und dann die Rust-Methode aufruft. Dieselbe Rust-Methode wird vom API-Server (axum) genutzt. Business-Logik genau einmal — kein doppelter Code.

Das gilt für alle Programme: `fsn store`, `fsn node`, `fsn container`, `fsn bus`, `fsn bot` — alles.

Konsequenz: **Was man per UI machen kann, kann man auch per CLI machen, und umgekehrt.** Die CLI ist keine abgespeckte Version der UI.

## SysInfo

FreeSynergy kennt das System auf dem es läuft. Die `fsn-sysinfo` Library (Teil von `FreeSynergy.Lib`) liefert:

- **Statisch (gecacht):** OS, Architektur, verfügbare Features (PAM, systemd, launchd, Podman, Git, …)
- **Dynamisch (auf Anfrage):** Festplattenbelegung, RAM, CPU-Temperatur, SMART-Status
- **Alerting (Bus-Events):** Konfigurierbare Schwellenwerte → `sysinfo.alert.*`

Pakete deklarieren Platform-Anforderungen als Tags (`requires:systemd`, `platform:linux`). Der Store kombiniert diese mit SysInfo und zeigt bei der Installation nur was auf dem aktuellen System möglich ist.

Siehe: [SysInfo-Konzept](../konzepte/sysinfo.md)

## Die drei Ebenen (Store / Inventory / Managers)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   STORE         Das Mögliche   — Katalog aller verfügbaren         │
│                 Ressourcen. Git-Repo. Kein Zustand.                 │
│                                                                     │
│   INVENTORY     Der Jetzt-Zustand — Was ist installiert, was       │
│                 läuft, welche Rollen sind aktiv. Einzige Wahrheit.  │
│                                                                     │
│   MANAGERS      Das Wie — Container Manager, Theme Manager,    │
│                 Language Manager. Führen Aktionen aus und schreiben │
│                 das Ergebnis ins Inventory.                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Alles was im UI angezeigt wird, kommt ausschließlich aus dem Inventory.**
Der Store zeigt was möglich ist. Das Inventory zeigt was da ist.
Die Manager kümmern sich darum, dass aus dem Möglichen Wirklichkeit wird — und schreiben das Ergebnis ins Inventory.

Siehe: [Inventory-Konzept](../konzepte/inventory.md)

## Der Installationsweg

```
FreeSynergy.Init → Store → Alles andere
```

Kein separates Installationsprogramm. Der Store ist der Paketmanager. Siehe [Installation](../technik/installation.md).

## Datenfluss

```
Mensch tippt "Meine Gruppe Köln" in Lenses
    │
    ▼
Lenses fragt den Bus: "Welche Services haben Daten über 'Helfa Köln'?"
    │
    ▼
Bus leitet an alle Services weiter (über Rollen, nie direkt):
    │
    ├── Rolle 'wiki' (Outline): Artikel-Zusammenfassung + Link
    ├── Rolle 'map' (uMap): Kartenausschnitt + Link
    ├── Rolle 'chat' (Matrix): Letzte Nachrichten + Link
    ├── Rolle 'tasks' (Vikunja): Offene Aufgaben + Link
    └── Rolle 'git' (Forgejo): Repository-Status + Link
    │
    ▼
Lenses zeigt zusammengefasste Ansicht
Mensch klickt auf "Offene Aufgaben" → wird zu Vikunja weitergeleitet
```

**Der Mensch muss NICHT wissen welches Programm was hat.** Er sucht und bekommt alles.

---

Weiter: [Rollen-System](../konzepte/rollen.md) | [Init](../programme/init/README.md) | [Storage-Layer](../technik/storage.md)
