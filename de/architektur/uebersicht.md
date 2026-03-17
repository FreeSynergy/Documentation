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
│      │   Node   │  │Conductor │  │  Store  │                 │
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
| [Init](../programme/init/README.md) | Bootstrap: Installiert den Store | Ja (Einmal-Tool) | `FreeSynergy/Init` |
| [Node](../programme/node/README.md) | Projektverwalter + S3-Server | Ja | `FreeSynergy/Node` |
| [Conductor](../programme/conductor/README.md) | Service-Orchestrierer | Ja | `FreeSynergy/Node` (Crate) |
| [Desktop](../programme/desktop/README.md) | Mensch-Maschine-Schnittstelle | Ja (offline-fähig) | `FreeSynergy/Desktop` |
| [Store](../programme/store/README.md) | Paketmanager + Wissen | Ja (Git-Repo) | `FreeSynergy/Store` |
| [Lenses](../programme/lenses/README.md) | Informations-Betrachter | Nein (braucht Services) | Teil von Desktop |
| [Search](../programme/search/README.md) | Mehrstufige Suche | Nein (braucht Bus) | Teil von Node |

**Jedes Programm** funktioniert über CLI, API und UI. Die Business-Logik ist EINMAL implementiert — drei Eingänge.

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
