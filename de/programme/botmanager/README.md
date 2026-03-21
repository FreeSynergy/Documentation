# BotManager — Zentrale Bot-Übersicht

> Lebt in **fs-bots** (`bots/`), Crate: `fs-bots`.

[← Zurück zum Index](../../INDEX.md) | [Bots Konzept](../../konzepte/bots.md) | [Manager](../../konzepte/manager.md) | [Desktop](../desktop/README.md)

---

## Was der BotManager macht

Der BotManager ist die **zentrale Übersicht** für alle Bot-Instanzen.
Er zeigt alle Bots auf einen Blick — was bei einzelnen Bot-Programmen mehrere Klicks erfordert,
ist hier sofort sichtbar.

```
Einzelner Bot öffnen  → nur dieser Bot, direkte Arbeit
Bot Manager öffnen    → alle Bots, Vergleich, Gesamtstatus
```

**Der BotManager erstellt neue Bot-Instanzen.**
Eine fertige Instanz erscheint dann als eigenes Programm im Desktop (eigenes Icon, eigener Name).

```
Store ←→ BotManager ←→ Desktop (eigene Icons je Instanz)
              ↕
           Bus → Bots → fs-channel → Messenger
```

---

## Bot-Instanzen erstellen

Schritt 1 — Basis (Meta):

```toml
name        = "Community Bot"
description = "Verwaltet die Köln-Community"
icon        = "bot-community"
tags        = ["community", "köln", "broadcast"]
visibility  = "group"    # private | group | public
```

Schritt 2 — Bot-Typ wählen (aus Store):
```
bot-control     → Koordiniert andere Bots
bot-broadcast   → Bus-Events weiterleiten
bot-gatekeeper  → Beitritte verifizieren
bot-calendar    → Termine + Erinnerungen
...
```

Schritt 3 — Messenger-Konten verbinden:
```
+ Telegram hinzufügen   → Bot-Token eingeben
+ Telegram hinzufügen   → zweiter Account (separate ID + Token)
+ Matrix hinzufügen     → Homeserver-URL + Access Token
+ Discord hinzufügen    → Bot-Token
```

Schritt 4 — Steuerung konfigurieren:
```
FSN-Ebene:       private / group / public
Messenger-Ebene: everyone / admins / nobody
Control Bot:     (optional) nur dieser Control Bot steuert
```

Schritt 5 — Rechte pro Funktion pro Gruppe (nach dem Verbinden).

---

## Dashboard

Beim Öffnen des BotManagers: Übersicht aller Instanzen.

```
┌──────────────────────────────────────────────────────────┐
│  Bot Manager                                             │
├──────────────────────────────────────────────────────────┤
│  [Community Bot]  ● Online  · 47 Gruppen · 3 offen      │
│  [Projekt Bot]    ● Online  · 12 Gruppen                 │
│  [Privater Bot]   ○ Offline · Token fehlt ⚠️             │
│                                                          │
│  [+ Neuen Bot erstellen]                                 │
└──────────────────────────────────────────────────────────┘
```

Klick auf eine Instanz → öffnet diese Bot-Instanz als eigenes Programm-Fenster.

---

## Gruppen-Ansicht (pro Bot)

Jede Bot-Instanz verwaltet ihre Gruppen mit Filter + Collections.

### Filter

```
[Suche...]  [Messenger ▾]  [Aktivität ▾]  [Größe ▾]  [Status ▾]

Ergebnisse:
  📱 Projekt Köln (Telegram)      · 124 Mitglieder · broadcast, gatekeeper
  💬 #projekt-köln:matrix.org     ·  89 Mitglieder · broadcast
  🎮 #projekt-köln (Discord)      ·  34 Mitglieder · broadcast
```

Filter-Optionen:
- Mitglieder-Anzahl (z.B. > 100)
- Letzte Aktivität (aktiv in X Tagen)
- Gruppenname (enthält / beginnt mit / endet mit)
- Messenger (Telegram / Matrix / Discord / ...)
- Aktive Module
- Bot-Status in der Gruppe (OK / Fehler / Bot entfernt)

### Collections

Manuelle Gruppierungen — keine Automatik.
Eine Gruppe kann in mehreren Collections vorkommen.

```
Collection "Köln"           (12 Gruppen)
Collection "Projekt-Gruppen" (83 Gruppen)
  → "Projekt Köln" ist in beiden
```

### Bulk-Aktionen

Mehrere Gruppen markieren → Aktion auf alle:

```
☑ Projekt Köln  ☑ Projekt Berlin  ☑ Projekt Hamburg

[Broadcast senden]  [Modul aktivieren]  [In Collection]  [Rechte setzen]
```

---

## Views

### Accounts

Alle konfigurierten Messenger-Konten über alle Bot-Instanzen.
Status: verbunden / Fehler / nicht konfiguriert.
Zugangsdaten werden nie im Klartext angezeigt (Secrets-Store).

### Bot-Status

```
Community Bot (Telegram #1)   ● online   · 3 Module aktiv
Community Bot (Matrix)        ● online   · 2 Module aktiv
Projekt Bot (Discord)         ○ offline  · Token fehlt ⚠️ → Einstellungen
```

Klick auf Fehler → direkte Verlinkung zur Lösung.

### Broadcast

Nachricht an Gruppen senden ohne Messenger zu öffnen:

```
Bot:        [ Community Bot ▾ ]
Empfänger:  [ Alle Gruppen ▾ ] oder Collection / Filter / einzelne Gruppen
Messenger:  [ Alle ▾ ]
Nachricht:  [___________________________________________________]
            [ Markdown ]  [ Datei anhängen ]  [ Senden ]
```

Publiziert `bot.broadcast` als Bus-Event → Bot sendet.

### Subscriptions

Welche Gruppe abonniert welche Bus-Events:

```
Gruppe "Projekt Köln" (Telegram)
  ✅ git.commit          → Commits aus Forgejo
  ✅ wiki.page.created   → Neue Seiten
  ☐  tasks.assigned     → Aufgaben-Zuweisung
[ + Subscription hinzufügen ]
```

### Gatekeeper

Offene Beitrittsanfragen über alle Bots:

```
@max_muster → "Projekt Köln" (Community Bot, Telegram)
  IAM: ✅ Verifiziert (max.muster@helfa-koeln.de)
  [ Genehmigen ]  [ Ablehnen ]

@unknown_user → "Team Süd" (Projekt Bot, Matrix)
  IAM: ❌ Nicht im System
  [ Genehmigen ]  [ Ablehnen ]  [ Blockieren ]
```

### Module

Installierte Module je Bot-Instanz, aktiv/inaktiv:

```
Community Bot
  📢 Broadcast Module    v1.2.0  ● aktiv
  🚪 Gatekeeper Module   v1.0.3  ● aktiv   (IAM installiert ✅)
  📅 Calendar Module     v0.5.0  ● aktiv   (Desktop-Integration)
  🎫 Tickets Module      v0.8.1  ○ inaktiv (kein Ticket-Service)

[ + Modul aus Store installieren ]
```

Inaktive Module werden automatisch aktiv wenn die benötigte Rolle verfügbar wird.

### Log (aggregiert)

Alle Bot-Logs auf einen Blick — filterbar nach Bot, Zeitraum, Aktion:

```
2026-03-20 14:23  Community Bot   alice      /broadcast → 12 Gruppen       ✅
2026-03-20 13:51  Projekt Bot     [System]   Verbindung Discord verloren   ⚠️
2026-03-20 09:12  Community Bot   bob        /approve request-42            ✅
```

Jeder Bot hat zusätzlich sein eigenes Log im Bot-Programm selbst.

---

## Eigenständigkeit

Der BotManager läuft als Crate in `FreeSynergy.Managers` (`bots/`).
Desktop bindet ihn als Komponente ein (`app-botmanager`).
Er hat kein eigenes Binary — er ist eine Bibliothek mit UI-Komponenten.

Jede Bot-Instanz die hier erstellt wird, bekommt ein eigenes Binary + Icon im Desktop.

---

## Rechte

| Aktion | Benötigtes Recht |
|---|---|
| Bot-Status sehen | `read` |
| Broadcasts empfangen (Subscriptions) | `read` |
| Broadcasts senden | `execute` |
| Subscriptions verwalten | `write` |
| Gatekeeper: Anfragen sehen | `read` |
| Gatekeeper: Genehmigen / Ablehnen | `execute` |
| Module aktivieren / deaktivieren | `write` |
| Bot-Instanz erstellen / löschen | `execute` (Admin) |
| Bot-Tokens ändern | `execute` (Admin) |

---

## Bus-Integration

Der BotManager redet **nie direkt** mit Messengern — immer über den Bus:

| Event | Richtung | Beschreibung |
|---|---|---|
| `bot.broadcast` | BotManager → Bot | Nachricht senden |
| `bot.gatekeeper.approve` | BotManager → Bot | Beitritt genehmigen |
| `bot.gatekeeper.deny` | BotManager → Bot | Beitritt ablehnen |
| `bot.subscription.add` | BotManager → Bot | Subscription hinzufügen |
| `bot.subscription.remove` | BotManager → Bot | Subscription entfernen |
| `bot.status.request` | BotManager → Bot | Status abfragen |
| `bot.status.response` | Bot → BotManager | Status-Antwort |
| `chat.join_request` | Bot → BotManager | Neuer Beitrittsantrag |

---

## Bibliotheken

| Crate | Zweck |
|---|---|
| `dioxus` 0.7.x | UI-Komponenten |
| `fs-bus` | Bus-Client |
| `fs-inventory` | Welche Bots/Module sind installiert? |
| `bot-db` | BotDb Objekt — SeaORM via fs-db |

Messenger-Crates (`grammers`, `matrix-sdk`, `serenity`, ...) gehören zu den jeweiligen Bot-Instanzen,
nicht zum BotManager.

---

## Repo

https://github.com/FreeSynergy/fs-bots — Unterordner `bots/`

---

Weiter: [Bots Konzept](../../konzepte/bots.md) | [Manager](../../konzepte/manager.md) | [Desktop](../desktop/README.md) | [Bus](../../konzepte/bus.md)
