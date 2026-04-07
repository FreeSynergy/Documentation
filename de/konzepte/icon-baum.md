# Icon-Baum

[← Zurück zum Index](../INDEX.md) | [Icon-System](icon-system.md) | [Themes](themes.md)

---

## Grundprinzip

Der Icon-Baum ist nach **Ontologie** organisiert — was eine Sache ist, nicht wie sie benutzt wird.
Die Verwendung (Action vs. Identity) regelt das Dual-Icon-System.

Jede Gruppe hat:
- Einen **Namen** und eine **Begründung** warum sie existiert
- Eine klare **Abgrenzung** zu anderen Gruppen
- Dokumentierte **Einzelicons** mit Bedeutung und Einzahl/Mehrzahl

---

## Baumstruktur

```
icons/
├── places/          — Orte, Standorte, Umgebungen
├── people/          — Personen, Rollen, soziale Einheiten
├── things/          — Digitale Objekte und Artefakte
├── actions/         — Was man tun kann
├── states/          — Zustände und Status
├── time/            — Zeit und Planung
├── communication/   — Austausch und Verbindung
├── data/            — Daten, Strukturen, Visualisierung
├── security/        — Schutz und Zugriffsrechte
├── media/           — Medien und Inhaltsformen
├── programs/        — FreeSynergy-Programme
└── engines/         — Konkrete Implementierungen (Drittanbieter)
```

---

## places/ — Orte und Umgebungen

**Warum diese Gruppe:** Menschen denken räumlich. Ein Ort ist etwas wo man *ist* oder *hingeht*,
kein Zustand und keine Handlung. Orte sind stabil — sie verändern sich nicht durch Interaktion.

**Abgrenzung:** `programs/` enthält Programme als Konzept; `places/` enthält Orte auch wenn
ein Programm denselben Namen hat. `home` ist ein Ort, nicht ein Programm.

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `home` | Zuhause | — | Persönliches Home-Verzeichnis oder Startseite |
| `server` | Server | `servers` | Ein Rechner/Dienst-Host |
| `node` | Node | `nodes` | FreeSynergy-Node (Serverinstanz) |
| `network` | Netzwerk | `networks` | Verbundene Infrastruktur |
| `building` | Gebäude | `buildings` | Physischer Standort, Organisation |
| `city` | Stadt | `cities` | Geographische Ortsangabe |
| `country` | Land | `countries` | Geographische Länderangabe |
| `folder` | Ordner | `folders` | Verzeichnis im Dateisystem |
| `cloud` | Cloud | — | Entfernter Speicher oder Dienst |
| `desktop` | Desktop | — | Die Arbeitsoberfläche des Nutzers |
| `workspace` | Arbeitsbereich | `workspaces` | Virtueller Arbeitsbereich / Projekt |

---

## people/ — Personen und soziale Einheiten

**Warum diese Gruppe:** Menschen sind keine Objekte und keine Orte. Sie haben Rollen,
Berechtigungen und Identitäten. Diese Gruppe trennt klar zwischen "wer" und "was".

**Abgrenzung:** Ein Bot ist kein Mensch — Bots gehören in `programs/`. Ein Team ist eine
Sammlung von Personen, nicht ein Ort.

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `person` | Person | `people` | Eine einzelne Person, Nutzer-Profil |
| `user` | Nutzer | `users` | Nutzer im System-Kontext (technisch) |
| `admin` | Admin | `admins` | Administrativer Nutzer |
| `guest` | Gast | `guests` | Nicht authentifizierter oder temporärer Nutzer |
| `team` | Team | `teams` | Gruppe mit gemeinsamer Aufgabe |
| `group` | Gruppe | `groups` | Sammlung von Personen (ohne feste Aufgabe) |
| `role` | Rolle | `roles` | Berechtigungs-Rolle |
| `contact` | Kontakt | `contacts` | Adressbuch-Eintrag |
| `owner` | Eigentümer | — | Wer etwas besitzt oder verantwortet |

---

## things/ — Digitale Objekte und Artefakte

**Warum diese Gruppe:** Dinge sind das was Nutzer bearbeiten, speichern und teilen.
Sie sind die Hauptobjekte der Arbeit — keine Aktionen, keine Personen, keine Orte.

**Abgrenzung:** `data/` enthält Strukturen und Visualisierungen; `things/` enthält
die Objekte selbst. Eine Datei ist ein Ding; eine Tabelle ist eine Datenstruktur.

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `file` | Datei | `files` | Eine einzelne Datei |
| `document` | Dokument | `documents` | Textbasiertes Dokument |
| `package` | Paket | `packages` | Installiertes oder installierbares Paket |
| `container` | Container | `containers` | Laufende Container-Instanz |
| `image` | Image | `images` | Container-Image oder Disk-Image |
| `plugin` | Plugin | `plugins` | Erweiterungsmodul |
| `theme` | Theme | `themes` | Design-Paket |
| `icon-set` | Icon-Set | `icon-sets` | Sammlung von Icons |
| `cursor-set` | Cursor-Set | `cursor-sets` | Sammlung von Mauszeigern |
| `backup` | Backup | `backups` | Datensicherung |
| `link` | Link | `links` | Verweis auf eine Ressource |
| `token` | Token | `tokens` | Auth-Token, API-Key |

---

## actions/ — Was man tun kann

**Warum diese Gruppe:** Aktionen beschreiben Verben — was der Nutzer oder das System tut.
Sie sind die häufigsten Icons in Menüs und Buttons.

**Abgrenzung:** `states/` beschreibt wie etwas ist; `actions/` beschreibt was man damit tut.
"Error" ist ein Zustand; "Löschen" ist eine Aktion.

| Icon | Einzahl | Bedeutung |
|---|---|---|
| `add` | Hinzufügen | Neues Objekt erstellen oder hinzufügen |
| `edit` | Bearbeiten | Bestehendes Objekt ändern |
| `delete` | Löschen | Objekt entfernen (reversibel möglich) |
| `save` | Speichern | Zustand persistieren |
| `search` | Suchen | In Daten suchen |
| `filter` | Filtern | Ansicht einschränken |
| `sort` | Sortieren | Reihenfolge ändern |
| `refresh` | Aktualisieren | Daten neu laden |
| `sync` | Synchronisieren | Mit entferntem System abgleichen |
| `send` | Senden | Nachricht oder Daten übertragen |
| `share` | Teilen | Objekt anderen zugänglich machen |
| `copy` | Kopieren | Objekt duplizieren |
| `move` | Verschieben | Objekt an anderen Ort bringen |
| `open` | Öffnen | Programm oder Datei starten |
| `close` | Schließen | Programm oder Fenster beenden |
| `install` | Installieren | Paket einrichten |
| `uninstall` | Deinstallieren | Paket entfernen |
| `start` | Starten | Dienst oder Prozess beginnen |
| `stop` | Stoppen | Dienst oder Prozess beenden |
| `restart` | Neustarten | Dienst oder Prozess neu starten |
| `configure` | Konfigurieren | Einstellungen öffnen |
| `import` | Importieren | Externe Daten einlesen |
| `export` | Exportieren | Daten ausgeben |
| `download` | Herunterladen | Von Netz laden |
| `upload` | Hochladen | Ins Netz laden |
| `login` | Anmelden | Session starten |
| `logout` | Abmelden | Session beenden |
| `back` | Zurück | Vorherigen Schritt gehen |
| `forward` | Vorwärts | Nächsten Schritt gehen |

---

## states/ — Zustände und Status

**Warum diese Gruppe:** Zustände beschreiben wie etwas ist — nicht was es ist und nicht was
man damit tut. Sie erscheinen als Status-Indikatoren, Badges und Warnungen.

| Icon | Bedeutung |
|---|---|
| `active` | Läuft, aktiv, verfügbar |
| `inactive` | Gestoppt, inaktiv |
| `loading` | Wird geladen, in Bearbeitung |
| `success` | Erfolgreich abgeschlossen |
| `warning` | Warnung, Aufmerksamkeit nötig |
| `error` | Fehler, Aktion fehlgeschlagen |
| `info` | Neutral-Information |
| `new` | Neu, noch nicht gesehen |
| `draft` | Entwurf, unveröffentlicht |
| `locked` | Gesperrt, kein Zugriff |
| `unlocked` | Entsperrt, Zugriff möglich |
| `pinned` | Angepinnt, fixiert |
| `starred` | Favorisiert, markiert |
| `hidden` | Ausgeblendet |
| `archived` | Archiviert |
| `deprecated` | Veraltet, wird entfernt |

---

## time/ — Zeit und Planung

**Warum diese Gruppe:** Zeit ist eine eigene Dimension. Zeitobjekte (Kalender, Uhr)
und zeitliche Konzepte (Verlauf, Plan) gehören zusammen.

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `calendar` | Kalender | — | Datum-Auswahl oder Kalenderansicht |
| `clock` | Uhr | — | Uhrzeit, Timer |
| `schedule` | Zeitplan | `schedules` | Geplante Ausführungen |
| `history` | Verlauf | — | Vergangene Ereignisse |
| `deadline` | Frist | `deadlines` | Ablaufdatum |
| `duration` | Dauer | — | Zeitspanne |
| `recurrence` | Wiederholung | — | Wiederkehrende Aufgabe/Ereignis |

---

## communication/ — Austausch und Verbindung

**Warum diese Gruppe:** Kommunikation ist der Austausch zwischen Personen oder Systemen.
Diese Gruppe trennt klar von `actions/` (was man tut) und `things/` (was entsteht).

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `message` | Nachricht | `chat` | Eine Nachricht / Konversationsverlauf |
| `email` | E-Mail | `emails` | Elektronische Post |
| `notification` | Benachrichtigung | `notifications` | System-Hinweis |
| `feed` | Feed | `feeds` | Aktivitätsstrom |
| `comment` | Kommentar | `comments` | Anmerkung zu einem Objekt |
| `mention` | Erwähnung | `mentions` | @-Erwähnung |
| `call` | Anruf | — | Sprach- oder Videoverbindung |
| `broadcast` | Broadcast | — | Nachricht an viele |
| `webhook` | Webhook | `webhooks` | Automatische HTTP-Benachrichtigung |

---

## data/ — Daten, Strukturen, Visualisierung

**Warum diese Gruppe:** Datenstrukturen und ihre Visualisierung sind keine Dinge die man
"besitzt" (wie Dateien), sondern Arten wie Information organisiert und dargestellt wird.

| Icon | Einzahl | Bedeutung |
|---|---|---|
| `database` | Datenbank | Persistenter strukturierter Speicher |
| `table` | Tabelle | Tabellarische Darstellung |
| `list` | Liste | Geordnete oder ungeordnete Auflistung |
| `tree` | Baum | Hierarchische Struktur |
| `chart` | Diagramm | Visuelle Daten-Darstellung |
| `graph` | Graph | Vernetzte Knoten-Darstellung |
| `form` | Formular | Eingabemaske für Daten |
| `field` | Feld | Einzelnes Eingabefeld |
| `tag` | Tag | Markierung/Label |
| `category` | Kategorie | Klassifizierung |
| `namespace` | Namespace | Namensraum |
| `api` | API | Programmierschnittstelle |

---

## security/ — Schutz und Zugriffsrechte

**Warum diese Gruppe:** Sicherheit ist ein eigenständiges Konzept. Alles was mit Zugang,
Schutz und Verifizierung zu tun hat, gehört hierher — nicht in `things/` oder `actions/`.

| Icon | Bedeutung |
|---|---|
| `lock` | Gesperrt, verschlüsselt |
| `unlock` | Entsperrt, zugänglich |
| `key` | Zugangsschlüssel, Passwort |
| `shield` | Schutz, Sicherheit allgemein |
| `fingerprint` | Biometrische Authentifizierung |
| `certificate` | TLS-Zertifikat, Signatur |
| `permission` | Berechtigung |
| `audit` | Prüfung, Log |
| `2fa` | Zwei-Faktor-Authentifizierung |
| `vpn` | Verschlüsselte Netzwerkverbindung |

---

## media/ — Medien und Inhaltsformen

**Warum diese Gruppe:** Medien sind Inhalte die angezeigt, abgespielt oder gelesen werden.
Sie sind keine generischen Dateien (`things/`) — sie haben Formatcharakter.

| Icon | Einzahl | Mehrzahl | Bedeutung |
|---|---|---|---|
| `photo` | Foto | `photos` | Bild/Foto |
| `video` | Video | `videos` | Videodatei oder -stream |
| `audio` | Audio | — | Audiodatei oder -stream |
| `document` | Dokument | `documents` | Textbasierter Inhalt |
| `code` | Code | — | Quellcode |
| `terminal` | Terminal | — | Kommandozeilen-Oberfläche |
| `markdown` | Markdown | — | Markdown-formatierter Text |
| `pdf` | PDF | `pdfs` | PDF-Dokument |
| `presentation` | Präsentation | `presentations` | Folien-Dokument |
| `spreadsheet` | Tabellenkalkulation | — | Berechnungs-Dokument |

---

## programs/ — FreeSynergy-Programme

**Warum diese Gruppe:** FreeSynergy-Programme sind konzeptuelle Werkzeuge — kein
Drittanbieter-Produkt. `browser` ist das Konzept Browser, nicht Firefox.
Drittanbieter-Icons für konkrete Implementierungen gehören in `engines/`.

| Icon | Bedeutung |
|---|---|
| `browser` | Web-Browser (Engine-unabhängig) |
| `store` | Paket-Manager/Store |
| `desktop` | Desktop-Shell |
| `terminal` | Terminal-Emulator |
| `settings` | Einstellungen eines Programms |
| `lenses` | Aggregierte Daten-Ansichten |
| `tasks` | Automatisierungs-Pipeline |
| `ai` | KI-Assistent |
| `bots` | Bot-Manager |
| `container` | Container-Manager |
| `builder` | Paket-Ersteller |
| `theme` | Theme-Manager |
| `icons` | Icon-Manager |
| `language` | Sprach-Manager |
| `auth` | Authentifizierungs-Manager |
| `node` | FreeSynergy.Node-Server |
| `inventory` | Installiertes verwalten |
| `registry` | Dienste-Verzeichnis |
| `session` | Session-Verwaltung |
| `federation` | Föderations-Verbindungen |
| `sync` | CRDT-Synchronisation |
| `search` | Suche |

---

## engines/ — Konkrete Implementierungen

**Warum diese Gruppe:** Eine Engine ist eine konkrete Drittanbieter-Technologie
die hinter einem FreeSynergy-Programm-Konzept steckt. Das Engine-Icon erscheint
als Identity-Icon wenn ein Objekt oder Programm diese Implementierung nutzt.

Das Firefox-Icon ist nicht das Browser-Icon — Firefox ist die Engine.

| Icon | Bedeutung |
|---|---|
| `firefox` | Gecko/SpiderMonkey Browser-Engine |
| `servo` | Servo Browser-Engine |
| `kanidm` | Kanidm Auth-Backend |
| `matrix` / `tuwunel` | Matrix-Homeserver |
| `stalwart` | Stalwart Mail-Server |
| `mistral` | Mistral.rs LLM-Engine |
| `ollama` | Ollama LLM-Runtime |
| `podman` | Podman Container-Engine |
| `docker` | Docker Container-Engine |
| `sqlite` | SQLite Datenbank-Engine |
| `postgres` | PostgreSQL Datenbank-Engine |

---

## Regeln für neue Icons

1. **Gruppe zuerst** — zu welcher ontologischen Kategorie gehört das neue Konzept?
2. **Einzahl/Mehrzahl prüfen** — macht ein Plural-Icon semantisch Sinn?
3. **Begründung dokumentieren** — warum heißt es so? Was grenzt es ab?
4. **Kein Duplikat** — gibt es schon ein Icon das dieselbe Bedeutung hat?
5. **Kein Verwendungs-Icon** — Icons beschreiben was etwas ist, nicht wie es benutzt wird

---

Weiter: [Icon-System](icon-system.md) | [Themes](themes.md) | [Icon Manager](../programme/icons/README.md)
